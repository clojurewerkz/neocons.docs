---
title: "Neocons, a Clojure client for Neo4J REST API: Populating the Graph"
layout: article
---

## About this guide

 * Creating nodes
 * Connecting nodes by creating relationships
 * Node and relationship attributes
 * Indexing of nodes (Legacy)
 * Indexing of relationships (Legacy)
 * Deleting nodes
 * Deleting relationships
 * Performing batch operations via Neo4J REST API
 * Performing operations in a transaction
 * Node Labels
 * Schema

This work is licensed under a <a rel="license"
href="http://creativecommons.org/licenses/by/3.0/">Creative Commons
Attribution 3.0 Unported License</a> (including images &
stylesheets). The source is available [on
Github](https://github.com/clojurewerkz/neocons.docs).


## What version of Neocons does this guide cover?

This guide covers Neocons 2.0 (including preview releases).



## Creating nodes

Nodes are created using the `clojurewerkz.neocons.rest.nodes/create` function:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; creates a node without properties
  (let [node (nn/create)]
    (println node)))
```

Nodes typically have properties. They are passed to
`clojurewerkz.neocons.rest.nodes/create` as maps:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; creates a node wit two properties
  (let [node (nn/create {:url "http://clojureneo4j.info" :domain "clojureneo4j.info"})]
    (println node)))
```

### Efficiently creating a large number of nodes

It is possible to efficiently insert a large number of nodes (up to
hundreds of thousands of millions) in a single request using the
`clojurewerkz.neocons.rest.nodes/create-batch` function:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; efficiently insert a batch of nodes, can handle hunreds of thousands or even millions of
  ;; nodes with reasonably small heaps. Returns a lazy sequence, for it with clojure.core/doall
  ;; if you want to parse & calculate the entire response at once.
  (let [nodes (nn/create-batch [{:url "http://clojureneo4j.info" :domain "clojureneo4j.info"}
                                {:url "http://clojuremongodb.info" :domain "clojuremongodb.info"}
                                {:url "http://clojureriak.info" :domain "clojureriak.info"}])]
    ;; printing here will force the lazy response sequence to be evaluated
    (println nodes)))
```

It returns a lazy sequence of results, so if you want to retrieve them all at once, force the evaluation with
[clojure.core/doall](http://clojuredocs.org/clojure_core/clojure.core/doall).


### Nodes are just Clojure maps

The function returns a new node which is a Clojure record but for all
intents and purposes should be treated and handled as a map. In Neo4J,
nodes have identifiers so `:id` key for created and fetched nodes is
always set. Node identifiers are used by various Neocons API
functions.

Fetched nodes also have the `:location-uri` and `:data`
fields. `:location-uri` returns a URI from which a node can be fetched
again with a GET request. Location URI is typically not used by
applications. `:data`, however, contains node properties and is very
commonly used.


## Creating relationships

Now that we know how to create nodes, lets create two nodes
representing two Web pages that link to each other and add a directed
relationship between them:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.relationships :as nrl]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [page1 (nn/create {:url "http://clojurewerkz.org"})
        page2 (nn/create {:url "http://clojureneo4j.info"})
        ;; a relationship that indicates that page1 links to page2
        rel   (nrl/create page1 page2 :links)]
    (println rel)))
```

Relationships can have properties, just like nodes:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.relationships :as nrl]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [amy (nn/create {:username "amy"})
        bob (nn/create {:username "bob"})
        rel (nrl/create amy bob :friend {:source "college"})]
    (println (nn/get (:id amy)))
    (println (nn/get (:id bob)))))
```

### Relationships are just Clojure maps

Similarly to nodes, relationships are technically records with a few
mandatory attributes:

 * `:id`
 * `:start`
 * `:end`
 * `:type`
 * `:data`
 * `:location-uri`

`:id`, `:data` and `:location-uri` serve the same purpose as with
nodes. `:start` and `:end` return location URIs of nodes on both ends
of a relationship. `:type` returns relationship type (like "links" or
"friend" or "connected-to").

Just like nodes, created relationships have `:id` set on them.

`clojurewerkz.neocons.rest.relationships/create-many` is a function
that lets you creates multiple relationships for a node, all with the
same direction and type. It is covered in the [Populating the graph
guide](/articles/populating.html).


## Node and relationship attributes

Nodes and relationships in Neo4J have properties (attributes). It is
possible for a node or relationship to not have any properties. The
semantics of properties varies from application to application. For
example, in a social application nodes that represent people may have
`:name` and `:date-of-birth` properties, while relationships may have
`:created_at` properties that store a moment in time when two people
have met.

### Node properties

Nodes properties are passed to the
`clojurewerkz.neocons.rest.nodes/create` function when a ndoe is
created. In the following example, a node is created with two
properties, `:url` and `:domain`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; creates a node wit two properties
  (let [node (nn/create {:url "http://clojureneo4j.info" :domain "clojureneo4j.info"})]
    (println node)))
```


