# Starting with Datalog - Part 5

Previously, we discussed how to pass a single parameter into a query. However, there are cases where the data passed into the query is not a single variable but an array or even an array of tuples.

For such scenarios, SQL can handle them using the `ANY` syntax. (Note 1) (Note 2)

* **Database Setup**

```
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    category TEXT
);

INSERT INTO products (name, category) VALUES
('Laptop', 'Electronics'),
('Smartphone', 'Electronics'),
('Desk Chair', 'Furniture'),
('Coffee Table', 'Furniture'),
('Headphones', 'Electronics');
```

* **Passing an Array** into SQL Query

```
SELECT id, name, category
FROM products
WHERE category = ANY(ARRAY['Electronics', 'Furniture']);
```

* **Passing an Array of Tuples** into SQL Query

```
SELECT id, name, category
FROM products
WHERE (category, name) = ANY(ARRAY[
    ROW('Electronics', 'Laptop'),
    ROW('Furniture', 'Desk Chair')
]);
```

The same semantics can be expressed much more concisely in Datalog.

## Binding Forms in Datalog

Datalog has four binding forms. (Note 3)

| Binding Form          | Data Type Passed |
|-----------------------|------------------|
| `?a`                 | Scalar           |
| `[?a ?b]`            | Tuple            |
| `[?a ...]`           | Collection       |
| `[[?a ?b]]`          | Array of Tuples (Relation) |

We have previously demonstrated passing a scalar using a binding form. Let’s now explore the other **binding forms**.

### Passing a Tuple

To query "movie titles directed by 'James Cameron' and starring 'Arnold Schwarzenegger'," we can write the following query and pass the tuple `["James Cameron" "Arnold Schwarzenegger"]` as the parameter.

```
[:find ?title
 :in $ [?director ?actor]
 :where
 [?d :person/name ?director]
 [?a :person/name ?actor]
 [?m :movie/director ?d]
 [?m :movie/cast ?a]
 [?m :movie/title ?title]]
```

In this example, the tuple is destructured in the `:in` clause, allowing `"James Cameron"` to map to `?director` and `"Arnold Schwarzenegger"` to map to `?actor`.

### Passing an Array

To query "movie titles directed by either 'James Cameron' **or** 'Ridley Scott'," we can write the following query and pass the array `["James Cameron" "Ridley Scott"]`.

```
[:find ?title
 :in $ [?director ...]
 :where
 [?p :person/name ?director]
 [?m :movie/director ?p]
 [?m :movie/title ?title]]
```

Key points to note here:

1. In the query's `:in` clause, the `...` symbol denotes **array destructuring**.
2. Passing an array effectively implements a logical **OR operation**.

### Passing an Array of Tuples

To query "movie titles and their box office revenues directed by 'James Cameron'," we can write the following query:

```
[:find ?title ?box-office
 :in $ ?director [[?title ?box-office]]
 :where
 [?p :person/name ?director]
 [?m :movie/director ?p]
 [?m :movie/title ?title]]
```

And pass the scalar `"James Cameron"` along with the following **array of tuples** into the query:

```
[
 ...
 ["Die Hard" 140700000]
 ["Alien" 104931801]
 ["Lethal Weapon" 120207127]
 ["Commando" 57491000]
 ...
]
```

Passing an array of tuples can be viewed as performing a join between the Datalog query and an **external data table**. A data table is essentially a collection of tuples with the same shape. Ignoring implementation details about order (arrays are ordered, sets are unordered), treating an array of tuples as a data table is quite reasonable.

### Exercises

- [ ] Practice [Parameterized Queries](https://www.learndatalogtoday.org/chapter/3).

---

### Notes:

1. The example for passing an array of tuples into an SQL query uses the `ROW` syntax, which is PostgreSQL-specific and not part of ANSI SQL. However, I chose this syntax because it is relatively easy to read. In practice, using ANSI SQL allows code to be more portable across different SQL versions.
2. In the **Datalog - Part 4** article, the example of parameterized queries was illustrated using prepared statements. The examples in this article can also be adapted to use prepared statements.
3. The terms scalar, tuple, collection, and relation are taken from the Datomic official documentation. Although "collection" is not typically referred as "array," and "relation" is not usually referred as "array of tuples," the terms are used here to help readers connect with the concepts introduced in the SQL examples.
