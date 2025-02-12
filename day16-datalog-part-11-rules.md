# Starting with Datalog - Part 11 (Rules)

Here we’ll discuss a unique abstraction mechanism in Datalog: **rules**.

What are **rules**? Consider the following Datalog query that retrieves the names of actors in the movie *The Terminator*:

```
[:find ?name
 :where
 [?p :person/name ?name]
 [?m :movie/cast ?p]
 [?m :movie/title "The Terminator"]]
```

A small part of this query, which maps movie titles to actor names, is often repeated across different queries. Is there an abstraction mechanism that can help avoid rewriting the following code segment repeatedly?

```
[?p :person/name ?name]
[?m :movie/cast ?p]
[?m :movie/title ?title]
```

Yes, there is! This abstraction mechanism is called **rules**. Let’s see an example:

```
;; Our defined rule
[(actor-movie ?name ?title)
 [?p :person/name ?name]
 [?m :movie/cast ?p]
 [?m :movie/title ?title]]
 
;; Rewritten Datalog query
 [:find ?name
  :in $ %
  :where (actor-movie ?name "The Terminator")]
```

Since it’s an abstraction mechanism, rules are quite similar to functions in programming languages. A function typically consists of three parts: **name**, **parameters**, and **implementation**.

```
(defn add-2   ;; "add-2" is the function name
  [x]          ;; [x] are the parameters
  (+ x 2))     ;; (+ x 2) is the implementation
```

Datalog rules have a similar structure: `[rule-name rule-variables clauses]`. In the example, `actor-movie` is the **rule name**, `?name` and `?title` are the **rule variables**, and the rest are **clauses**. Together, the rule name and variables form the **rule head**.

## Using Datalog Rules

Refer to the code below:

1. Define an external variable `rules` as a collection of rules.
2. In the Datalog query, refer to the external `rules` variable and use `%` in the `:in` clause to include the rules.
3. In the `:where` clause, invoke the rule by its name, such as `(actor-movie ?name ?movie-title)`.

```
(def rules
  '[[(actor-movie ?name ?title)
    [?p :person/name ?name]
    [?m :movie/cast ?p]
    [?m :movie/title ?title]]]])

(d/q '[:find ?name
       :in $ % ?movie-title
       :where (actor-movie ?name ?movie-title)]
     db rules "The Terminator")
```

## Characteristics of Rules

Readers might wonder, "Does SQL have a similar feature for rules?"

The answer is no. While tools like [dbt/Jinja](https://docs.getdbt.com/docs/build/jinja-macros) allow packaging and reusing sections of `JOIN` statements across SQL queries, the features of Datalog rules extend beyond SQL capabilities. Let’s explore these characteristics:

### Logic Programming

```
[(actor-movie ?name ?title)
 [?p :person/name ?name]
 [?m :movie/cast ?p]
 [?m :movie/title ?title]]
```

Since Datalog is a logic programming language, it supports bi-directional computation. Any **rule variable** can serve as either an **input** or an **output**.

For example, using the `actor-movie` rule:

1. Bind `?title` to find actor names based on a movie title.
2. Bind `?name` to find movie titles based on an actor name.
3. Bind both variables to check for a specific pairing in the database.
4. Bind neither variable to retrieve all possible combinations.

### Reusability of Rule Heads

Rules can also facilitate **`OR` logic** by reusing the same rule head.

```
[[(associated-with ?person ?movie)
  [?movie :movie/cast ?person]]
 [(associated-with ?person ?movie)
  [?movie :movie/director ?person]]]
```

Using the `associated-with` rule, the query below identifies both actors and directors for *Predator*:

```
[:find ?name
 :in $ %
 :where
 [?m :movie/title "Predator"]
 (associated-with ?p ?m)
 [?p :person/name ?name]]
```

### Recursive Calls

Rules can invoke themselves recursively. In [Day 13](./day13-datalog-part-8-predicates-and-transformation-functions.md), we mentioned five types of clauses in Datalog:

1. `not` clause
2. `not-join` clause
3. `or` clause
4. `or-join` clause
5. Expression clauses: data patterns, assertions, functions, or rules.

If a rule’s clauses include a self-reference, the rule becomes recursive. This enables Datalog to handle tasks like tree or graph traversal seamlessly. (Note 1)

### Exercises

- [ ] [Rules](https://www.learndatalogtoday.org/chapter/8)

### Notes:

1. Modern SQL supports Recursive CTEs for tasks like tree or graph traversal, but Datalog’s syntax remains more intuitive.

### References

1. [Datomic Rules Documentation](https://docs.datomic.com/query/query-data-reference.html#rules)
