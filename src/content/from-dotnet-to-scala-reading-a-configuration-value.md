---
layout: post
title: From .NET to Scala - Reading a configuration value
image: img/building-blocks.jpg
author: Yannick Meeus
draft: false
date: 2017-08-15T10:58:14.000Z
tags: 
  - scala
  - getting-started
  - archive
---

Coming from .NET, we have what is called the `ConfigurationManager`,
your one-stop shop into the land of the app.config or web.config files.
.NET Core has introduced a [new](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration)
way of fetching and registering configuration, but I've yet to use that in anger.

`ConfigurationManager` is the entry-point API into a static configuration file,
by convention called `app.config` or `web.config`. By default, you access
either the `ConnectionStrings` or `AppSettings` nodes, with the latter containing
pretty much every configuration value under the sun that is not a connection string.
More advanced use-cases are supported, such as [custom configuration sections](https://msdn.microsoft.com/en-us/library/2tw134k3.aspx),
but the basic usage is the two fore mentioned default sections.

So what does Scala have to offer with regards to configuration value access?
One of the most obvious libraries I came across was [TypeSafe - Config](https://github.com/typesafehub/config).
Authored by TypeSafe (rebranded as Lightbend), it's a no-nonsense configuration library with
a [clear API](https://github.com/typesafehub/config#api-example).

So, let's see this library in action. I started out with a simple SBT project, willed
into existence using `sbt new` and the [scala-seed giter8 template](https://github.com/scala/scala-seed.g8)

```sbt
sbt new scala/scala-seed.g8
```

This gives me a nicely structured Hello-World console application.
I've removed the Dependencies.Scala class, as I don't want any more
noise than I need to at this point.
So all dependencies will be declared in the multi-project `build.sbt`
file for now.

Speaking of which, this is how I add the dependency to
`config` in `build.sbt`:

```sbt
libraryDependencies += "com.typesafe" % "config" % "1.3.1"
```

I then add a new file in my project, called `application.conf`.
Where you put this file is quite important, as there are some conventions you need to be aware of.
The following folder/file structure is recommended:

![Configuration-Spike---Example-Folder-Structure](/img/configuration-spike-example-folder-structure.png)

This will ensure that your application will find the right configuration file,
without you having to manually link to a path or a resource.

Now onto the actual code, I've created an `EntryPoint` object, extending `App`.
For those coming from .NET, this is your `Program.cs` with a `Main` Method.
The name of the object is not important, but it needs to be an object and it
needs to extend App. There are ways around it, and you can have multiple App
extending objects, but to keep it simple just have one.

So let's have a look at the contents of both the configuration file and the EntryPoint:

## application.conf

```scala
appConfig = {
  startupMessage = "This welcome message originates from the application.conf file."
}
```

## EntryPoint.scala

```scala
package me.meeus.spikes.configuration
import com.typesafe.config.ConfigFactory

object EntryPoint extends App{
  val config = ConfigFactory.load()
  val firstMessage = config.getString("appConfig.startupMessage")
  println(firstMessage)
}
```

When I compile and run, the output should look something like this:

![Configuration-Spike---Output](/img/configuration-spike-output.png)

Success! It's not much, but it works. It's not very safe, though.
There are a myriad ways of dealing with missing application configuration settings,b
ut the most basic scenario - I request a value and one does not exist -
simply throws a `ConfigException$Missing` exception. It also does not feel
very functional as a library, and that is because it's not. It's a library
designed to be agnostic of what language you're using it in, as long as it
runs on the JVM you'll find a wrapper for it to access your configuration in an idiomatic way.

So next time, I'll take a look at Ficus, a Scala wrapper around Config, with the intention
of moving away from representing configuration values as primitive types and moving towards
strongly typed domain specific structures.