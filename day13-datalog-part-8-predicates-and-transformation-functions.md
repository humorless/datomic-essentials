# Starting with Datalog -- Part 8 (Predicates and Transformation Functions)

First, let's define two terms: predicates and transformation functions.

- **Predicate**: A special type of function that takes a condition as input and outputs either `true` or `false`. For example, in the code below, `even?` is a predicate.

```
(even? 0) 
;; => true
(even? 1)
;; => false
(filter even? (range 10))
;;=> (0 2 4 6 8)
```

- **Transformation Function**: A pure function that takes data as input and transforms it in some way. In the example below, `add-2` is a transformation function. Predicates can be considered a special case of transformation functions.

```
(defn add-2  [x]
  (+ 2 x))

(add-2 x)
;; => 4

(map add-2 [0 1 2 3 4 5])
;; => (2 3 4 5 6 7)
```

## Five Types of `where` Clauses

According to the Datalog documentation, there are five types of clauses that can follow a `:where` keyword: (Note 1)

1. `not` clause
2. `not-join` clause
3. `or` clause
4. `or-join` clause
5. Expression clause

The `not`, `not-join`, `or`, and `or-join` clauses were covered in [Day 12](./day12-datalog-part-7-more-joins.md). Expression clauses, however, come in four forms:

1. Data patterns
   - The most commonly used type of clause.
2. Predicate expressions
3. Function expressions
   - Refers to transformation functions.
4. Rule expressions

## Predicate Expressions

Consider the following SQL query that retrieves movies released before 1984:

```
SELECT title
FROM movie
WHERE year < 1984;
```

This can be rewritten in Datalog as:

```
[:find ?title
 :where
 [?m :movie/title ?title]
 [?m :movie/year ?year]
 [(< ?year 1984)]]
```

In this Datalog query, `[(< ?year 1984)]]` is a predicate expression. It evaluates to either `true` or `false`. If `true`, the associated variables are included in the query results; otherwise, they are excluded.

Two important points to note:

1. Common predicates like `<`, `>`, `<=`, `>=`, `=`, and `not=` are built into Datalog and available out of the box.
2. When using Datomic with languages like Clojure, Java, or Kotlin, you can pass **user-defined predicates** into a Datalog query.

## Function Expressions

The Datomic database supports storing time using the `:db.type/instant` data type (Note 2). This type is equivalent to `java.util.Date`. To perform arithmetic operations on such timestamps, you need to convert them to `java.lang.Long`.

The following Clojure function calculates the age (`age`) of a person based on their birthdate (`birthday`) and the current date (`today`):

```
(defn age [birthday today]
  (quot (- (.getTime today)
           (.getTime birthday))
        (* 1000 60 60 24 365)))
```

Explanation of the implementation:

1. Accepts two `java.util.Date` objects (`birthday` and `today`) and converts them to `Long` timestamps using `.getTime`.
2. Subtracts the timestamps to get the time difference.
3. Divides the difference (in milliseconds) by the number of milliseconds in a year to calculate the age. (`quot` performs division.)

Using the `age` function, you can write a Datalog query to calculate a person's age. The following query takes a person's name and the current date as input, retrieves their birthdate, calculates their age using the `tutorial.fns/age` function, and returns the result:

```
[:find ?age
 :in $ ?name ?today
 :where
 [?p :person/name ?name]
 [?p :person/born ?born]
 [(tutorial.fns/age ?born ?today) ?age]]
```

In this query, `[(tutorial.fns/age ?born ?today) ?age]]` is a function expression. Its format is `[(<fn> <arg1> <arg2> ...) <result-binding>]`. The result of evaluating the function `(<fn> <arg1> <arg2> ...)` is bound to `<result-binding>`.

Three key points to note:

1. All pure functions in the `clojure.core` namespace can be used as functions in Datalog, except for `eval`.
2. The variable bound to the function result can take one of four forms: scalar, tuple, collection, or relation. These forms are identical to the [variable binding forms](./day10-datalog-part-5.md) in Datalog queries.
3. When using Datomic with languages like Clojure, Java, or Kotlin, **user-defined functions** can be passed into Datalog queries as transformation functions. (In this example, the user-defined function is `tutorial.fns/age`, and the full namespace must be specified.)

## Runtime Environment

SQL users might wonder if Datalog’s flexibility with **user-defined predicates** and **user-defined functions** is realistic. In SQL, you must explicitly install user-defined functions or stored procedures into the database before using them. This is because the SQL database runtime is separate from the backend program.

In the examples above, there’s no “installation” step for user-defined functions, as if the backend program and Datomic database share the same runtime. 

This phenomenon is due to Datomic’s unique architecture. In Datomic, reads and writes are completely decoupled. Query execution occurs in the backend program’s runtime, which explains why custom predicates and functions don’t require installation.

### Exercises

- [ ] [Predicates](https://www.learndatalogtoday.org/chapter/5)
- [ ] [Transformation Functions](https://www.learndatalogtoday.org/chapter/6)

Notes:

1. [Five Types of `where` Clauses](https://docs.datomic.com/query/query-data-reference.html#where-clauses)
2. [Datomic Supported Data Types](https://docs.datomic.com/schema/schema-reference.html#db-valuetype)
