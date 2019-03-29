---
layout: post
title: Cleaning up after Greenkeeper
image: img/greenkeeper.png
author: Yannick Meeus
draft: false
date: 2019-03-28T08:37:12Z
tags: 
  - tooling
  - bite-size
---

I've been using [Greenkeeper](https://greenkeeper.io/) for a fair bit of time now
and it never ceased to amaze me how *good* it actually is.

If I have one complaint, it's the fact it does not clean up
after itself. At least, it struggles when you take on a
dependency on a mono-repository - looking at you, Gatsby -
which is updated every 5th heartbeat.

So Greenkeeper fall over itself creating branch after
branch in order to help you out. Maybe this pull request,
no wait, the next one will surely be good.

This goes on at nauseum, until the mono-repo developers
go to sleep and I can comfortably update to the latest versions.

This leaves me with a ton of dangling branches against my
repositories. Since I am not a fan of silly branches, and love
keeping my repositories clean, I ~~COMPLETELY AND ON MY OWN~~ adapted
a quick command I found [here](https://coderwall.com/p/eis0ba/remove-a-thousand-stale-remote-branches-on-git):

```bash
git branch -r | awk -F/ '/\/greenkeeper/{print $2"/"$3}' | xargs -I {} git push origin :{}
```

It is slow, but glorious to see branch after branch get nuked into oblivion.