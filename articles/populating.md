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
 * Performing batch operations via Neo4J REST API


This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/neocons.docs).


## What version of Neocons does this guide cover?

This guide covers Neocons 1.0.x.



## Creating nodes

Nodes are created using the `clojurewerkz.neocons.rest.nodes/create` function:

{% gist 0bb48f1ea71864e42f0e %}

Nodes typically have properties. They are passed to `clojurewerkz.neocons.rest.nodes/create` as maps:

{% gist 2968df2fc93b08a69c88 %}

### Efficiently creating a large number of nodes

It is possible to efficiently insert a large number of nodes (up to hundreds of thousands of millions) in a single request using
the `clojurewerkz.neocons.rest.nodes/create-batch` function:

{% gist 13f1c5ca92a304a46ffd %}

It returns a lazy sequence of results, so if you want to retrieve them all at once, force the evaluation with
[clojure.core/doall](http://clojuredocs.org/clojure_core/clojure.core/doall).


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

Nodes and relationships in Neo4J have properties (attributes). It is possible for a node or relationship to not have any properties. The semantics of properties
varies from application to application. For example, in a social application nodes that represent people may have `:name` and `:date-of-birth` properties,
while relationships may have `:created_at` properties that store a moment in time when two people have met.

### Node properties

Nodes properties are passed to the `clojurewerkz.neocons.rest.nodes/create` function when a ndoe is created. In the following example, a node is created with
two properties, `:url` and `:domain`:

{% gist 2968df2fc93b08a69c88 %}


### Updating node properties

Node properties can be updated using `clojurewerkz.neocons.rest.nodes/update` that takes a node or node id and a map of properties:

{% gist e87b49d6f38b8241bbfb %}

It is also possible to set a single property with `clojurewerkz.neocons.rest.nodes/set-property`:

{% gist 54888e2e7db956985794 %}


### Relationship properties

Relationship properties are very similar to node properties. They are passed to the `clojurewerkz.neocons.rest.relationship/create` function when a relationship is created.
In the following example, a relationship between two nodes is created with two properties, `:link-text` and `:created-at`:

{% gist 15f5b75267b2387096fd %}


### Updating relationship properties

Relationships properties can be updated using `clojurewerkz.neocons.rest.relationships/update` that takes a node or node id and a map of properties:

{% gist 7d66d0bf9db06553dc16 %}

It is also possible to set a single property with `clojurewerkz.neocons.rest.nodes/set-property`:

{% gist 2009f34ebe2becb48475 %}



## Indexes

Indexes are data structures that data stores maintain to make certain queries significantly faster. Nodes and relationships in Neo4J are typically retrieved by
id but this is not always convenient. Often what's needed is a way to efficiently retrieve a node by email or URL or other attribute other than `:id`. Indexes
make that possible. In this sense Neo4J is not very different from other databases.

You can index nodes and relationships and have as many indexes as you need (within the limit of Neo4J server disk and RAM resources).

### Indexing of nodes

Before nodes can be indexed, an index needs to be created. Neo4J has a feature called [automatic indexes](http://docs.neo4j.org/chunked/milestone/rest-api-auto-indexes.html) but it may be disabled via server configuration, so typically it is a good idea to just create an index and use it.

`clojurewerkz.neocons.rest.nodes/create-index` is the function to use to create a new index for nodes. Indexes can be created with a specific *configuration*: it determines
whether it is a regular or full text search index and allows for specifying [additional index parameters](http://docs.neo4j.org/chunked/milestone/indexing-create-advanced.html) (like analyzer for full text search indexes).

{% gist 1e488af3c84d41bc7660 %}

{% gist 9346f92533c865e3959e %}

To add a node to an index, use `clojurewerkz.neocons.rest.nodes/add-to-index`. To remove a node from an index, use `clojurewerkz.neocons.rest.nodes/delete-from-index`.

{% gist d1c280b81329402b4172 %}

To add a node to an index [as unique](http://docs.neo4j.org/chunked/stable/rest-api-unique-indexes.html), pass one more argument to `clojurewerkz.neocons.rest.nodes/add-to-index`:

{% gist cb6e53040b5777240b9c %}

To look a node up in an exact match (not full text search) index, use `clojurewerkz.neocons.rest.nodes/find`:

{% gist efcc86dff27c88e2f599 %}

It returns a (possibly empty) collection of nodes found. There is also a similar function, `clojurewerkz.neocons.rest.nodes/find-one`, that works just like `find`
but assumes there only ever going to be a single node with the given key in the index, so it can be returned instead of a collection with the only value.

With full text search indexes, the function to use is `clojurewerkz.neocons.rest.nodes/query`:

{% gist 3247533ad751946fd55b %}


### Indexing of relationships

Similarly to node indexes, relationship indexes typically need to be created before they are used.
`clojurewerkz.neocons.rest.nodes/create-index` is the function to use to create a new relationship index. Just like node Indexes, relationship ones can be created with
a specific configuration.

{% gist 5099d4c99845bb9fd390 %}

{% gist 54def19b3eda74eca0a8 %}

To add a relationship to an index, use `clojurewerkz.neocons.rest.relationships/add-to-index`. To remove a relationship from an index, use `clojurewerkz.neocons.rest.relationships/delete-from-index`.

{% gist 69af83b409b2b0ac0961 %}

To add a relationship to an index [as unique](http://docs.neo4j.org/chunked/stable/rest-api-unique-indexes.html), pass one more argument to `clojurewerkz.neocons.rest.relationships/add-to-index`:

{% gist f3a475b9650e6ccc7c02 %}

To look a relationship up in an exact match (not full text search) index, use `clojurewerkz.neocons.rest.relationships/find`:

{% gist 78d46c020080ac620b6e %}

There is also a similar function, `clojurewerkz.neocons.rest.relationships/find-one`, that works just like `find` but assumes there
only ever going to be a single relationship with the given key in the index, so it can be returned instead of a collection with the only
value.

With full text search indexes, the function to use is `clojurewerkz.neocons.rest.relationships/query`:

{% gist 4e2881ce65a55c8980fc %}


## Deleting nodes

Nodes are deleted using the `clojurewerkz.neocons.rest.nodes/delete` function:

{% gist 2a7bec01c38ca5cf3800 %}

Note, however, that a node only can be deleted if they have no relationships. To remove all node relationships and the node itself,
use `clojurewerkz.neocons.rest.nodes/destroy`:

{% gist 1b07c86938fbc810d0db %}

`clojurewerkz.neocons.rest.nodes/delete-many` and `clojurewerkz.neocons.rest.nodes/destroy-many` are convenience functions that
delete or destroy multiple nodes.


## Deleting relationships

Nodes are deleted using the `clojurewerkz.neocons.rest.relationships/delete` function:

{% gist aa58a89314dcc1e00c4a %}

`clojurewerkz.neocons.rest.relationships/maybe-delete` will delete a relationship by id but only if it exists. Otherwise it
just does nothing. Unlike nodes, relationships can be deleted without any restrictions, so there is no `clojurewerkz.neocons.rest.relationships/destroy`.


## Performing batch operations via Neo4J REST API

Neocons 1.0.0-rc3 and later supports batch operations via Neo4J REST API. The API is fairly low level but is very efficient (can handle millions of
operations per request). To use it, you pass a collection of maps to `clojurewerkz.neocons.rest.batch/perform`:

{% gist 67e0f2748d5a81e2d7ac %}


## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Traversing the graph](/articles/traversing.html)
 * [The Cypher query language](/articles/cypher.html)



## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Neocons mailing list](https://groups.google.com/forum/#!forum/clojure-neo4j)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.
