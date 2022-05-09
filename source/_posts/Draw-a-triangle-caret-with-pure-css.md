---
title: Draw a triangle caret with pure CSS
date: 2022-05-03 01:24:09
categories: "Front-end"
tags:
- CSS
- Web
---
# Prefix
---
It is very common that we need a triangle caret on a web page to display as an arrow or other similar symbols. It can be achieved by using a png or svg file, or importing from third-party icon library such as [Font Awesome](https://fontawesome.com/icons/caret-up?s=solid)

![Caret icon](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/hexo-post-caret.jpg)

However, a more efficient low-carbon way is to draw it using CSS `border` attribute.

# A normal border
---
The most common use case of a border would be this:

```html
<div class="box"></div>

<style>
  .box {
    height: 100px;
    width: 100px;
    border: 5px solid #000;
  }
</style>
```

![plain border](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/hexo-caret-plain-border1.jpg)

If we take a look at the border shape by giving a wider width of it, we will see it looks like this:

```html
<div class="box"></div>

<style>
  .box {
    width: 200px;
    height: 200px;
    border-color: red blue yellow green;
    border-style: solid;
    border-width: 50px 50px 50px 50px;
  }
</style>
```

![bold border](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/hexo-colored-boder.jpg)

# Border with zero height and width
---

Instead of giving height and width to the box, when we set the width and height of the outer box to 0, the width of the border will stretch out the outer box, four sides of the borders will collapse together and each side of the border will be a triangle of 100px wide and 50px height, just like this:

```html
<div class="box"></div>

<style>
  .box {
    width: 0;
    height: 0;
    border-color: red blue yellow green;
    border-style: solid;
    border-width: 50px 50px 50px 50px;
  }
</style>
```

![triangle border](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/hexo-border-triangle.jpg)

# Display as a triangle
---
Now what we need to do is to hide three other sides of the border and only show one of it:
```html
<div class="box"></div>

<style>
  .box {
    width: 0;
    height: 0;
    border-color: red transparent transparent transparent;
    border-style: solid;
    border-width: 50px 50px 50px 50px;
  }
</style>
```

![single triangle border](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/hexo-border-single-caret.jpg)

#Other cases
If we try to set one side of the border width to be zero, we'll get and rectangle with three triangles:

```html
<div class="box"></div>

<style>
  .box {
    width: 0;
    height: 0;
    border-color: red blue yellow green;
    border-style: solid;
    border-width: 50px 50px 0 50px;
  }
</style>
```

![rectangle box](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/hexo-caret-3-sides.jpg)

# Use cases

This can be useful if we need a ribbon icon on page just like this:

![ribbon](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/20220506025424.png)

The solution is to draw a rectangle on top and add a sudo class `:after` below it to implement 3 sides of the border:


```html
<div class="box"></div>

<style>
  .box {
    width: 50px;
    height: 70px;
    background: red;
    position: relative;
  }

  .box:after {
    position: absolute;
    left: 0;
    top: 70px;
    content: '';
    border-width: 12px 25px;
    border-style: solid;
    border-color: green green transparent green;
  }
</style>
```

![ribbon](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/hexo-ribbon.jpg)

And we just need to set the color all red, and maybe add some text to make it a perfect ribbon.

![red ribbon](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/hexo-red-ribbon.jpg)