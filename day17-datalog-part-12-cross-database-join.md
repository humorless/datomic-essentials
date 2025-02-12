# Starting with Datalog -- part 12 (Cross-Database Join)

In the microservices design pattern, the "Database per Service" approach is a commonly used pattern. (Note 1) When adopting this pattern with SQL databases, there are typically three approaches:

1. Each service corresponds to several dedicated tables.
2. Each service corresponds to a dedicated schema.
3. Each service corresponds to a dedicated database.

Options 1 and 2 incur much lower additional development costs, while option 3 provides the most complete resource isolation. However, no matter how perfectly service boundaries are defined, there will often be a need to combine data from multiple services for queries. When this need arises, options 1 and 2 still allow direct SQL joins across data associated with different services. However, if option 3 is chosen, the development cost increases significantly.

## Cross-Database Queries in Datalog

Datomic does not provide a concept similar to **schema** in SQL. In other words, the only options are 1 and 3. On the other hand, Datalog provides syntax to enable cross-database queries. This means that when implementing the microservices design pattern with a Datomic database, you can confidently use option 3 as the default without worrying about the potential query difficulties that may arise in the future.

Let’s look at an example of a cross-database query in Datalog:

```
;; All revenue but orders in database b
[:find ?o ?r
 :in $a $b
 :where
 [$a ?o :order/revenue ?r]
 ($b not [?o :order/revenue])]
```

In a certain company, database `a` records the revenue of all orders, while orders with issues like refunds or bad debts are recorded in database `b`. The query above finds all orders and their corresponding revenues in database `a`, excluding orders that also appear in database `b`.

The syntax changes appear only in the `:in` clause and the `:where` clause, where **source variables** are added. In the `:in` clause, `$a` and `$b` are source variables representing data sources, usually databases but potentially arrays. In the first `:where` clause, `$a` indicates that the matching applies only to database `$a`, while in the second clause, `$b` specifies that the negation applies only to database `$b`.

### Variables in the `:in` Clause

So far, we’ve encountered three types of variables in the `:in` clause:

1. General binding variables, which must start with `?`.
2. Source variables, which must start with `$`.
3. Rule variables, represented by `%`.

### Omitting Source Variables

The main purpose of source variables is to **limit the scope of clauses**. When only one source variable is involved (i.e., not a cross-database query), there’s no need to consider the scope since it defaults to the primary database. Consequently, Datalog allows source variables to be omitted in such cases.

### Syntax for Source Variables

Examining the example above, we notice consistent usage of source variables in the `:where` clause:

1. They can be used in `not` clauses, placed at the far left.
2. They can be used in data pattern clauses, also placed at the far left.

But can source variables be used in other types of clauses? For example, can we apply source variables to rules? Yes, we can, and they are similarly placed at the far left.

In the example below, the rule expression clause `($ actor-movie ?name ?movie-title)` uses a source variable:

```
(def rules
  '[[(actor-movie ?name ?title)
    [?p :person/name ?name]
    [?m :movie/cast ?p]
    [?m :movie/title ?title]]]])

(d/q '[:find ?name
       :in $ % ?movie-title
       :where ($ actor-movie ?name ?movie-title)]
     db rules "The Terminator")
```

To summarize, source variables can be used as follows:

1. They can be applied to `not`, `not-join`, `or`, and `or-join` clauses.
2. They can be applied to data pattern clauses.
3. They can be applied to rule expression clauses.
4. When there is only one data source, source variables can be omitted. When added to a clause, they are always placed at the far left.

---

Note:

1. [Database per Service](https://microservices.io/patterns/data/database-per-service.html)
