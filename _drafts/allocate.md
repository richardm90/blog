---
layout: post
title: "DB2 Variable Length Columns"
date: 2024-03-07
tags:
- IBM i
- DB2
- SQL
---

Whilst trying to understand the impact of the `ALLOCATE` keyword on the table column definition for LOB columns I realised my understanding wasn't as complete as I originally thought.

This document records my findings.


## Useful References

Firstly, I'd like to thank the authors and contributors of the following resources, which helped cement my understanding of the topic.

* Jon Paris and Susan Gantner wrote an excellant article titled [An Introduction to Variable Field Lengths in RPG](https://techchannel.com/SMB/09/2019/variable-Field-Lengths-in-RPG). Within this article there's a section titled 'Defining Variable-Length Fields in the Database' that provides an excellant overview on the subject.
* Simon Hutchinson for his article titled [Variable length field in DDS file](https://www.rpgpgm.com/2020/03/variable-length-field-in-dds-file.html), of special interest are the comments from Birgitta Hauser.
* Various contributors to the [Variable length fields question](https://archive.midrange.com/midrange-l/201407/msg00823.html) post on [midrange.com](https://midrange.com/).

In addition to these resources I found some excellant information in the IBM [Database Performance and Query Optimization](https://www.ibm.com/docs/en/i/7.5?topic=database-performance-query-optimization) manual, specifically the [Tips for using VARCHAR and VARGRAPHIC data types in databases](https://www.ibm.com/docs/en/i/7.5?topic=considerations-varchar-vargraphic) section. I wish I'd found this earlier in my investigations!


## Salient Points

Below I've provided an overview of the information I've gathered from the above resources. Most of this I've pulled straight from the IBM manual.

* Data in a variable-length column is stored internally in two areas: a fixed-length or ALLOCATE area and an overflow area.
* Storage for LOB columns is allocated in the same manner as for VARCHAR columns.
* If you don't specify the ALLOCATE keyword the default is ALLOCATE(0).
* ALLOCATE(0) requires two reads; one to read the fixed-length portion of the row and one to read the overflow space. 
* The overflow area is exactly that, if you have a 50 VARCHAR column with ALLOCATE(20) and you store 30 characters in the column then 20 will be stored in the fixed-length area and 10 will be stored in the overlow area.
* In order to keep track of the data in the auxiliary area, an additional 25 bytes is added to any record that includes variable length fields.
* When a column stored in the overflow storage area is referenced, all the columns in that area are paged into memory. A reference to a "smaller" VARCHAR column that is in the overflow area can potentially force extra paging of LOB columns. For example, a VARCHAR(256) column retrieved by an application can have a side effect of paging in two 5 MB BLOB columns that are in the same row. In order to prevent this side effect, you might want to use the ALLOCATE keyword to ensure that only LOB columns are stored in the overflow area.


## DB2 Stats

DB2 for i contains two views that can help you understand how variable length fields are used within our system.

* [QSYS2.SYSTABLESTAT](https://www.ibm.com/docs/en/i/7.5?topic=views-systablestat)
* [QSYS2.SYSCOLUMNSTAT](https://www.ibm.com/docs/en/i/7.5?topic=views-syscolumnstat)

This information can also be accessed via ACS Schemas.

* For table level statistics, right-click on table and select Description.
* For column level statistics, right-click on table and select Statistic Data. You will be presented with a list of columns that have statistic data (you can add a column to force collection of statistics, press New). Select the required column and press Details.


### SYSTABLESTAT

The interesting stats for me can be accessed with the following SQL.

```sql
select table_schema, table_name, number_rows, overflow, data_size, variable_length_size from qsys2.systablestat where table_name='YOUR_TABLE_NAME';
```

This shows you the total numbers of rows and the number of rows that contain data in the overflow area.


### SYSCOLUMNSTAT

The interesting stats for me can be accessed with the following SQL.

```sql
select table_schema, table_name, column_name, number_distinct_values, average_column_length, maximum_column_length, length_at_90th_percentile, overflow_rows from qsys2.syscolumnstat where table_name='YOUR_TABLE_NAME';
```

I had to look up what the "Length at 90th Percentile" was telling me.


## Example 1: VARCHAR with no ALLOCATE value

In this simple example I'm creating a new table called `AUTHORS` to contain a list of my favourite authors. This table contains a single variable-length column called `NAME` with a maximum length of 50 characters. I have not specified the `ALLOCATE` keyword so the default `ALLOCATE(0)` is used.

I then insert three rows into the table.

The final select statement on the `QSYS2.SYSTABLESTAT` view will show me how many rows are contained in the overflow area.

```sql
create table authors ( name varchar(50) ) ; 

insert into authors (name) values('Agatha Christie');
insert into authors (name) values('J. K. Rowling');
insert into authors (name) values('William Shakespeare');

select table_schema, table_name, number_rows, overflow, data_size, variable_length_size from qsys2.systablestat where table_name='AUTHORS';
```

The result is as I expected. All three rows have data stored in the overflow area.


## Example 2: VARCHAR with ALLOCATE(15) Part 1

This examples builds on Example 1 with one simple change. I've added the `ALLOCATE` keyword to the `NAME` column. I'm expecting the majority of the names to be 15 characters or less, which means that most of the names should be stored in the fixed-length portion of the row and therefore only require one read.

Note that only one row has a name that exceeds 15 characters.

```sql
drop table authors ; 

create table authors ( name varchar(50) allocate(15) ) ; 

insert into authors (name) values('Agatha Christie');
insert into authors (name) values('J. K. Rowling');
insert into authors (name) values('William Shakespeare');

select table_schema, table_name, number_rows, overflow, data_size, variable_length_size from qsys2.systablestat where table_name='AUTHORS';
```

The result is not as I expected. No rows have data stored in the overflow area. The data for all three rows is stored in the fixed-length portion.

I can only conclude that there are some DB2 internal rules that decide when to use the overflow area that are not solely based on the `ALLOCATE` keyword. I am going to assume that DB2 may choose to place data in the fixed-length area, even when the length of the data exceeds the `ALLOCATE`value, but that it would never place data in the overflow area when the length of the data does not exceed the `ALLOCATE`value.


## Example 3: VARCHAR with ALLOCATE(15) Part 2

This examples builds on Example 2 with another simple change. A second column has been introduced to the `AUTHORS` table to hold the number of books the author has written though note that this new column isn't being populated in this example.

```sql
drop table authors ; 

create table authors ( name varchar(50) allocate(15), num_books integer ) ;

insert into authors (name) values('Agatha Christie');
insert into authors (name) values('J. K. Rowling');
insert into authors (name) values('William Shakespeare');

select table_schema, table_name, number_rows, overflow, data_size, variable_length_size from qsys2.systablestat where table_name='AUTHORS';
```

The results are now as I expected. One row has data stored in the overflow area.


## Example 4: CLOB with no ALLOCATE value

In this example I'm creating a second new table called `BOOK_QUOTES` to contain a list of my favourite book quotes. This table contains two columns; the first is the book name, and note that I am allocating the full column length to the fixed-length area to ensure this doesn't affect the CLOB results and the second column is the book quote itself, a 1KB CLOB. I have not specified the `ALLOCATE` keyword so the default `ALLOCATE(0)` is used.

I then insert four rows into the table.

The final select statement on the `QSYS2.SYSTABLESTAT` view will show me how many rows are contained in the overflow area.

```sql
create table book_quotes ( book varchar(50) allocate(50) , quote clob(1k) ) ;

insert into book_quotes (book, quote) values('Hamlet','To be, or not to be: that is the question: Whether ’tis nobler in the mind to suffer the slings and arrows of outrageous fortune, or to take arms against a sea of troubles, and by opposing end them. To die: to sleep;');
insert into book_quotes (book, quote) values('Julius Caesar','Cowards die many times before their deaths; the valiant never taste of death but once.');
insert into book_quotes (book, quote) values('Romeo and Juliet','Young men’s love then lies not truly in their hearts, but in their eyes.');
insert into book_quotes (book, quote) values('Twelfth Night','…be not afraid of greatness. Some are born great, some achieve greatness, and some have greatness thrust upon ’em.');

select table_schema, table_name, number_rows, overflow, data_size, variable_length_size from qsys2.systablestat where table_name='BOOK_QUOTES';
```

The result is as I expected. All four rows have data stored in the overflow area.


## Example 5: CLOB with ALLOCATE(200)

This examples builds on Example 4 with one simple change. I've added the `ALLOCATE` keyword to the `QUOTE` column. I'm expecting the majority of the names to be 200 characters or less, which means that most of the quotes should be stored in the fixed-length portion of the row and therefore only require one read.

Note that only one quote has a name that exceeds 200 characters.

```sql
drop table book_quotes ; 

create table book_quotes ( book varchar(50) allocate(50) , quote clob(1k) allocate(200) ) ;

insert into book_quotes (book, quote) values('Hamlet','To be, or not to be: that is the question: Whether ’tis nobler in the mind to suffer the slings and arrows of outrageous fortune, or to take arms against a sea of troubles, and by opposing end them. To die: to sleep;');
insert into book_quotes (book, quote) values('Julius Caesar','Cowards die many times before their deaths; the valiant never taste of death but once.');
insert into book_quotes (book, quote) values('Romeo and Juliet','Young men’s love then lies not truly in their hearts, but in their eyes.');
insert into book_quotes (book, quote) values('Twelfth Night','…be not afraid of greatness. Some are born great, some achieve greatness, and some have greatness thrust upon ’em.');

select table_schema, table_name, number_rows, overflow, data_size, variable_length_size from qsys2.systablestat where table_name='BOOK_QUOTES';
```

The result is as I expected. One row has data stored in the overflow area.


## Example 6: CLOB change ALLOCATE value

This examples builds on Example 5 and adjusts the ALLOCATE value from 200 to 500, which means all rows will now fit in the allocated value.

```sql
alter table book_quotes alter column quote set data type clob(1k) allocate(500) ;

select table_schema, table_name, number_rows, overflow, data_size, variable_length_size from qsys2.systablestat where table_name='BOOK_QUOTES';
```

The results are as I expected. No rows have data stored in the overflow area.

What about if I reduce the ALLOCATE value?

```sql
alter table book_quotes alter column quote set data type clob(1k) allocate(75) ;

select table_schema, table_name, number_rows, overflow, data_size, variable_length_size from qsys2.systablestat where table_name='AUTHORS';
```

The results are as expected. Three rows have data stored in the overflow area.



## Monitor the Overflow Area

The following SQL command is useful at identifying an columns where the ALLOCATE value may need tweaking.

```sql
select c.table_schema, c.table_name, ts.number_rows, ts.overflow, c.column_name, c.data_type , c.length, c.inline_length, cs.average_column_length, cs.maximum_column_length, length_at_90th_percentile, cs.overflow_rows
from qsys2.syscolumns2 as c
left outer join qsys2.syscolumnstat as cs on c.table_schema=cs.table_schema and c.table_name=cs.table_name and c.column_name=cs.column_name
left outer join qsys2.systablestat as ts on c.table_schema=ts.table_schema and c.table_name=ts.table_name
where c.table_schema='MYLIB' and cs.OVERFLOW_ROWS is not null;
```


## Conclusions

My preferred approach when defining variable-length columns is as follows.

* Always specify the ALLOCATE keyword.
* Unless you know the column will rarely be populated then if the column length is 50 characters or less then set the ALLOCATE value to be the same as the column length.
* Use the `SYSCOLUMNSTAT` view to monitor the overflow area and tweak the `ALLOCATE` value accordingly.
* When using LOB columns consider forcing VARCHAR columns into the fixed area.