### Updating node properties

Node properties can be updated using
`clojurewerkz.neocons.rest.nodes/update` that takes a node or node id
and a map of properties:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; creates a node and updates its properties
  (let [node (nn/create {:url "http://clojureneo4j.info"})]
    (println node)
    (nn/update (:id node) {:url "http://clojureneo4j.info" :domain "clojureneo4j.info"})
    (println "After the update: " node)))
```

It is also possible to set a single property with
`clojurewerkz.neocons.rest.nodes/set-property`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; creates a node and updates a single property
  (let [node (nn/create {:url "http://clojureneo4j.info"})]
    (println node)
    (nn/set-property (:id node) :domain "clojureneo4j.info")
    (println "After the update: " node)))
```


### Relationship properties

Relationship properties are very similar to node properties. They are
passed to the `clojurewerkz.neocons.rest.relationship/create` function
when a relationship is created.  In the following example, a
relationship between two nodes is created with two properties,
`:link-text` and `:created-at`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.nodes :as nrel]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [node1 (nn/create {:url "http://clojurewerkz.org"})
        node2 (nn/create {:url "http://clojureneo4j.info"})
        ;; creates a relationships of type :links with two properties
        rel   (nrel/create node1 node2 :links {:link-text "Neocons" :created-at "2012-07-03T19:40:27.269-00:00"})]
    (println rel)))
```


### Updating relationship properties

Relationships properties can be updated using
`clojurewerkz.neocons.rest.relationships/update` that takes a node or
node id and a map of properties:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.nodes :as nrel]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [node1 (nn/create {:url "http://clojurewerkz.org"})
        node2 (nn/create {:url "http://clojureneo4j.info"})
        rel   (nrel/create node1 node2 :links)]
    (println node)
    (nn/update (:id rel) {:link-text "Neocons" :created-at "2012-07-03T19:40:27.269-00:00"})
    (println "After the update: " node)))
```

It is also possible to set a single property with `clojurewerkz.neocons.rest.nodes/set-property`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.nodes :as nrel]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; sets a single relationship property
  (let [node1 (nn/create {:url "http://clojurewerkz.org"})
        node2 (nn/create {:url "http://clojureneo4j.info"})
        rel   (nrel/create node1 node2 :links)]
    (println node)
    (nn/set-property (:id rel) :link-text "Neocons")
    (println "After the update: " node)))
```



## Indexes

Indexes are data structures that data stores maintain to make certain
queries significantly faster. Nodes and relationships in Neo4J are
typically retrieved by id but this is not always convenient. Often
what's needed is a way to efficiently retrieve a node by email or URL
or other attribute other than `:id`. Indexes make that possible. In
this sense Neo4J is not very different from other databases.

You can index nodes and relationships and have as many indexes as you
need (within the limit of Neo4J server disk and RAM resources).

### Indexing of nodes

Before nodes can be indexed, an index needs to be created. Neo4J has a
feature called [automatic
indexes](http://docs.neo4j.org/chunked/milestone/rest-api-auto-indexes.html)
but it may be disabled via server configuration, so typically it is a
good idea to just create an index and use it.

`clojurewerkz.neocons.rest.nodes/create-index` is the function to use
to create a new index for nodes. Indexes can be created with a
specific *configuration*: it determines whether it is a regular or
full text search index and allows for specifying [additional index
parameters](http://docs.neo4j.org/chunked/milestone/indexing-create-advanced.html)
(like analyzer for full text search indexes).

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [idx (nn/create-index "by-name")]
    (println idx)))
```

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; creates a new full text search index with the given analyzer
  (let [idx (nn/create-index "imported" {:type "fulltext" :provider "lucene" :analyzer  "org.neo4j.index.impl.lucene.LowerCaseKeywordAnalyzer"})]
    (println idx)))
```

To add a node to an index, use
`clojurewerkz.neocons.rest.nodes/add-to-index`. To remove a node from
an index, use `clojurewerkz.neocons.rest.nodes/delete-from-index`.

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [idx  (nn/create-index "by-username")
        node (nn/create {:username "joe"})]
    (nn/add-to-index (:id node) (:name idx) "username" "joe")))
```

