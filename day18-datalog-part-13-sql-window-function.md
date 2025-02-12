# Starting with Datalog -- part 13 (SQL Window Function)

In [day 11](./day11-datalog-part-6-more-queries.md), [day 16](./day16-datalog-part-11-rules.md), and [day 17](./day17-datalog-part-12-cross-database-join.md), we discussed several features unique to Datalog, such as:

1. Querying table schemas.
2. Querying transactions.
3. Datalog rules.
4. Cross-database queries.

But does SQL have features that Datalog struggles to handle? Yes, though not many. The most notable is the **window function**.

Why doesn’t Datalog have window functions? This goes back to the use case of Datomic, which is primarily designed for online transaction processing (OLTP). Window functions, on the other hand, shine in online analytical processing (OLAP) scenarios, as OLAP heavily relies on complex report generation.

That said, there are a few reporting scenarios encountered in both OLTP and OLAP. Even though Datalog doesn’t support window functions, it can still elegantly handle some **common reporting tasks**.

## Top N per Group

Given the `cities` table below, how can we use SQL to find the two most populous cities in each country?

| country        | city         | population |
|----------------|--------------|------------|
| United States  | New York     | 8175133    |
| United States  | Los Angeles  | 3792621    |
| United States  | Chicago      | 2695598    |
| France         | Paris        | 2181000    |
| France         | Marseille    | 808000     |
| France         | Lyon         | 422000     |
| United Kingdom | London       | 7825300    |
| United Kingdom | Birmingham   | 1016800    |
| United Kingdom | Leeds        | 770800     |

We can achieve this with the following SQL query:

```
SELECT
  *
FROM
  (
    SELECT
      country,
      city,
      population,
      row_number() OVER (
        PARTITION BY country
        ORDER BY
          population DESC
      ) AS country_rank
    FROM
      cities
  ) ranks
WHERE
  country_rank <= 2;
```

The key technique here is using the SQL window function `row_number()`.

### Datalog Version

```
(d/q '[:find ?country (max 2 ?population)
       :in $
       :where
       [?ct :country/name ?country]
       [?ct :city/name ?city]
       [?ct :city/population ?population]]
     (db/db))

(comment
 ;; => 
 [["France" [2181000 808000]]
  ["United Kingdom" [7825300 1016800]]
  ["United States" [8175133 3792621]]])
```

Surprisingly, this query is even simpler than the SQL version with window functions!

While Datalog doesn’t provide window functions for handling complex reports, its built-in aggregate functions include `(max n ?variable)`, an **advanced aggregate function**. This makes common queries like "Top N per group" much clearer and more concise in Datalog.

However, this solution isn’t complete since `(max n ?variable)` only handles a single variable. Thus, we don’t see the "city" information.

### User-Defined Aggregate Functions

What if we want to include the "city" information as well?

For such relatively complex requirements, we need to use a **user-defined aggregate function**.

In the example below:

- `max-by-population` is a user-defined aggregate function. In [Day 13](./day13-datalog-part-8-predicates-and-transformation-functions.md), we discussed runtime environment issues: user-defined aggregate functions don’t require special installation due to Datomic’s unique architecture.
- When calling it in a Datalog query, you need to provide the full namespace, so it becomes `repl-sessions.day18/max-by-population`.
- `tuple` is a built-in Datomic function used to bundle variables like `?city` and `?population` into a single array.
- [Code link](https://github.com/humorless/ithome2024/blob/main/repl-sessions/day18.clj)

```
(defn max-by-population
  [coll]
  (let [result (sort-by second > coll)]
    (take 2 result)))

(d/q '[:find ?country (repl-sessions.day18/max-by-population ?tup)
       :in $
       :where
       [?ct :country/name ?country]
       [?ct :city/name ?city]
       [?ct :city/population ?population]
       [(tuple ?city ?population) ?tup]]
     (db/db))

(comment
  ;; => 
 [["France" (["Paris" 2181000] ["Marseille" 808000])]
  ["United Kingdom" (["London" 7825300] ["Birmingham" 1016800])]
  ["United States" (["New York" 8175133] ["Los Angeles" 3792621])]])
```
