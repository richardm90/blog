
I ran the following on naiad.

```shell
isql -v myibmi <user> <password> -b -c -q -d\| < mysql.sql
```
mysql.sql contained


```shell
select * from qiws.qcustcdt
```

This produced the following.

```csv
CUSNUM|LSTNAM|INIT|STREET|CITY|STATE|ZIPCOD|CDTLMT|CHGCOD|BALDUE|CDTDUE
938472|"Henning "|"G K"|"4859 Elm Ave "|"Dallas"|"TX"|75217|5000|3|37.00|0
839283|"Jones   "|"B D"|"21B NW 135 St"|"Clay  "|"NY"|13041|400|1|100.00|0
392859|"Vine    "|"S S"|"PO Box 79    "|"Broton"|"VT"|5046|700|1|439.00|0
938485|"Johnson "|"J A"|"3 Alpine Way "|"Helen "|"GA"|30545|9999|2|3987.50|33.50
397267|"Tyron   "|"W E"|"13 Myrtle Dr "|"Hector"|"NY"|14841|1000|1|0|0
389572|"Stevens "|"K L"|"208 Snow Pass"|"Denver"|"CO"|80226|400|1|58.75|1.50
846283|"Alison  "|"J S"|"787 Lake Dr  "|"Isle  "|"MN"|56342|5000|3|10.00|0
475938|"Doe     "|"J W"|"59 Archer Rd "|"Sutter"|"CA"|95685|700|2|250.00|100.00
693829|"Thomas  "|"A N"|"3 Dove Circle"|"Casper"|"WY"|82609|9999|2|0|0
593029|"Williams"|"E D"|"485 SE 2 Ave "|"Dallas"|"TX"|75218|200|1|25.00|0
192837|"Lee     "|"F L"|"5963 Oak St  "|"Hector"|"NY"|14841|700|2|489.50|.50
583990|"Abraham "|"M T"|"392 Mill St  "|"Isle  "|"MN"|56342|9999|3|500.00|0
```
