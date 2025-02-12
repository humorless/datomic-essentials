# Starting with Datalog -- Part 9 (Aggregates and the With Clause)

Given a `person` table with a `date` column representing birthdates, if we want to find the youngest individual, we can use the following SQL query:

```
SELECT MAX(date) AS max_date
FROM person;
```

In Datalog, this can be rewritten as:

```
[:find (max ?date)
 :where
 [_ :person/date ?date]]
```

In this query, `max` is an **aggregate function**.

## Built-in Aggregate Functions in Datalog

| Aggregate Function      | Number of Returned Values | Notes                          |
|--------------------------|---------------------------|---------------------------------|
| `avg`         | 1                         |                                 |
| `count`          | 1                         | Includes duplicates             |
| `count-distinct` | 1               | Counts only unique values       |
| `distinct`    | n                         | Returns a collection of unique values |
| `max`         | 1                         | Compares all types, not limited to numbers |
| `max n`      | n                         | Returns up to `n` largest values |
| `median`        | 1                         |                                 |
| `min`          | 1                         | Compares all types, not limited to numbers |
| `min n`      | n                         | Returns up to `n` smallest values |
| `rand n`      | n                         | Randomly selects up to `n` values, allows duplicates |
| `sample n`    | n                         | Randomly selects up to `n` values, no duplicates |
| `stddev` | 1                    |                                 |
| `sum`              | 1                         |                                 |
| `variance`    | 1                         |                                 |

Built-in aggregate functions fall into two categories:

1. Those returning a single value, e.g., `min`, `max`, `sum`, `avg`.
2. Those returning a collection of values, e.g., `(min n ?d)`, `(max n ?d)`, `(sample n ?e)`. Here, `n` is any integer specified by the user, defining the size of the collection.

## The With Clause

Datalog's aggregate functions and their SQL counterparts may seem similar but exhibit subtle differences. Often, aggregate functions require a `:with` clause to clarify their semantics.

Let's explain with an example:

### SQL Database Setup

```
CREATE TABLE monsters (
    name VARCHAR(50) NOT NULL,
    count INT NOT NULL
);

INSERT INTO monsters (name, count)
VALUES 
    ('Cerberus', 3),
    ('Medusa', 1),
    ('Cyclops', 1),
    ('Chimera', 1);
```

### Querying the Total Number of Monster Heads in SQL

```
SELECT SUM(count) AS total_count
FROM monsters;

;; => 
 total_count 
─────────────
           6
(1 row)
```

### Datalog Setup for External Table

```
(def monsters [["Cerberus" 3]
               ["Medusa" 1]
               ["Cyclops" 1]
               ["Chimera" 1]])
```

### Querying the Total Number of Monster Heads in Datalog

```
(d/q '[:find (sum ?heads) .
       :in [[_ ?heads]]]
     monsters)
     
;; => 4
```

The result is `4`. Why?

#### Inspecting Intermediate Results Without Summation

```
(d/q '[:find ?heads
       :in [[_ ?heads]]]
     monsters)
     
;; => #{[1] [3]}
```

The issue is clear! The query semantics in Datalog mean "What are the unique numbers of monster heads?" rather than "What are the numbers of heads for all monsters?"

#### Solution: Add the With Clause

```
(d/q '[:find (sum ?heads) .
       :with ?monster
       :in [[?monster ?heads]]]
     monsters)
     
;; => 6
```

### Understanding the With Clause

The official documentation describes the `:with` clause as follows:

> The `:with` clause considers additional variables in forming the base set of query results, which are subsequently removed, leaving a useful bag of results constrained by the `:with` variables.

This explanation might feel abstract. I find it easier to understand the `:with` clause through comparison with SQL:

- In SQL, the `DISTINCT` keyword expresses "remove duplicates," but SQL defaults to "include duplicates."

```
SELECT count FROM monsters;

;; => 
 count 
───────
     3
     1
     1
     1
(4 rows)

SELECT DISTINCT count FROM monsters;
 
;; => 
 count 
───────
     3
     1
(2 rows)
```

- In contrast, Datalog's default semantics are "remove duplicates." The `:with` clause is used to override this default and allow duplicates.

```
(d/q '[:find ?heads
       :in [[_ ?heads]]]
     monsters)
;; => #{[1] [3]}

(d/q '[:find ?heads
       :with ?monster
       :in [[?monster ?heads]]]
     monsters)
;; => [[1] [1] [3] [1]]
```

### Exercises

- [ ] [Aggregates](https://www.learndatalogtoday.org/chapter/7).

### References

1. [The With Clause](https://docs.datomic.com/query/query-data-reference.html#with)
2. [Day of Datomic Example for the With Clause](https://github.com/Datomic/day-of-datomic/blob/daa457f766e16f55243a95513e759573b8827329/tutorial/with.clj)