To add a node to an index [as
unique](http://docs.neo4j.org/chunked/stable/rest-api-unique-indexes.html),
pass one more argument to
`clojurewerkz.neocons.rest.nodes/add-to-index`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; create a unique index
  (let [idx  (nn/create-index "by-username" {:unique true})
        node (nn/create {:username "joe"})]
    ;; add a node to the index as unique
    (nn/add-to-index (:id node) (:name idx) "username" "joe" true)))
```

To look a node up in an exact match (not full text search) index, use `clojurewerkz.neocons.rest.nodes/find`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [puma  (nn/create {:name "Puma"  :hq-location "Herzogenaurach, Germany"})
        apple (nn/create {:name "Apple" :hq-location "Cupertino, CA, USA"})
        idx   (nn/create-index "companies")]
    (nn/delete-from-index (:id puma)  (:name idx))
    (nn/delete-from-index (:id apple) (:name idx))
    (nn/add-to-index (:id puma)  (:name idx) "country" "Germany")
    (nn/add-to-index (:id apple) (:name idx) "country" "United States of America")
    (println (nn/query (:name idx) "country:Germany"))))
```

It returns a (possibly empty) collection of nodes found. There is also
a similar function, `clojurewerkz.neocons.rest.nodes/find-one`, that
works just like `find` but assumes there only ever going to be a
single node with the given key in the index, so it can be returned
instead of a collection with the only value.

With full text search indexes, the function to use is `clojurewerkz.neocons.rest.nodes/query`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [puma  (nn/create {:name "Puma"  :hq-location "Herzogenaurach, Germany"})
        apple (nn/create {:name "Apple" :hq-location "Cupertino, CA, USA"})
        idx   (nn/create-index "companies")]
    (nn/delete-from-index (:id puma)  (:name idx))
    (nn/delete-from-index (:id apple) (:name idx))
    (nn/add-to-index (:id puma)  (:name idx) "country" "Germany")
    (nn/add-to-index (:id apple) (:name idx) "country" "United States of America")
    (println (nn/query (:name idx) "country:Germany"))))
```


### Indexing of relationships

Similarly to node indexes, relationship indexes typically need to be
created before they are used.
`clojurewerkz.neocons.rest.nodes/create-index` is the function to use
to create a new relationship index. Just like node Indexes,
relationship ones can be created with a specific configuration.

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.relationships :as nrel]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [node1 (nn/create {:url "http://clojureneo4j.info"})
        node2 (nn/create {:url "http://clojureneo4j.info"})
        rel   (nrel/create node1 node2 :links {:link-text "Neocons"})
        idx   (nrel/create-index "by-link-text" {:type "exact"})]
    (println idx)))
```

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.relationships :as nrel]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; creates a new full text search index with the given analyzer
  (let [idx (nrel/create-index "imported" {:type "fulltext" :provider "lucene" :analyzer  "org.neo4j.index.impl.lucene.LowerCaseKeywordAnalyzer"})]
    (println idx)))
```

To add a relationship to an index, use
`clojurewerkz.neocons.rest.relationships/add-to-index`. To remove a
relationship from an index, use
`clojurewerkz.neocons.rest.relationships/delete-from-index`.

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.relationships :as nrel]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [node1 (nn/create {:url "http://clojureneo4j.info"})
        node2 (nn/create {:url "http://clojureneo4j.info"})
        rel   (nrel/create node1 node2 :links {:link-text "Neocons"})
        idx   (nrel/create-index "by-link-text")]
    (nrel/add-to-index (:id rel) (:idx name) :link-text "Neocons")
    (println rel)))
```

To add a relationship to an index [as
unique](http://docs.neo4j.org/chunked/stable/rest-api-unique-indexes.html),
pass one more argument to
`clojurewerkz.neocons.rest.relationships/add-to-index`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.relationships :as nrel]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [node1 (nn/create {:url "http://clojureneo4j.info"})
        node2 (nn/create {:url "http://clojureneo4j.info"})
        rel   (nrel/create node1 node2 :links {:link-text "Neocons"})
        idx   (nrel/create-index "by-link-text" {:unique true})]
    (nrel/add-to-index (:id rel) (:idx name) :link-text "Neocons" true)
    (println rel)))
```

To look a relationship up in an exact match (not full text search)
index, use `clojurewerkz.neocons.rest.relationships/find`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.relationships :as nrel]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [node1 (nn/create {:url "http://clojureneo4j.info"})
        node2 (nn/create {:url "http://clojureneo4j.info"})
        rel   (nrel/create node1 node2 :links {:link-text "Neocons"})
        idx   (nrel/create-index "by-link-text")]
    (nrel/add-to-index (:id rel) (:idx name) :link-text "Neocons")
    (println (nrel/find (:name idx) :link-text "Neocons"))))
```

