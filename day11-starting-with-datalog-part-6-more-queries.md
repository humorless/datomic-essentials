# Starting with Datalog - Part 6 (More Queries)

The Datomic database can be seen as a collection of **datoms**, where a datom is a 5-element tuple `[eid attr val tx op]`.

Previously, our queries focused solely on querying the `eid` or `val` elements. However, Datalog can also query other elements, particularly **attributes** (`attr`) and **transactions** (`tx`), which can provide valuable insights.

## Querying Attributes

Let’s first delve into attributes:

In SQL databases, the schema is always defined at the table level, where the table is the smallest unit of schema. In Datomic, the smallest unit of schema is not the table but the column. Each column schema, also called an attribute, can be defined independently.

### Example

To query all schemas corresponding to `person` entities, we can try the following query. The clause `[?p :person/name]` matches `?p` to `person` entities, while `[?p ?attr]` matches all attributes of these entities.

```
[:find ?attr
 :where 
 [?p :person/name]
 [?p ?attr]]
```

Run this query at the [bottom of this page](https://www.learndatalogtoday.org/chapter/0).

```
?attr
68
69
70
```

The result shows numbers instead of meaningful attribute names. What’s going on?

Unlike SQL databases, Datomic stores attributes as **datoms**. Each user-defined attribute is also a data entity, and this entity is associated with a built-in attribute `:db/ident`.

Based on Datomic’s internal structure, we can modify the attribute query as follows:

```
[:find ?attr
 :where
 [?p :person/name]
 [?p ?a]
 [?a :db/ident ?attr]]
```

The result would be:

```
?attr
:person/born
:person/death
:person/name
```

## Querying Transactions

Let’s first clarify some terminology.

In conventional database terms, a transaction refers to: “a batch of operations grouped together to prevent **race conditions**. It either succeeds or fails as a whole.” However, in Datomic, a transaction is roughly synonymous with a database write operation.

Every time we write to the Datomic database, it writes several datoms. During this process, Datomic automatically creates a data entity to record this batch of write operations, and this entity is called a **transaction**. In other words, transactions in Datomic are queryable.

A typical transaction entity looks like this:

```
[13194139534390 50 #inst "2024-08-29T17:03:57.519-00:00" 13194139534390 true]
```

Where:

- The first element `13194139534390` is the entity ID of the transaction itself.
- `50` corresponds to the entity ID of `:db/txInstant`.
- `#inst "2024-08-29T17:03:57.519-00:00"` is the timestamp when the transaction occurred.
- For transaction entities, the fourth element is identical to the first element, as it records the transaction entity ID at the moment the datom was recorded.

### Example

To query the timestamp when "James Cameron" was set as the name of a `person` entity, we can use the following query:

```
[:find ?timestamp
 :where
 [?p :person/name "James Cameron" ?tx]
 [?tx :db/txInstant ?timestamp]]
```

### Exercise

- [ ] Complete the [More Queries](https://www.learndatalogtoday.org/chapter/4) section.

