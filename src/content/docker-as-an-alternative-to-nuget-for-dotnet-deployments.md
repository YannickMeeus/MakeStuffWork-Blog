---
layout: post
title: Docker as an alternative to nuget packaging for .NET deployments
image: img/gift-wrapped-present.jpg
author: Yannick Meeus
draft: true
date: 2019-04-22T07:33:36Z
tags: 
  - dotnet
  - dotnet-core
  - docker
---

## NuGet as a packaging strategy

I've used [NuGet](https://www.nuget.org/) extensively in the past - 
in conjunction with [Octopus Deploy](https://octopus.com/) - to package
up and deploy projects. NuGet has existed for a while now, but has historically
always been a package manager for shared libraries, and not necessarily for
deployable components.

When Octopus Deploy came on the scene, a shift occurred in how we regarded
our NuGet feeds and they became first-class citizens in our CI/CD pipelines.
We shoved all of our deployables (Windows Services,
Web and API focused back-ends, job schedulers, ..) into NuGet, and used it as
our primary artifact feed.

Fast-forward a couple of years - most of which I haven't been working on
.NET based projects - and I felt like experimenting and applying what I've
experienced on the other side of the fence on my beloved .NET (core).

## Multiple... Projects

## From .Nuspec to Dockerfile

### Tagging is a bit weird

## Bringing it all together