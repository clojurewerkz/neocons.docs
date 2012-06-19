---
title: "Neocons, a Clojure client for Neo4J REST API: Populating the Graph"
layout: article
---

## About this guide

 * Creating nodes
 * Connecting nodes by creating relationships
 * Node and relationship attributes
 * Indexing of nodes
 * Indexing of relationships
 * Deleting nodes
 * Deleting relationships


This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/neocons.docs).


## What version of Neocons does this guide cover?

This guide covers Neocons 1.0.0-rc1 and 1.0.x releases.



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


## Node and relationship attributes

TBD


## Indexes

TBD


### Indexing of nodes

TBD


### Indexing of relationships

TBD


### Fetching nodes via index

TBD


### Fetching relationships via index

TBD


## Deleting nodes

TBD


## Deleting relationships

TBD


## Updating nodes

TBD





## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Neocons mailing list](/)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.



## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Traversing the graph](/articles/traversing.html)
 * [The Cypher query language](/articles/cypher.html)



## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Neocons mailing list](https://groups.google.com/forum/#!forum/clojure-neo4j)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.
