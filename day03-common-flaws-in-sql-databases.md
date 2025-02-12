# Common Flaws in SQL Databases

When introducing Datomic to friends, many of them often ask right away:

> Is this database SQL or NoSQL?

Here’s how I usually respond:

1. It’s not SQL. Datomic’s query language is not SQL but Datalog.
2. Unlike most NoSQL databases, which are typically designed for use cases where SQL databases struggle—such as horizontal scaling, distributed applications, or extreme performance—Datomic was explicitly designed to compete with SQL databases. It is meant for the same use case: serving as an operational database for applications.

Since Datomic is designed to replace SQL databases, you might wonder if SQL databases share any common design flaws. The answer is yes—there are quite a few. However, many of these flaws have long been mitigated by various workarounds, to the point where people have grown used to them and treat them as normal.

Here are some notable flaws worth discussing:

1. Inability to rewind database state.
2. String-based query languages.
3. Issues with locks and isolation.
4. Difficulty querying schema.
5. Rigid and static schemas.
6. Impedance mismatch.

## Inability to Rewind Database State

Let’s say you run a fruit store. On August 1st, the inventory in your database looks like this:

| id  | Product | Stock |
|-----|---------|-------|
| 1   | Apple   | 100   |
| 2   | Orange  | 200   |

By the end of August 1st, you’ve sold 60 apples. On August 2nd, your inventory is updated as follows:

| id  | Product | Stock |
|-----|---------|-------|
| 1   | Apple   | 40    |
| 2   | Orange  | 200   |

Now, at the end of the month, on August 31st, you want to know what your inventory looked like at the beginning of the month. Unfortunately, you can’t.

Without explicitly storing previous states, SQL databases cannot rewind to past states like version control software can.

## String-Based Query Languages

Readers familiar with SQL might wonder, “What’s the problem with this?” A query like the one below has been in use for years and feels pretty intuitive:

```sql
SELECT * 
FROM table_name
WHERE column_A = 5; 
```

However, if we could redesign this to use a data structure-based query language, it would be far better. For example:

```
{:select :*
 :from "table_name"
 :where [:= "column_A" 5]}
```

### Data Structures

Before diving into why string-based query languages are suboptimal, let’s define the term: **data structures**.

In traditional computer science courses, data structures refer to things like hash maps, heaps, trees, stacks, and queues, focusing on their performance characteristics. However, in programming languages, data structures refer to collection types such as: **sets**, **vectors**, **dictionaries**. Here, the focus is on the representation.

Take Clojure as an example

- Sets: `#{:a :b :c}`
- Vectors: `[:a :b :c}`
- Dictionaries: `{:a "hello" :b 15}`

### Advantages of Data Structure-Based Query Languages

Comparing the two query representations, let’s ask:

1. Which one is easier to read?
1. Which one is easier to manipulate programmatically?

When you write a SQL query, you implicitly rely on a syntax tree that exists in the background. Switching to a data structure-based representation makes this syntax tree explicit, enhancing clarity.

Moreover, in a data structure-based query, data types like strings and integers must be explicitly defined. For instance, in the example above, `"column_A"` (a string) and `5` (an integer) would be type-checked by the compiler.

Modern programming languages provide robust libraries for manipulating data structures. For example, if we need to modify a query to change the condition to check if column `B` equals `16`, it would look like this:

```
{:select :*
 :from "table_name"
 :where [:= "column_B" 16]}
```

Using Clojure, we could achieve this transformation programmatically as follows:

```
(def old-query 
  {:select :*
   :from "table_name"
   :where [:= "column_A" 5]})

(def new-query 
  (assoc old-query :where [:= "column_B" 16]))

(prn new-query)
;; =>
;; {:select :*
;;  :from "table_name"
;;  :where [:= "column_B" 16]}
```

This simple snippet `(assoc old-query :where [:= "column_B" 16])` demonstrates how straightforward it is to generate the `new-query`.