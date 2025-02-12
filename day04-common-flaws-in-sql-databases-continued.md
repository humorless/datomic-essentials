# Common Flaws of SQL Databases - Continued

## Locking and Isolation Issues

In SQL databases, writes require locks to ensure consistency.

Refer to the illustration below. A and B are two threads. If they simultaneously update the same piece of data, errors may occur. Ideally, the result after both A and B succeed should be `3`. However, if the operations are not properly isolated, the result might be `1` or `2`.

```
[The initial value of row i, column j in table X is 0]
 A -------> [Increment value of row i, column j in table X by 1]
 B -------> [Increment value of row i, column j in table X by 2]

-------------------> Time Axis
```

The overhead of locking is significant. Studies show that under high-concurrency conditions, a considerable portion of database performance is consumed by locking operations.

Due to this overhead, SQL databases often default to using **optimistic locks**—locking only when necessary, leaving most write failures for users to handle. While this improves performance, it compromises isolation, leading to intricate workarounds in high-concurrency programs.

## Schemas Are Hard to Query

* Imagine a PostgreSQL database with several tables containing the column `id`. How would you query to find all such tables?

In PostgreSQL, a practical workaround is to query **system tables**.

```sql
SELECT table_schema, table_name
FROM information_schema.columns
WHERE column_name = 'id'
ORDER BY table_schema, table_name;
```

In SQL databases, schemas are instructions, not data. Yet in real-world scenarios, you often need to treat schema information as data for querying purposes.

For users unaware of the existence of vendor-specific system tables, querying schema information can feel like a major obstacle.

## Rigid and Fixed Schemas

Suppose we have a table A with three columns:

```
;; Table A
id, item_name, price
```

Occasionally, you might want to add a fourth column `has_discount`, where some rows include this column, and others do not.

There are usually two approaches:

1. Modify the schema of table `A` to add the fourth column and allow NULL values.
2. Create another table `B` to hold the fourth column and reference table `A`. When the fourth column is needed, use a **join** to retrieve it.

```
;; Table B
A_id, has_discount
```

Both approaches have their downsides:

* The first approach introduces `NULL`, which can be ambiguous. In database literature, `NULL` can mean "missing" or "invalid," making the database semantics less clear.
* The second approach is more general but requires additional mental effort and feels like overengineering for simple cases.


## Impedance Mismatch

The term **impedance mismatch** originates from electronics, describing the incompatibility between two systems interacting with differing impedance. It has been adapted to describe the conceptual gap between object-oriented programming (OOP) models and relational database models, causing development challenges.

One common manifestation of this mismatch is the N+1 problem.

What Is the N+1 Problem? When using frameworks like Ruby on Rails to generate SQL automatically, the resulting queries are not optimized for performance. For instance, suppose you need to fetch all students in a class from a school database and calculate the square root of each student’s math score multiplied by ten. The framework might generate SQL like this:

* Run one query to retrieve all students in the class, yielding N records.
* Run N separate queries to fetch each student’s math score.

Given the inherently slow nature of I/O operations, this N+1 approach is highly inefficient. The optimal solution is to generate a single SQL query with a JOIN, retrieving all students and their math scores in one go—aligning better with SQL’s relational model.

While the N+1 problem’s performance impact might be tolerable, its inconsistency risks are more severe. With N+1 queries instead of one, these queries run at different times. If the data changes mid-execution, inconsistencies arise, potentially leading to logical errors.