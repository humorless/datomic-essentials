# Performance Optimization -- Part 2

In several of my previous articles discussing primary keys, constraints, and indexes, I argued that Datomic is a **high-level database** that provides advanced semantics. This allows users to avoid overthinking many "important decisions" since Datomic makes them for you:

- No need to design primary keys; they are already handled.
- Constraints are simplified to just considering "unique values," as the rest are either decided by Datomic, discouraged by Datomic, or specific to business logic.
- Indexes require only consideration of AVET, with the other three taken care of.

However, **performance optimization** is inherently opposed to this high-level abstraction: while high-level semantics mean you can use the API without worrying about underlying details, performance optimization often requires some understanding of those details.

Since I don’t work at Datomic, I don’t have access to all of its inner workings. Nonetheless, based on my limited knowledge of Datomic's internals, here are two performance optimization techniques I've successfully used in practice.

## Direct Access to Datomic's Indexes

Consider the following two pieces of data, where the `:person/roles` attribute has `:db/cardinality` set to `many`, allowing one-to-many relationships:

```
  [{:db/id "temp-1"
    :person/name "John"
    :person/roles [:driver :student]}
   {:db/id "temp-2"
    :person/name "Mary"
    :person/roles [:driver :teacher]}]
```

If we already have the entity ID for the first record and want to find out which `:person/roles` are associated with it, how can we do it?

The most intuitive approach is to use a query:

```
(d/q '[:find ?r
       :in $ ?e
       :where [?e :person/roles ?r]]
      (db/db) entity-id)
```

However, its performance is average. By switching to **direct index access**, we can achieve faster results:

```
;;; Compare the speed of `d/datoms` and ordinary `d/q` query
(time (map :v
           (d/datoms (db/db) :eavt
                     entity-id
                     attr-id)))

;; => 
;; (out) "Elapsed time: 0.091166 msecs"
;; (:driver :student)

(time (d/q '[:find ?r
             :in $ ?e
             :where [?e :person/roles ?r]]
           (db/db) entity-id))

;; =>
;; (out) "Elapsed time: 2.878042 msecs"
;; #{[:student] [:driver]}

```

This simple comparison using Clojure's `time` function shows that `d/q` results can vary significantly due to possible internal optimizations, with execution times ranging from 2 ms to 8 ms. On the other hand, `datoms` delivers more consistent performance.

In summary, `d/datoms` is a much lower-level access method. While less user-friendly, it can unlock significantly better performance for critical use cases.

* [Example Code](https://github.com/humorless/ithome2024/blob/main/repl-sessions/day29.clj)

## For OLAP Queries, Use Change Data Capture (CDC)

Datomic queries are highly flexible and versatile. Some have even experimented with using Datomic as a graph database, finding that its query performance far outpaces SQL databases.

However, for OLAP scenarios, Datomic's query performance can lag behind Postgres by a factor of 10 to 20 (based on my subjective experience, not precise measurements). Unfortunately, even Postgres isn't the fastest in the OLAP world, particularly when compared to DuckDB.

So, if the query performance issue is precisely in OLAP analytics—Datomic's weak spot—what can we do? The solution is to use change data capture (CDC) to sync Datomic's contents directly to an SQL database. `plenish` is a library designed for this purpose, enabling real-time synchronization of Datomic data with Postgres.

* [plenish](https://github.com/lambdaisland/plenish)

## References

1. [Example of Datomic Raw Index Access on StackOverflow](https://stackoverflow.com/questions/39432061/)
2. [Using Datomic as a Graph Database](https://hashrocket.com/blog/posts/using-datomic-as-a-graph-database)
