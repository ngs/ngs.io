---
title: Fetch distributions and invalidate with hubot-cloudfront
slug: "hubot-cloudfront"
description: I published a Hubot script to fetch Amazon CloudFront distribution list and invalidate objects.
date: "2014-06-28T21:00:00+09:00"
public: true
tags: ["aws","amazon","cloudfront","middleman","hubot","script","hipchat","kaizenplatform"]
alternate: true
archives: ["2014-06"]
image: hipchat.png
---
I published a [Hubot] script to fetch [Amazon CloudFront] distribution list and invalidate objects.

**[ngs/hubot-cloudfront]**

```sh
npm install --save hubot-cloudfront
```

<!--more-->

List Distributions
------------------

Lists distributions with ID, domain name, status, comment and number of invalidation batches (if exists).



Shortcut:



Create invalidations
--------------------



You can use either ID and 0-based index number (on the left of distribution ID) to specify distribution.



Shortcut:



After creating invalidations, [Hubot] checks every 1 minute and notifies if completed.



List invalidations
------------------

List distribution with ID or 0-based index number.



Shortcut:



[Amazon S3]: http://aws.amazon.com/s3/
[Amazon CloudFront]: http://aws.amazon.com/cloudfront/
[ngs/hubot-cloudfront]: https://github.com/ngs/hubot-cloudfront
[Hubot]: https://hubot.github.com/

