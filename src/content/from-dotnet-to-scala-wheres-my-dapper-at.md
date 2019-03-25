---
layout: post
title: From .NET to Scala - Where's my Dapper at?
image: img/medicine-cabinet.jpg
author: Yannick Meeus
draft: false
date: 2017-12-09T19:00:46.000Z
tags: 
  - scala
  - getting-started
  - archive
  - database-access
---

Connecting to a database is something you'll do for a considerable amount of applications.
From an in-memory cache to a globally distributed data model, pretty much every solution has some storage requirements.
Scala seems to encompass a wide range of paradigms. Some use it best in a Java++ way,
while others try and approach it with Category Theory front and centre of their endeavours
This means that "idiomatic" database access code depends on what idioms you embrace.

For now, I'm going to stay on the 'safe' end of the spectrum and have a crack
at some imperative data storage access code. So, let's start by putting
together a non-idiomatic version of talking to a database in Scala.

I will need a database. For this post I'll be using the official PostgreSQL
docker image, found [here](https://hub.docker.com/_/postgres/).

From the command line, let's start this one up:

`docker run --name me.meeus.db.pg -p 5432:5432 -d postgres`

The above snippet is aliased to `dbpg` in my bash profile. If you're, like me,
experimenting with PostgreSQL, then I'd highly recommend doing this at the least.
It defaults to a `postgres` user with a blank password, and a `postgres` database
will be made available to you when the container is up and running.

From here on out, I'll be interacting with my database entirely from within the
application. This includes creating some tables and performing some CRUD operations.

As always, I'll start off with the [g8 seed template](https://github.com/scala/scala-seed.g8),
and worked my way up from there.

I'm going to use [scalikejdbc](http://scalikejdbc.org/) as the library of choice.
Not because it's a fully-featured, highly abstract ORM, but explicitly because it is not.
It's a very thin layer over the native JDBC library, which gives you a lot of power
but can also be considered a foot gun. I like to be close to the metal when I'm
learning new languages because it hides nothing and breaks in spectacular ways if
it fails. This, in itself, provides valuable learning opportunities.

So we have ourselves a database, we have ourselves a library, let's write some code.

We'll need some dependencies declared in our build.sbt file. As a reminder,
build.sbt is pretty much your `.csproj` file. It's not XML based but uses a
custom DSL to capture your build dependencies, including project and external dependencies.

**Build.sbt**
```
libraryDependencies ++= Seq(
      "postgresql" % "postgresql" % "9.1-901.jdbc4",
      "org.scalikejdbc" %% "scalikejdbc" % "3.1.0",
      "io.jvm.uuid" %% "scala-uuid" % "0.2.3"
    )
```

The first line pulls in a PostgreSQL driver, the next one then pulls in the wrapper
library I'll use to interact with the database, and the third one pulls in a
native-like Scala UUID library (just because I like to use UUIDs).

**EntryPoint.scala**
```scala
package me.meeus.spikes.db

import scalikejdbc.AutoSession
import scalikejdbc.ConnectionPool

import scala.io.StdIn

object EntryPoint extends App {
  println("Running")
  println("Setting up Connection Pool")

  Class.forName("org.postgresql.Driver")
  ConnectionPool.singleton("jdbc:postgresql://localhost:5432/postgres", "postgres", "")
  implicit val session: AutoSession.type = AutoSession


  println("Press any key to exit...")
  StdIn.readLine()
}
```

So we've made a start on some of the plumbing required. I'm importing the driver
itself into an implicit context and set up the connection pool with my
connection string. I then load a database session into implicit scope.

We have a database, but we need a table. There are different ways
of managing database schemas but for my purposes, I'm assuming that
no table exists, and as part of my application bootstrapping I need to create one.

As a note, I'll be using the CQS pattern, which in C# translates into an
interface describing the intention of the resource interaction, and implementation of
said contract against a given data source. This has several benefits:
1. It keeps your database access requirements explicit in the right context
2. It keeps your data access classes small and self-contained
3. It adheres to the ISP (Interface Segregation Principle)
4. It's a nice segway into a functional composition way of working

**IInitializeAnAuthorTable.scala**
```scala
package me.meeus.spikes.db.commands

import scala.util.Try

trait IInitializeAnAuthorTable {
  def execute(): Try[Unit]
}
```

**InitializeAuthorTable.scala**
```scala
import me.meeus.spikes.db.commands.IInitializeAnAuthorTable
import scalikejdbc._
import scala.util.Try

class InitializeAuthorTable(implicit session: DBSession = AutoSession) extends IInitializeAnAuthorTable {
  private val createAuthorTable =
    sql"""
         |CREATE TABLE IF NOT EXISTS authors (
         |  id VARCHAR(36) NOT NULL PRIMARY KEY,
         |  first_name VARCHAR(64) NOT NULL,
         |  last_name VARCHAR(64) NOT NULL
         |)
    """.stripMargin

  override def execute(): Try[Unit] = {
    Try {
      createAuthorTable
        .execute()
        .apply()
    }
  }
}
```

There are quite a few things going on, but from the top:
1. A new class declaration for `InitializeAuthorTable`, implementing the corresponding trait.
It declares an implicit requirement for a session, which I mentioned before.
2. A multi-line SQL statement, denoted by the triple-quote syntax.
The `|` symbols are there for readability and are removed by the `stripMargin` call.
3. The implementation of the `execute` method, returning a `Try` of type Unit. `execute`,
followed by `apply` will evaluate the SQL statement, the `Try` will catch any exceptions if one does occur and the callee will have to deal with this accordingly.

Speaking of the callee, `EntryPoint` can now be changed to include the following:

```scala
println("Creating an author table...")
val createdTableResult: Try[Unit] = new InitializeAuthorTable().execute()
createdTableResult match {
  case Success(_) => println("Created Authors Table or already exists")
  case Failure(e) => println(s"Failed to create Authors Table with exception: $e")
}
```

It's rather self-explanatory, the pattern matching might throw some first-time scala users off a bit but in essence it's a `switch` statement on steroids.

Now that we have a table, we need to get some data in there. I'll use a case class to represent our data,
and a case class to describe our domain model. It's not a requirement to have two different models,
but as the domain model should not rely on the persistence technology, keeping them separated is considered best practice.

**models/Author.scala**
```scala
import io.jvm.uuid._

case class Author(
                   id: UUID,
                   firstName: String,
                   lastName: String
                 )
```
**postgres/models/DBAuthor.scala**
```scala
import me.meeus.spikes.db.models.Author
import io.jvm.uuid._
import scalikejdbc._
case class DBAuthor(
                    id: String,
                    firstName: String,
                    lastName: String
                  ) {
 def toAuthor: Author = {
   Author(UUID(this.id), this.firstName, this.lastName)
 }
}
object DBAuthor extends SQLSyntaxSupport[DBAuthor] {
  override val tableName = "authors"

  def apply(result: WrappedResultSet): DBAuthor = new DBAuthor(result.string("id"), result.string("first_name"), result.string("last_name"))

  def fromAuthor(author: Author): DBAuthor = DBAuthor(author.id.toString, author.firstName, author.lastName)
}
```

The domain model is kept as pure as possible, but the database model needs some more functionality to move back and forth between the two representations of an Author. It also implements an `apply` function, which will be used later on when I'll be fetching data from the table.

The trait and implementation used for saving an Author follow:

**ISaveAnAuthor.scala**
```scala
import me.meeus.spikes.db.models.Author

import scala.util.Try

trait ISaveAnAuthor {
  def execute(author: Author): Try[Unit]
}
```

**SaveAuthor**
```scala
import scala.util.Try
import scalikejdbc._

class SaveAuthor(implicit session: DBSession = AutoSession) extends ISaveAnAuthor {
  override def execute(author: Author): Try[Unit] = {
    val toSave = DBAuthor.fromAuthor(author)
    val statement =
      sql"""
           | INSERT INTO
           |   authors
           |   (
           |     id
           |     , first_name
           |     , last_name
           |   )
           | VALUES
           | (
           |    ${toSave.id}
           |    , ${toSave.firstName}
           |    , ${toSave.lastName}
           | )
      """.stripMargin
    Try {
      statement
        .execute()
        .apply()
    }
  }
}
```

The same pattern applies, but we're using first converting the `Author` into a `DBAuthor`, and then use some string interpolation to build up the SQL statement.

So far, so good, now I'll try to retrieve an author.

**IGetAnAuthorById**
```scala
import io.jvm.uuid._
import me.meeus.spikes.db.models.Author

trait IGetAnAuthorById {
  def execute(id: UUID): Option[Author]
}
```

**GetAuthorById**
```scala
import me.meeus.spikes.db.models.Author
import me.meeus.spikes.db.queries.IGetAnAuthorById
import scalikejdbc._
import io.jvm.uuid._
import me.meeus.spikes.db.postgres.models.DBAuthor

class GetAuthorById(implicit session: DBSession = AutoSession) extends IGetAnAuthorById {
  override def execute(id: UUID): Option[Author] = {
    val statement =
      sql"""
           |SELECT
           | id
           | , first_name
           | , last_name
           | FROM
           |   authors
           | WHERE
           |   id = ${id.toString}
         """.stripMargin

    statement
      .map(rs => DBAuthor(rs)) // Apply the result set to a DBAuthor
      .first() // Grab the first result if it exists
      .apply() // Run the query
      .map(_.toAuthor) // Map the DBAuthor back to an Author
  }
}
```

We're starting to notice a pattern here and at this point,
most of the interactions are quite easy to model, as long as
we're working against a single table. I'm not going to explore
multi-table operations yet, but I'll leave that for a future post.

Our EntryPoint.scala file now looks like this:

```scala
...
  println("Creating an author table...")
  val createdTableResult: Try[Unit] = new InitializeAuthorTable().execute()
  createdTableResult match {
    case Success(_) => println("Created Authors Table or already exists")
    case Failure(e) => println(s"Failed to create Authors Table with exception: $e")
  }

  println("Insert a new author")
  val authorToSave = Author(UUID.random, "yannick", "meeus")
  val savedAuthor = new SaveAuthor().execute(authorToSave)

  savedAuthor match {
    case Success(_) => println("Saved author")
    case Failure(e) => println(s"Failed to save author with exception: $e")
  }

  val retrievedAuthor = new GetAuthorById().execute(authorToSave.id)
  retrievedAuthor match {
    case Some(s) => println(s"Author found with first name: ${s.firstName} and last name: ${s.lastName}")
    case None => println(s"No author found with id: ${authorToSave.id.toString}")
  }
...
```

And that's that. I've created an Authors table, inserted some data and
retrieved said data. It isn't fancy, and it might not be idiomatic,
it's not functional-oriented, but it gets the job done.

As always, you can find the source for this post on
[GitHub](https://github.com/YannickMeeus/me.meeus.spikes.db).
If you have any questions or comments, reach out on Twitter.