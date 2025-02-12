# Starting with Datalog -- part 14 (Subqueries)

In the previous chapter, we discussed SQL's window functions, which don’t have a direct equivalent in Datalog. However, we demonstrated how to achieve similar semantics using user-defined aggregate functions.

If we think about it, window functions are already considered advanced syntax. However, in SQL-92, there is a `HAVING` clause that allows filtering results after aggregation. How can we implement this in Datalog?

### SQL `HAVING` for Filtering Aggregated Results

Suppose we have a database containing posts and tags, and we want to find out: *“For each tag, how many posts it appears in, and filter out tags that appear fewer than three times, then output the results.”*

This can be achieved with the following SQL query:

```
SELECT tag, COUNT(post_id) AS tag_usage
FROM post_tag
GROUP BY tag
HAVING COUNT(post_id) >= 3;
```

### Datalog Version: Using Subqueries

```
(d/q '[:find ?tag ?n
       :where
       [(datomic.api/q
         (quote [:find ?e (count ?post)
                 :where [?post :post/tag ?e]]) $)
        [[?tag ?n]]]
       [(>= ?n 3)]] (db/db))
```

How should we understand this query?

- In [Day 13](./day13.md), we mentioned that Datalog also allows us to use **function expressions**. The first `:where` clause, `[(datomic.api/q ... $)  [[?tag ?n]]]`, is a function expression. Its form is `[(<fn> <arg1> <arg2> ...) <result-binding>]`. This binds the result of the function computation `(datomic.api/q ...)` to the variable `[[?tag ?n]]`.

- Since the function computation involves a subquery, let’s first break down the subquery. The query below finds all post-tag combinations, groups them by tag, and counts how many times each tag appears in posts. The result is in the form of `Tag: Occurrences`, which is then bound to the variable `[[?tag ?n]]` that the parent query can access.

```
(d/q '[:find ?e (count ?post)
       :where [?post :post/tag ?e]]
     (db/db))
```

- Finally, the parent query uses the assertion expression `[(>= ?n 3)]` to filter the results. This achieves the equivalent effect of the SQL `HAVING` clause.
- [Code link](https://github.com/humorless/ithome2024/blob/main/repl-sessions/day19.clj)
