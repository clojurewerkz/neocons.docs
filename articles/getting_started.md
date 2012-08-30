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

    [clojurewerkz/neocons "1.0.1"]

### With Maven

Add Clojars repository definition to your `pom.xml`:

{% gist 65642c4b53d26539e5f6 %}

And then the dependency:

    <dependency>
      <groupId>clojurewerkz</groupId>
      <artifactId>neocons</artifactId>
      <version>1.0.1</version>
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

### Nodes are just Clojure maps

The function returns a new node which is a Clojure record but for all intents and purposes should be treated and handled
as a map. In Neo4J, nodes have identifiers so `:id` key for created and fetched nodes is always set. Node identifiers
are used by various Neocons API functions.

Fetched nodes also have the `:location-uri` and `:data` fields. `:location-uri` returns a URI from which a node can be fetched
again with a GET request. Location URI is typically not used by applications. `:data`, however, contains node properties
and is very commonly used.


## Creating relationships

Now that we know how to create nodes, lets create two nodes representing two Web pages that link to each other and add a directed
relationship between them:

{% gist 708313e463a60e55e388 %}

Relationships can have properties, just like nodes:

{% gist 22edd0e9707c2ce7067c %}

### Relationships are just Clojure maps

Similarly to nodes, relationships are technically records with a few mandatory attributes:

 * `:id`
 * `:start`
 * `:end`
 * `:type`
 * `:data`
 * `:location-uri`

`:id`, `:data` and `:location-uri` serve the same purpose as with nodes. `:start` and `:end` return location URIs of nodes on both
ends of a relationship. `:type` returns relationship type (like "links" or "friend" or "connected-to").

Just like nodes, created relationships have `:id` set on them.

`clojurewerkz.neocons.rest.relationships/create-many` is a function that lets you creates multiple relationships for a node,
all with the same direction and type. It is covered in the [Populating the graph guide](/articles/populating.html).


## Fetching nodes

Now that we know how to populate a small graph, lets look at how you query it. The most basic operation is to fetch
a node by id with `clojurewerkz.neocons.rest.nodes/get`:

{% gist 22edd0e9707c2ce7067c %}

It returns a node value that is a Clojure map.


## Fetching relationships

`clojurewerkz.neocons.rest.relationships/get` fetches a single relationship by id:

{% gist a7039af62c858b95b024 %}

Neocons also provides other ways of fetching relationships (based on the start node and type, for example) that will be described
in detail the [Traversing the graph](/articles/traversing.html) guide. `clojurewerkz.neocons.rest.relationships/outgoing-for` and
`clojurewerkz.neocons.rest.relationships/incoming-for` are two such functions, lets take a look at them:

{% gist 20edac3ebc3a9d9c0294 %}
{% gist 9ff952ae3c92ecb2a5dd %}

Both accept a node and a collection of relationship types you are interested in as the `:types` option, returning a collection
of relationships.


## Using Cypher queries

One of the most powerful features of Neo4J is Cypher, a query language (like SQL) for querying, traversing and mutating (Neo4J Server 1.8+)
graphs. Cypher makes queries like "return me all friends of my friends" or "return me all pages this page links to that were updated
less than 24 hours ago" possible in a couple of lines of code. As such, operations and ad hoc queries in the Clojure REPL or Neo4J shell
with Cypher are very common.

Cypher also enables several operations Neo4J Server REST API does not provide to be executed efficiently. One common example is
the "multi-get" operation that returns a collection of nodes by ids.

Cypher queries are performed using `clojurewerkz.neocons.rest.cypher/tquery` and `clojurewerkz.neocons.rest.cypher/query` functions.
`clojurewerkz.neocons.rest.cypher/tquery` is more common because it returns data in a more convenient tabular form, while
`/query` returns columns and result rows separately.

Covering Cypher itself is out of scope for this tutorial so lets just take a look at a couple of examples. Here is how
to find all Amy's friends via Cypher:

