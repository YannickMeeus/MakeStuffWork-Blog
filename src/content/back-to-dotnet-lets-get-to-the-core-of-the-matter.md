---
layout: post
title: Back to .NET - Let's get to the core of things
image: img/core-building.jpg
author: Yannick Meeus
draft: false
date: 2018-04-02T17:08:36.000Z
tags: 
  - dotnet
  - dotnet-core
---

My Scala journey was fraught with perils and short-lived as it turned out.
I've mostly been hacking away at TypeScript on Node.js.

In the spirit of making some changes in my life, and considering that change is the spice of life,
I'll be spending some time reading up on what's happened on the .NET side of things.
Core, and ASP.NET core as an extension has been all the rage, and it's exciting to see where it's going.

So I plan to do what I do best, build an industry B2B or C2B solution,
in .NET core, using the latest and greatest frameworks, and try and deploy
it to a semi-reliable hosting platform.

So first of all, I need to define some sort of pipeline and spend some
time spiking out how to best deploy both web-based solutions (views,
assets, APIs, etc.) and asynchronous workloads. [Docker](https://docs.docker.com/)
is definitely a packaging option that is slowly surpassing NuGet for endpoints,
but [Kubernetes (K8s)](https://kubernetes.io/)
and [Service Fabric](https://azure.microsoft.com/en-us/services/service-fabric/)
are still duking it out on the field of orchestration.

So my next project and my next post will be about
a **thing** that will live **somewhere** and do **something**.

Stay tuned!
