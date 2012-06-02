---
title: "Neocons, a Clojure client for Neo4J REST API: Getting Started"
layout: article
---

## About this guide

This guide combines an overview of Neocons with a quick tutorial that helps you to get started with it.
It should take about 15 minutes to read and study the provided code examples. This guide covers:

 * Features of Neocons
 * Clojure and Neo4J Server version requirements
 * How to add Neocons dependency to your project
 * A very brief introduction to graph databases and theory
 * Basic operations (creating nodes and relationships, fetching nodes, using Cypher queries, traversing graph paths)

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/neocons.docs).


## What version of Neocons does this guide cover?

This guide covers Neocons 1.0.x.


## Neocons Overview

Neocons is an idiomatic Clojure client for the Neo4J Server REST API. It is simple and easy to use, strives to support
every Neo4J Server feature, makes working with Cypher queries a joy, takes a "batteries included" approach and is well maintained.


### What Neocons is not

Neocons is a REST API client, it currently does not support working with embedded Neo4J databases. Neocons was designed for
commercial products and using embedded open source Neo4J editions is not legal without obtaining a commercial license or
open sourcing your entire application.

Neocons is not an ORM/ODM. It does not provide graph visualization features, although this is an interesting area to explore
in the future versions. Neocons may or may not be Web Scale and puts correctness and productivity above sky high benchmarks.


## Supported Clojure versions

Neocons is built from the ground up for Clojure 1.3 and later.


## Supported Neo4J Server versions

Neocons supports Neo4J Server 1.6.0 and later but some features (in particular, in the Cypher language) may be specific to more recent versions.
The most recent Neo4J Server is thus recommended. We continuously test Neocons against bleeding edge server features and milestone releases,
so upgrade path should be smooth in most cases.


## Adding Neocons Dependency To Your Project

### With Leiningen

    [clojurewerkz/neocons "1.0.0-beta4"]

### With Maven

    <dependency>
      <groupId>clojurewerkz</groupId>
      <artifactId>neocons</artifactId>
      <version>1.0.0-beta4</version>
    </dependency>

It is recommended to stay up-to-date with new versions. New releases and important changes are announced [@ClojureWerkz](http://twitter.com/ClojureWerkz).


## Connecting to Neo4J

### Basics

Before you use Neocons, you need to connect to Neo4J Server. "Connect" here means "perform service discovery" since REST/HTTP services like Neo4J Server
do not have a concept of persistent stateful connection, but we use a more common term for database clients here. For that, you use
`clojurewerkz.neocons.rest/connect!` function:

{% gist 860e534d79471180a5f6 %}

Neocons uses implicit endpoint argument because most applications only ever use one Neo4J database.

### Authenticating

Neo4J REST API uses [HTTP authentication](http://www.ietf.org/rfc/rfc2617.txt) to authenticate clients. Authentication is mandatory in PaaS environments such as Heroku.
With Neocons, you can either pass credentials as user info in the connection URL:

{% gist fb98630152ad2e88ea82 %}

Alternatively, if the connection URL does not have user info but `NEO4J_LOGIN` and `NEO4J_PASSWORD` environment variables are set,
Neocons will use them.

Related Neo4J Server guide: [Securing Access to Neo4J Server](http://docs.neo4j.org/chunked/stable/security-server.html)


## A very short intro to graph databases

Graph is a data structure that represents connections (or lack of them) between things. Connected things
are called "nodes" or "vertices" and connections are called "relationships" or "edges". Nodes may have properties
(like person name or age), same for relationships (for example, when two people first met each other). There
may be more than one relationships between two nodes. Relationships are directed (have a start and an end; for example,
Web pages link to each other).


## Creating nodes

Nodes are created using the `clojurewerkz.neocons.rest.nodes/create` function:

{% gist 0bb48f1ea71864e42f0e %}

Nodes typically have properties. They are passed to `clojurewerkz.neocons.rest.nodes/create` as maps:

{% gist 2968df2fc93b08a69c88 %}

TBD


## Creating relationships

TBD


## Fetching nodes

TBD


## Fetching relationships

TBD


## Using Cypher queries

TBD


## Traversing the graph

TBD



## Wrapping up

TBD


## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * TBD


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Neocons mailing list](https://groups.google.com/forum/#!forum/clojure-neo4j)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.
