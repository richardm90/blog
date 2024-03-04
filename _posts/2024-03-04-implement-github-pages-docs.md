---
layout: post
title: "Using GitHub Pages to Host My Docs"
date: 2024-03-04
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