There is also a similar function,
`clojurewerkz.neocons.rest.relationships/find-one`, that works just
like `find` but assumes there only ever going to be a single
relationship with the given key in the index, so it can be returned
instead of a collection with the only value.

With full text search indexes, the function to use is
`clojurewerkz.neocons.rest.relationships/query`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.relationships :as nrel]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [node1 (nn/create {:url "http://clojureneo4j.info"})
        node2 (nn/create {:url "http://clojureneo4j.info"})
        rel   (nrel/create node1 node2 :links {:link-text "Neocons"})
        idx   (nrel/create-index "by-link-text" {:type "fulltext"})]
    (nrel/add-to-index (:id rel) (:idx name) :link-text "Neocons is an idiomatic Clojure client for the Neo4J Server REST interface")
    (println (nrel/query (:name idx) "link-text:*idiomatic*"))))
```


## Deleting nodes

Nodes are deleted using the `clojurewerkz.neocons.rest.nodes/delete` function:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; creates a node wit two properties
  (let [node (nn/create {:url "http://clojureneo4j.info" :domain "clojureneo4j.info"})]
    (nn/delete (:id node)))
```

Note, however, that a node only can be deleted if they have no
relationships. To remove all node relationships and the node itself,
use `clojurewerkz.neocons.rest.nodes/destroy`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; creates a node wit two properties
  (let [node (nn/create {:url "http://clojureneo4j.info" :domain "clojureneo4j.info"})]
    (nn/destroy node))
```

`clojurewerkz.neocons.rest.nodes/delete-many` and
`clojurewerkz.neocons.rest.nodes/destroy-many` are convenience
functions that delete or destroy multiple nodes.


## Deleting relationships

Nodes are deleted using the `clojurewerkz.neocons.rest.relationships/delete` function:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.nodes :as nrel]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [node1 (nn/create {:url "http://clojurewerkz.org"})
        node2 (nn/create {:url "http://clojureneo4j.info"})
        rel   (nrel/create node1 node2 :links {:link-text "Neocons"})]
    (nrel/delete rel)))
```

`clojurewerkz.neocons.rest.relationships/maybe-delete` will delete a
relationship by id but only if it exists. Otherwise it just does
nothing. Unlike nodes, relationships can be deleted without any
restrictions, so there is no
`clojurewerkz.neocons.rest.relationships/destroy`.


## Performing batch operations via Neo4J REST API

Neocons supports batch operations via Neo4J REST API. The API is
fairly low level but is very efficient (can handle millions of
operations per request). To use it, you pass a collection of maps to
`clojurewerkz.neocons.rest.batch/perform`:

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ;; efficiently executes a batch of operations in a single HTTP request, can handle hunreds of thousands or even millions of
  ;; nodes with reasonably small heaps. Returns a lazy sequence, for it with clojure.core/doall
  ;; if you want to parse & calculate the entire response at once.
  (let [ops [{:method "POST"
              :to     "/node"
              :body   {}
              :id     0}
             {:method "POST"
              :to     "/node"
              :body   {}
              :id     1}
             {:method "POST",
              :to     "{0}/relationships",
              :body   {:to   "{1}"
                       :data {}
                       :type "knows"}
              :id     2}]
        res (doall (b/perform ops))]
    ;; printing here will force the lazy response sequence to be evaluated
    (println res)))
```

## Performing Operations in a Transaction (Neo4J 2.0+)

Neocons 2.0 and later support Neo4j 2.0's [transactions over HTTP
API](http://docs.neo4j.org/chunked/milestone/rest-api-transactional.html). This
API only accepts a series of Cypher statements and can not be used
along side the other REST APIs.

To create a cypher statement, you can use the
`clojurewerkz.neocons.rest.transaction/statement` method which takes a
cypher query string and an optional map of parameters.

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.transaction :as tx]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [a (tx/statement "CREATE (n {props}) RETURN n" {:props {:name "Node 1"}})
        b (tx/statement "CREATE (n) RETURN n")]
    (println a)
    (println b)))
```

To execute a series of statements and commit them in a single
transaction with rollback on any cypher error, use the
`clojurewerkz.neocons.rest.transaction/in-transaction` method.

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.transaction :as tx]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (tx/in-transaction
    (tx/statement "CREATE (n {props}) RETURN n" {:props {:name "Node 1"}})
    (tx/statement "CREATE (n {props}) RETURN n" {:props {:name "Node 2"}})))
