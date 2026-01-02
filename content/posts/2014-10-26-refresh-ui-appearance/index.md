---
title: Apply UIAppearance immediately on the screen
slug: "refresh-ui-appearance"
description: How to apply appearance changes made with UIAppearance's proxy method  immediately
date: "2014-10-26T21:30:00+09:00"
public: true
tags: ["ios","uiappearance","objective-c","swift","uikit"]
alternate: true
archives: ["2014-10"]
image: screen.gif
---
I was looking for how to apply changes made with UIAppearance's proxy method immediately.

I found a solution in a library [UISS], that handles Stylesheets written with JSON format.

<!--more-->



This code detaches all subviews of `window` and attach them again.

I also tried `setNeedsDisplay` and `setNeedsLayout`, but didn't work. So I adapted this hack.

Here is Swift version.



[UISS]: https://github.com/robertwijas/UISS

