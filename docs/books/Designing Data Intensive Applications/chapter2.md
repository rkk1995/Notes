# 2. Data Models and Query Languages

Most applications are built by layering one data model on top of another. Each layer hides the complexity of the layers below by providing a clean data model. These abstractions allow different groups of people to work effectively.

## Relational model vs document model

The roots of relational databases lie in _business data processing_, _transaction processing_ and _batch processing_.

The goal was to hide the implementation details behind a cleaner interface.

_Not Only SQL_ has a few driving forces:

* Greater scalability
* preference for free and open source software
* Specialised query optimisations
* Desire for a more dynamic and expressive data model

**With a SQL model, if data is stored in a relational tables, an awkward translation layer is needed, this is called _impedance mismatch_.**

JSON model reduces the impedance mismatch and the lack of schema is often cited as an advantage.

JSON representation has better _locality_ than the multi-table SQL schema. All the relevant information is in one place, and one query is sufficient.

In relational databases, it's normal to refer to rows in other tables by ID, because joins are easy. In document databases, joins are not needed for one-to-many tree structures, and support for joins is often weak.

If the database itself does not support joins, you have to emulate a join in application code by making multiple queries.

**The main arguments in favour of the document data model are schema flexibility, better performance due to locality, and sometimes closer data structures to the ones used by the applications. The relation model counters by providing better support for joins, and many-to-one and many-to-many relationships.**

If the data in your application has a document-like structure, then it's probably a good idea to use a document model. The relational technique of _shredding_ can lead unnecessary complicated application code.

The poor support for joins in document databases may or may not be a problem. Changes to records in document databases oftentimes have to rewrite the entire document.

If you application does use many-to-many relationships, the document model becomes less appealing. Joins can be emulated in application code by making multiple requests. Using the document model can lead to significantly more complex application code and worse performance.

### Schema flexibility

Most document databases do not enforce any schema on the data in documents. Arbitrary keys and values can be added to a document, when reading, **clients have no guarantees as to what fields the documents may contain.**

Document databases are sometimes called _schemaless_, but maybe a more appropriate term is _schema-on-read_, in contrast to _schema-on-write_.

Schema-on-read is similar to dynamic (runtime) type checking, whereas schema-on-write is similar to static (compile-time) type checking.

The schema-on-read approach if the items on the collection don't have all the same structure (heterogeneous)

* Many different types of objects
* Data determined by external systems

### Data locality for queries

If your application often needs to access the entire document, there is a performance advantage to this _storage locality_.

The database typically needs to load the entire document, even if you access only a small portion of it. On updates, the entire document usually needs to be rewritten, it is recommended that you keep documents fairly small.

### Convergence of document and relational databases

PostgreSQL does support JSON documents. RethinkDB supports relational-like joins in its query language and some MongoDB drivers automatically resolve database references. Relational and document databases are becoming more similar over time.

## Query languages for data

SQL is a _declarative_ query language. In an _imperative language_, you tell the computer to perform certain operations in order.

In a declarative query language you just specify the pattern of the data you want, but not _how_ to achieve that goal.

A declarative query language hides implementation details of the database engine, making it possible for the database system to introduce performance improvements without requiring any changes to queries.

Declarative languages often lend themselves to parallel execution while imperative code is very hard to parallelise across multiple cores because it specifies instructions that must be performed in a particular order. Declarative languages specify only the pattern of the results, not the algorithm that is used to determine results.

## Graph-like data models

If many-to-many relationships are very common in your application, it becomes more natural to start modelling your data as a graph.

A graph consists of _vertices_ (_nodes_ or _entities_) and _edges_ (_relationships_ or _arcs_).

Well-known algorithms can operate on these graphs, like the shortest path between two points, or popularity of a web page.

There are several ways of structuring and querying the data. The _property graph_ model (implemented by Neo4j, Titan, and Infinite Graph) and the _triple-store_ model (implemented by Datomic, AllegroGraph, and others). There are also three declarative query languages for graphs: Cypher, SPARQL, and Datalog.

### Property graphs

Each vertex consists of:

* Unique identifier
* Outgoing edges
* Incoming edges
* Collection of properties (key-value pairs)

Each edge consists of:

* Unique identifier
* Vertex at which the edge starts (_tail vertex_)
* Vertex at which the edge ends (_head vertex_)
* Label to describe the kind of relationship between the two vertices
* A collection of properties (key-value pairs)

Graphs provide a great deal of flexibility for data modelling. Graphs are good for evolvability.
