---
layout: post
title: "Using GitHub Pages to Host My Docs"
date: 2024-03-04
modified_date: 2024-03-07
tags:
- GitHub Pages
- Jekyll
---

After following the standard GitHub Pages tutorial I went ahead and created a
new GitHub Pages repo to host my docs.

The following GitHub Pages repos helped me with ideas.
* [https://github.com/peterroelants/peterroelants.github.io/](https://github.com/peterroelants/peterroelants.github.io/)
* [https://github.com/qian256/qian256.github.io/](https://github.com/qian256/qian256.github.io/)

This pages is mostly documenting my preferences and changes.



## Table of Contents

Adding a table of contents to all articles was easy to achieve using the [allejo/jekyll-toc](https://github.com/allejo/jekyll-toc) repo.

I copied the `toc.html` file into my `_includes` directory and then created a `_layouts/post.html` file. My starting point for the `post.html` file was to copy the content from the same [file](https://github.com/jekyll/minima/blob/master/_layouts/post.html) held in the Minima repo.

I then amended my copy of the `post.html` file as follows.

```html
...
<div class="post-content e-content" itemprop="articleBody">
  {% raw %}{% include toc.html html=content %}{% endraw %}
  <hr>
  {% raw %}{{ content }}{% endraw %}
</div>
...
```

This outputs the table of contents at the top of every article, then outputs a horizontal line and then outputs the article content.



## Theme

I picked the `Minima` theme but decided to use the remote version, as opposed to
the GitHub version. The GitHub version is older and didn't allow me to  select
the `dark` skin that I wanted.

```yaml
# _config.yml
...
remote_theme: jekyll/minima
minima:
  skin: dark
...
```

* [GitHub: minima repo](https://github.com/jekyll/minima/tree/master)




## Favicon

Drop your `favicon.ico` file into the repo root directory.

Create a `_includes/custom-head.html` file.

```html
<link rel="shortcut icon" type="image/x-icon" href="favicon.ico">
```
