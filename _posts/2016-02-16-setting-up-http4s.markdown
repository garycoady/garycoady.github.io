---
layout: post
title:  "Setting up a http4s skeleton project"
categories: http4s
---
## Introduction

[http4s](https://http4s.org) provides a minimal interface for creating a HTTP
server using Scala. It tries to offer a typesafe and functional approach to
building applications with HTTP. On top of that, a user may then choose from
a number of backends on which their service can run (currently, the main
options are Blaze and servlets).

The documentation for http4s is fairly light at the moment, so I hope to
provide a series of posts here to go through the main features that will
be needed for creating a full application with the service.

First, we will look at the setup of an initial skeleton application.

## Build configuration

Set up a `build.sbt` like this:

{% highlight scala %}
val http4sVersion = "0.12.1"

libraryDependencies += "org.http4s" %% "http4s-dsl"          % http4sVersion
libraryDependencies += "org.http4s" %% "http4s-blaze-server" % http4sVersion
{% endhighlight %}

## The code

Then a basic skeleton for serving can look like:

{% highlight scala %}
import org.http4s._
import org.http4s.dsl._
import org.http4s.server.blaze.BlazeBuilder

object SkeletonService {
  val service = HttpService {
    case GET -> Root =>
      Ok("hello world")
  }
}

object SkeletonServer {
  def main(args: Array[String]): Unit = {
    BlazeBuilder.bindHttp(8080)
      .mountService(SkeletonService.service, "/")
      .run
      .awaitShutdown()
  }
}
{% endhighlight %}

`BlazeBuilder` is an object which provides a builder pattern for setting up a running
server which uses the Blaze (async IO) backend.

The code in `SkeletonServer` binds to all interfaces on port 8080, sets up a service at the
path `/`, starts the server, and waits until the server loop terminates.

When defining your service, `HttpService` expects a partial function from `Request` to
`Task[Response]`. That is, given a pattern match against the HTTP request on
the left-hand-side (which we will generally write using the http4s DSL), the
right-hand-side will return a `scalaz.concurrent.Task`, which, when run, will
return the response.

### Task in a Nutshell

If you haven't come across `Task` before, an initial intuition could be that it is
somewhat like `scala.concurrent.Future` in that it can encompass the idea of
asynchronous computation. Some obvious differences in behaviour are that it does
no memoization of the result, and it only does the computation to calculate the
result on request. So at the point where a `Task` is returned, no work has yet
been done to generate a response. This does not mean that we can't write code which
depends on the result - for a start, its `map` and `flatMap` methods allow the
response to be adapted.

## Upcoming

The above code is enough to get a service running. The next step will be to
parse HTTP requests, in particular, query and path parameters. We will look in
more detail at `HttpService`, and how the request pattern matching works.
