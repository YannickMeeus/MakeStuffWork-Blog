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

## Multiple Projects - One 'Solution'

**Soap-box alert** - I seriously dislike micro-services in their current
incarnation and the below is a *wee* bit of a rant.

Microservices are the big thing right now (or if we're to believe the interwebs
then serverless takes that crown), but they seem to be miss-applied liberally.

If everything is a micro-service, what are they 'micro' to,
comparatively speaking? How do you create a cohesive, but uncoupled system if
everything is deployed in isolation? Do we isolate the code as well?

These are all fun questions I tend to ask both myself and my team members
when someone brings up micro-services, or utters those damned words -
'I think we need a micro-service for that'.

It feels as if Microsoft actually had the right idea with the naming of
their project structure, at least at a high-level.

Projects are just that, individual projects that rely on one another, and
solutions group these together in a **Solution**. It's staring us in the face,
I know. Obvious as it is, few people actually consider what this means.

A solution solves a business problem, and it might require multiple projects to
do so. Projects could be separate deployable components, but all need to be
deployed in unison in order for the solution to work.

This means that you code share by keeping the code and the deployables in the
same solution. You then create one project per deployable, and package
those up separately. Voila, you now have your desired microservices, without
splitting up everything into separate VSC repositories, with separate CI
pipelines, etc.

## From .Nuspec to Dockerfile

### Tagging is a bit weird

## Bringing it all together