# Constraints

In [Day25](./day25-primary-key-and-entity-id.md), we discussed enumerated types (enums). To some extent, enums are also a type of constraint—they limit the values that can be assigned to a specific field to a predefined set.

When it comes to SQL databases, the most commonly used constraints are the following three:

1. **Not Null Constraint** (not null)  
2. **Unique Constraint** (unique)  
3. **Foreign Key** (foreign key)  

In my experience, for simple use cases, the above three are sufficient to ensure a reasonable degree of system correctness. Next, let’s explore how these common applications correspond to practices in Datomic.

## Common Applications

### Primary Key

In SQL, setting a primary key for a specific column implies two constraints: the value must not be null, and it must be unique.

However, in Datomic, this issue doesn’t arise because the primary key is the entity ID. The entity ID is unique across the entire database, and every entity is guaranteed to have one. Thus, it inherently satisfies both properties.

### Not Null

Datomic does not have a corresponding syntax for the "not null" constraint because Datomic does not allow you to write a `NULL` value for any attribute. Therefore, such syntax is unnecessary.

How, then, can we represent that a specific entity lacks a particular attribute?

It’s simple—just don’t write it into the database. For example, in the **transaction data (tx-data)** below, we insert two items, each with two attributes:

```
[{:db/id item-1-id
  :line-item/product chocolate
  :line-item/quantity 1}
 {:db/id item-2-id
  :line-item/product whisky
  :line-item/quantity 2}]
```

If we want item-3 to lack the `:line-item/quantity` attribute, we can write it as follows:

```
[{:db/id item-3-id
  :line-item/product happy}]
```

Notice the advantage of this column schema: it provides clearer semantics.

### Unique

Refer to the example below. In SQL, a unique constraint can be applied to a column. In Datomic, you only need to add `:db/unique :db.unique/identity` to the corresponding attribute:

```
{:db/ident :user/uuid,
 :db/valueType :db.type/uuid,
 :db/doc "Unique user identifier",
 :db/cardinality :db.cardinality/one,
 :db/unique :db.unique/identity}
```

### Foreign Key

Datomic does not provide **concise syntax** to express foreign key semantics.

While foreign keys help ensure data consistency, they also introduce additional development overhead, especially during testing. Often, when creating simple integration tests, you might only want to generate test data for one table. However, with foreign keys, you might end up needing data for three tables instead.

As such, I believe Datomic deliberately avoids concise syntax for foreign key constraints because it does not encourage the traditional SQL semantic of foreign key.

## Advanced Applications

Regarding foreign keys mentioned in the common applications section, Datomic does not provide concise syntax for them. This statement is only half the story. With custom assertion functions, there is virtually no complex constraint that cannot be expressed.

As demonstrated in [Day18](./day18-datalog-part-13-sql-window-function.md), if complex aggregation queries are hard to express, custom aggregation functions can be used, enhancing semantic clarity. The same applies to constraints in Datomic. It provides two types of custom constraints: **Attribute Predicates** for strengthening attribute semantics and **Entity Specs** for imposing various restrictions on entities.

Here’s a terminology clarification: in the context of Clojure/Datomic, "constraints" and "specs" are nearly synonymous and are often used interchangeably.

### Attribute Predicates

Sometimes, we might declare an attribute’s data type as a string, but in practice, it always stores email addresses, which follow a specific format. In such cases, we can use attribute predicates to further enhance semantics.

Below is an example of checking that the string length for a name must be between 3 and 15 characters:

* Define a custom function `'datomic.samples.attr-preds/user-name?`:

```
(ns datomic.samples.attr-preds)

(defn user-name?
  [s]
  (<= 3 (count s) 15))
```

* Assign the custom function to `:db.attr/preds` to set up the attribute predicate:

```
{:db/ident :user/name,
 :db/valueType :db.type/string,
 :db/cardinality :db.cardinality/one,
 :db.attr/preds 'datomic.samples.attr-preds/user-name?}
```

### Entity Specs

Attribute predicates strengthen individual attribute constraints. But what if the constraints belong to the entire entity? For instance, certain entities must contain specific attributes, or relationships exist between entities.

For entity-wide constraints, Datomic provides **Entity Specs** syntax.

First, note that Entity Specs themselves are entities. Throughout this series, we’ve encountered many cases where Datomic uses entities to express semantics, including:

- Business domain data (e.g., people, products)  
- Transactions  
- Attributes  
- Enumerated types  
- Entity Specs  

### Required Fields

Here’s an example of using Entity Specs to enforce required fields in two steps:

* Define an Entity Spec:

Below, we define an Entity Spec called `:user/validate`, which requires `:user/name` and `:user/email`:

```
{:db/ident        :user/validate
 :db.entity/attrs [:user/name :user/email]}
```

* Use `:db/ensure` in transaction data to trigger checks:

In the transaction data below, the `:db/ensure` virtual attribute ensures that Datomic checks whether the entity being written satisfies the corresponding **Entity Spec**:

```
{:user/name "John Doe"
 :db/ensure :user/validate}
```

### Relationships Between Data

Relationships between data require invoking functions within Entity Specs, i.e., **Entity Predicates**. Here’s an example consisting of three steps:

* Define a custom function `datomic.samples.entity-preds/scores-are-ordered?`:

```
(ns datomic.samples.entity-preds
  (:require [datomic.api :as d]))

(defn scores-are-ordered?
  [db eid]
  (let [m (d/pull db [:score/low :score/high] eid)]
    (<= (:score/low m) (:score/high m))))
```

* Assign the custom function to `:db.entity/preds` to set up the Entity Predicate:

```
{:db/ident        :score/guard
 :db.entity/attrs [:score/low :score/high] ;; required attributes
 :db.entity/preds 'datomic.samples.entity-preds/scores-are-ordered?} ;; entity predicate
```

* Use `:db/ensure` in transaction data to trigger checks:

The `:db/ensure` below triggers the `:score/guard` check when this transaction data is written:

```
{:score/low 100
 :score/high 20
 :db/ensure :score/guard}
```

## References

* [Datomic Schema Reference](https://docs.datomic.com/schema/schema-reference.html)
