---
title: "Neocons, a Clojure client for Neo4J REST API: Traversing the Graph"
layout: article
---

## About this guide

 * Graph traversals
 * Operations on paths
 * Path predicates


This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/neocons.docs).


## What version of Neocons does this guide cover?

This guide covers Neocons 1.0.0-rc3 and 1.0.x releases.


## Overview

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


TBD: more options, more examples


### Relationship traversal

To perform relationship traversal, use the `clojurewerkz.neocons.rest.relationships/traverse` function. It is very similar to its
counterpart that traverses nodes:

{% gist 32e0da63195ee8f24d22 %}

TBD: more options, more examples


### Working with Paths

To perform relationship traversal, use the `clojurewerkz.neocons.rest.paths/traverse` function. Several predicate functions
make it easy to determine whether a particular node or relationship belong to a path:

* `clojurewerkz.neocons.rest.paths/node-in?`,
* `clojurewerkz.neocons.rest.paths/relationship-in?`
* `clojurewerkz.neocons.rest.paths/included-in?`

Finally, `clojurewerkz.neocons.rest.paths/shortest-between` and `clojurewerkz.neocons.rest.paths/all-shortest-between` calculate and return
the shortest path(s) between two nodes.

TBD: examples for all these


### Checking if a path exists

Another common operation is checking whether a path between two nodes exists at all.

`clojurewerkz.neocons.rest.paths/exists-between?` checks whether there is a path from node A to node B:

{% gist 13d2b968f5973c429a04 %}

Relationship types that can be used (followed) during traversal are given via the `:relationships` option.

TBD: more path operations


## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [The Cypher query language](/articles/cypher.html)



## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Neocons mailing list](https://groups.google.com/forum/#!forum/clojure-neo4j)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.
