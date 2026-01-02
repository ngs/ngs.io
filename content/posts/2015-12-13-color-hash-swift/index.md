---
title: Generate NSColor / UIColor based on Swift String
description: I've published a Swift library ColorHash, that generates UIColor and NSColor based on given string.
date: "2015-12-13T17:00:00+09:00"
public: true
tags: ["color","swift","coccoa","ios"]
alternate: true
archives: ["2015-12"]
image: og.png
---
![](screen.gif)

I've published a Swift library [ColorHash], that generates `UIColor` and `NSColor` based on given string.

**[ngs/color-hash.swift]**

```swift
import ColorHash

let str = "こんにちは、世界"
let saturation = CGFloat(0.30)
let lightness = CGFloat(0.70)

ColorHash(str, [saturation], [lightness]).color
```

This is a Swift port of a JavaScript library [Color Hash](https://github.com/zenozeng/color-hash).

<!--more-->

Install
-------

### CocoaPods

```rb
pod 'ColorHash'
```

### Carthage

```rb
github 'ngs/color-hash.swift'
```


[ngs/color-hash.swift]: https://github.com/ngs/color-hash.swift
[ColorHash]: https://github.com/ngs/color-hash.swift
