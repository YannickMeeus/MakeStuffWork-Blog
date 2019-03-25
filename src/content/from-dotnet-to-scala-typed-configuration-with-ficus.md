---
layout: post
title: From .NET to Scala - Typed Configuration with Ficus
image: img/camera-tweaking.jpg
author: Yannick Meeus
draft: false
date: 2017-09-12T14:21:49.000Z
tags: 
  - scala
  - getting-started
  - archive
---

Continuing on from my [previous post](/from-dotnet-to-scala-reading-a-configuration-value/),
I've been playing around with a configuration reading library called
[Ficus](https://github.com/iheartradio/ficus). This library allows you to read out configuration values,
compensate for missing values, map values to explicit types and you can quite easily implement custom
readers in order to map configuration structures to your desired custom shapes.

In its most basic form, Ficus is exposed as an extension method on the 
[Config](https://typesafehub.github.io/config/latest/api/com/typesafe/config/Config.html) interface,
contained in `com.typesafe.config`, and looks extremely straightforward at a glance.

```scala
// Application.conf contains the following at the root level:
// SimpleTextualEntry = "This is a simple Entry"

object EntryPoint extends App{
    val config = ConfigFactory.load()
    val simpleTextualEntry = config.as[String]("SimpleTextualEntry")
    println(s"Simple Entry: $simpleTextualEntry")
    // Output: Simple Entry: This is a simple Entry
}

```

There are a couple of things going on in the example above. But the essence is the
`.as[String]("X")` method call. This will find the "X" key in `application.conf`,
parse it as a string assign it to `simpleTextualEntry`.

This is no different from calling `getString` on `Config` though,
as the implementation is doing
[exactly that](https://github.com/ceedubs/ficus/blob/master/src/main/scala/net/ceedubs/ficus/readers/StringReader.scala).
But the fact it's accepting a [generic type parameter](https://docs.scala-lang.org/tour/generic-classes.html) opens the API
up to a host of possibilities.

You are still responsible for dealing with missing values,
which is where the generic type parameter comes into play.
If I, instead of expecting a `String`, expect an `Option[String]`,
we are back in functional land and can deal with missing values in a more elegant way.

Let's say we run the following:

```scala
  val c = config.as[Option[String]]("TotallyNotAConfigurationKey")
  println(s"Default Value: ${c.getOrElse("Value Not Found")}")
  
  //> Default Value: Value Not Found
  
  ```
  
  To me, that seems a lot cleaner compared to having to catch an exception,
  handling the outcome, defaulting a value declared in an outer scope, etc.
  
  Ficus also supports parsing out arbitrary types as the type parameter:
  
  ```scala
    // application.conf contains the following entry:
    // ComplexEntryWithVariousTypes = {
    //    TextualEntry = "This is a complex Entry",
    //    NumericEntry = 1254
    // }
    case class ComplexEntry(TextualEntry : String, NumericEntry : Int)
    
    object EntryPoint extends App {
        val config = ConfigFactory.load()
        val complexEntry = config.as[ComplexEntry]("ComplexEntryWithVariousTypes")
        println(s"Complex Entry represented as a Type, TextualEntry: ${complexEntry.TextualEntry} - NumericEntry: ${complexEntry.NumericEntry}")
        //> Complex Entry represented as a Type, TextualEntry: This is a complex Entry - NumericEntry: 1254
    }
  ```
  I've put together a couple of examples on [GitHub](https://github.com/YannickMeeus/me.meeus.spikes.configuration/tree/FicusExample),
  do have a look and, if you find anything awry with it, just raise an issue against it and I'll have a look.