```


### Low level transaction API

For more complex operations, you can directly use the lower level REST API
for transactions.

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.transaction :as tx]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [transaction (tx/begin-tx)
       [_ result] (tx/execute transaction [(tx/statement "CREATE (n) RETURN ID(n)")])]
       (println result)
       (tx/commit transaction)))
```

Similarly you can use the `clojurewerkz.neocons.rest.transaction/rollback` method instead of
`clojurewerkz.neocons.rest.transaction/commit` to rollback the existing transaction.

For encapulating the commit-on-success and rollback-on-error pattern, you can use the
`clojurewerkz.neocons.rest.transaction/with-transaction` macro which has
parameters: `transaction`, `commit-on-success?` and a `body`. If `commit-on-success?`
is `false` then the user will have to manually commit the transaction. This can be
useful if you wanted to test changes made by cypger statement without actually committing
them to the database. If there are any errors in the body or any cypher errors in
any statement sent the server, the transaction will automatically be rolled back.

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.transaction :as tx]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (let [transaction (tx/begin-tx)]
    (tx/with-transaction
      transaction
      true
      (let [[_ result] (tx/execute transaction [(tx/statement "CREATE (n) RETURN ID(n)")])]
        (println result)))))

```

## Node Labels (Neo4J 2.0+)

Neo4j 2.0 added the concept of
[Labels](http://docs.neo4j.org/chunked/milestone/graphdb-neo4j-labels.html)
and `clojurewerkz.neocons.rest.labels` implements that functionality
[over HTTP
API](http://docs.neo4j.org/chunked/milestone/rest-api-node-labels.html).

An example which shows the basic functionality is listed below.

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.nodes :as nn]
            [clojurewerkz.neocons.rest.labels :as nl]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  ; create a node
  (let [node (nn/create {:name "Clint Eastwood"})]
    ; Add a single label to the node
    (nl/add node "Person")
    ; or, add multiple labels to the node
    (nl/add node ["Person" "Actor"])

    ; replace the current labels with new ones
    (nl/replace node ["Actor" "Director"])

    ; remove a particular label
    (nl/remove node "Person")

    ; listing all labels for a node
    (println (nl/get-all-labels node))

    ; list all labels for the whole graph db
    (println (nl/get-all-labels))

    ; getting all nodes with a label
    (println (nl/get-all-nodes "Actor"))

    ; get nodes by label and property
    (println (nl/get-all-nodes "Person" :name "Client Eastwood"))))

```

## Schema & Constraints (Neo4J 2.0+)

Since Neo4j 2.0, you can add schema meta information for speed
improvements or modelling benefits. They fall into two categories,
Indices and Constraints.  These features are very new and subject to
change.

### Indexing

To find out more about Indexing, see the Neo4j documentation
[here](http://docs.neo4j.org/chunked/milestone/rest-api-schema-indexes.html).

An example which shows the basic functionality is listed below.

``` clojure
(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.index :as ni]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")
  (do
    ; create an index on a label and a property name
    (ni/create "Person" :name)

    ; get all indices on a label
    (println (ni/get-all "Person"))

    ; drop an index on a label and a property name
    (ni/drop "Person" :name)))

```

### Constraints

To find out more about Constraints, see the Neo4j documentation
[here](http://docs.neo4j.org/chunked/milestone/rest-api-schema-constraints.html).

An example which shows the basic functionality is listed below.

``` clojure

(ns neocons.docs.examples
  (:require [clojurewerkz.neocons.rest :as nr]
            [clojurewerkz.neocons.rest.constraints :as nc]))

(defn -main
  [& args]
  (nr/connect! "http://localhost:7474/db/data/")

  (do
    ; create an uniqueness constraint on a label and a property name
    (nc/create-unique "Person" :name)

    ; get an existing uniquness constraint on a label and a property name
    (println (nc/get-unique "Person" :name))

    ; get all existing uniquness constraints on a label
    (println (nc/get-unique "Person"))

    ; get all existing constraints on a label
    (println (nc/get-all "Person"))

    ; get all existing constraints for the whole graph db
    (println (nc/get-all))

    ; drop an existing uniqueness constraint on a label and a property name
    (nc/drop "Person" :name)))

```

## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Traversing the graph](/articles/traversing.html)
 * [The Cypher query language](/articles/cypher.html)



## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on
Twitter or the [Neocons mailing
list](https://groups.google.com/forum/#!forum/clojure-neo4j)

Let us know what was unclear or what has not been covered. Maybe you
do not like the guide style or grammar or discover spelling
mistakes. Reader feedback is key to making the documentation better.
