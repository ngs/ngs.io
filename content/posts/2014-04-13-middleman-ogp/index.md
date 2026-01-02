---
title: middleman-ogp
description: Released an OpenGraph Protocol helper extension for Middleman.
date: "2014-04-13T02:20:00+09:00"
public: true
tags: ["middleman","opengraph"]
alternate: true
archives: ["2014-04"]
---
I released an OpenGraph Protocol helper extension for Middleman.

[ngs/middleman-ogp on GitHub][middleman-ogp]


In your Gemfile

```ruby
gem 'middleman-ogp'
```

<!--more-->

Activate the extension with adding this snippet in config.rb.



Add `ogp_tags` method and write `<meta>` tag in the block.



Page data (root key is `ogp`) will be merged to the default properties.



Outputs `<meta>` tags like this.



[middleman-ogp]: https://github.com/ngs/middleman-ogp
