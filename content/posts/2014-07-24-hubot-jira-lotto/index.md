---
title: Pick assignee for JIRA issues with hubot-jira-lotto
slug: "hubot-jira-lotto"
description: I published a Hubot script to pick assignee for JIRA issues.
date: "2014-07-24T10:30:00+09:00"
public: true
tags: ["hubot","script","jira"]
alternate: true
archives: ["2014-07"]
---
![](chat.png)

I published a [Hubot] script to pick assignee for [JIRA] issues.

**[ngs/hubot-jira-lotto]**

```sh
npm install --save hubot-jira-lotto
```

<!--more-->

Commands
--------

```
hubot pick (an) assignee (for) <ISSUE-NUMBER> from <ASSIGNEE-GROUP>
```

This command picks an assignee for the issue randomly from `<ASSIGNEE-GROUP>`.

Random logic is weighted with number of issues in same project already assigned to.

Configuration
-------------



This Hubot script require **Admin priviledges** for the login account.

Please add the login account to *administrators* group.


Converting JIRA username to chat username to mention
----------------------------------------------------

If you use different username in your adapter (Campfire, HipChat, Slack ...) and JIRA, you can define converter method in `robot` instance.

eg: map `ngs` to `atsushi_nagase`



[ngs/hubot-jira-lotto]: https://github.com/ngs/hubot-jira-lotto
[Hubot]: https://hubot.github.com/
[JIRA]: https://www.atlassian.com/software/jira

