---
title: "Visualizing Bootstrap 4 breakpoints"
date: 2018-05-21
lastmod: 2021-03-21
draft: false
keywords: []
description: ""
tags: []
categories: []
---

When designing websites using Bootstrap 4, I found it challenging to figure out
which widths corresponded to which breakpoint. This post describes the HTML
layout I came up with to display a small banner (during development) to help me
visualize the different breakpoints.

<!--more-->

Bootstrap 4 comes with many sets of breakpoints for different screen widths and
devices - xs, sm, md, lg and xl. These affect the column widths in the grid
system. Bootstrap 4 grids need to be wrapped around a ``container`` element. The
``container`` can have ``row`` elements, which in turn will have ``col``
elements. The columns can have different widths depending on the specified
breakpoint.

I wanted a way to display a banner indicating which breakpoint was currently
active. This meant creating a single ``container`` with 5 different ``row``
div's. Each div would have some text (and styling) indicating a different
breakpoint. However, it would not be useful to display all of them at the same
time.

According to the Bootstrap 4 documentation, the way to hide elements is to use
the ``display`` classes. This took me a bit of time to wrap my head around, but
once I figured out that they work identical to how columns are setup - it
immediately clicked in my head!

Here is the resulting code I created,

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
            content="width=device-width, initial-scale=1.0, shrink-to-fit=no">

    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="css/bootstrap.min.css">
    <title>Responsive Banner</title>
</head>
<body>

    <div class="container-fluid">
        <div class="row d-block d-sm-none">
            <div class="col bg-primary text-white text-center">Extra Small</div>
        </div>
        <div class="row d-none d-sm-block d-md-none">
            <div class="col bg-secondary text-white text-center">Small</div>
        </div>
        <div class="row d-none d-md-block d-lg-none">
            <div class="col bg-success text-white text-center">Medium</div>
        </div>
        <div class="row d-none d-lg-block d-xl-none">
            <div class="col bg-info text-white text-center">Large</div>
        </div>
        <div class="row d-none d-xl-block">
            <div class="col bg-warning text-dark text-center">Extra Large</div>
        </div>
    </div>

</body>
</html>
```


In the first iteration of this, I used ``container`` instead of
``container-fluid``, but it feels better when the banner is end-to-end, so the
latter was selected.