{% gist 8affe3de7f02779fc992 %}

And here is how to get back usernames and ages of multiple people using Cypher:

{% gist 9ff952ae3c92ecb2a5dd %}

The latter query is roughly equivalent to

{% gist c2d579a7f94f79eb24bf %}

in SQL.

Cypher is fundamental to Neo4J is the most powerful (and easy to use) tool for many common cases. As such, Neocons documentation
has a whole guide dedicated to it: [The Cypher query language](/articles/cypher.html).


## Traversing the graph

To traverse a graph means to extract information from it, often by starting at a node and following 0 or more relationships. There
are certain well known algorithms that can be executed on graphs: for example, finding shortest path(s) between two nodes. A lot of Neo4J
power comes from support for some of those algorithms and flexible graph traversal in general.

### Traversal overview

Traversing the graph means following relationships between nodes and accumulating nodes (node traversal) or relationships (relationship
traversal) along the way. In the end, the client is returned a collection of nodes, relationships or *paths*. Path is a data structure which
has a start node, an end node, a length and collections of nodes and/or relationships.

Traversing supports various options that define whether returned node set should be unique or not, what order ([depth first](http://en.wikipedia.org/wiki/Depth-first_search)
or [breadth first](http://en.wikipedia.org/wiki/Breadth-first_search)) traversing should happen in,
when to terminate the traversal and so on.


### Node traversal

Node traversal with Neoncons is performed using the `clojurewerkz.neocons.rest.nodes/traverse` function. It takes several arguments:
a node to start traversing from, relationships to follow and additional options like what nodes to return:

{% gist 29db4ea258a66676c534 %}


### Relationship traversal

To perform relationship traversal, use the `clojurewerkz.neocons.rest.relationships/traverse` function. It is very similar to its
counterpart that traverses nodes:

{% gist 32e0da63195ee8f24d22 %}


### Working with paths

To perform a path traversal, use the `clojurewerkz.neocons.rest.paths/traverse` function. Several predicate functions
make it easy to determine whether a particular node or relationship belong to a path:

* `clojurewerkz.neocons.rest.paths/node-in?`,
* `clojurewerkz.neocons.rest.paths/relationship-in?`
* `clojurewerkz.neocons.rest.paths/included-in?`

Finally, `clojurewerkz.neocons.rest.paths/shortest-between` and `clojurewerkz.neocons.rest.paths/all-shortest-between` calculate and return
the shortest path(s) between two nodes.

For more examples, see the [Traversing the graph](/articles/traversing.html) guide.


### Checking if a path exists

Another common operation is checking whether a path between two nodes exists at all.

`clojurewerkz.neocons.rest.paths/exists-between?` checks whether there is a path from node A to node B:

{% gist 13d2b968f5973c429a04 %}

Relationship types that can be used (followed) during traversal are given via the `:relationships` option.


### Traversing vs Cypher queries

While traversing features are powerful, they predate a newer, more generic and even more powerful Neo4J Server feature: the Cypher query language.
With the introduction and several revisions on Cypher, in some situations traversing the graph is no longer necessary. Please keep this in mind.
That said, some problems are easier to solve using traversing than sophisticated Cypher queries.


## Wrapping up

Congratulations, you now can use Neocons to perform fundamental operations with Neo4J. Now you know enough to start building
a real application. There are many features that we haven't covered here; Cypher alone is worth a long guide full of examples.
They will be explored in the rest of the guides.

We hope you find Neocons reliable, consistent and easy to use. In case you need help, please ask on the [mailing list](https://groups.google.com/forum/#!forum/clojure-neo4j)
and follow us on Twitter [@ClojureWerkz](http://twitter.com/ClojureWerkz).


## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Populating the graph](/articles/populating.html)
 * [Traversing the graph](/articles/traversing.html)
 * [The Cypher query language](/articles/cypher.html)


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Neocons mailing list](https://groups.google.com/forum/#!forum/clojure-neo4j)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.
