# Starting with Datalog - Part 3

Previously, while explaining the operation of basic queries, we mentioned:

> Using the condition [?e :person/name "Ridley Scott"] to match the entire database content. The `?e` in the `:where` clause is an unknown variable.

Here, we define some additional terms to make future discussions more precise.

- **Data Pattern**
  - Arrays like [?e :person/name "Ridley Scott"] appearing after `:where` are called **data patterns**.
  - Since data patterns are typically used to query Datomic databases, they consist of arrays containing [E A V Tx]. However, in many cases, the `Tx` value is irrelevant, so data patterns are often written as [E A V].
  - Elements within a data pattern can be one of three possibilities:
     - **Variable**: It can **bind** to new values.
     - **Constant**: A fixed value, typically a keyword, number, or string.
     - **Blank**: It can match any value but won't be used; it's only a placeholder. (Note 1)

## Multiple Data Patterns Sharing a Variable

Let's look at a more complex query:

```
[:find ?title
 :where
 [?e :movie/year 1987]
 [?e :movie/title ?title]]
```

This query is intended to find the titles of all movies from 1987. Here are two notable points:

1. This query contains two **data patterns**, and both use the same variable `?e`.
   - In Datalog, when multiple data patterns use the same variable, the variable across all patterns binds to the same value.
2. The order of data patterns doesn't matter. While the order may impact performance, it doesn't affect semantics. Thus, the query above can be rewritten as follows, yielding the same results:

```
[:find ?title
 :where
 [?e :movie/title ?title]
 [?e :movie/year 1987]]
```

The corresponding SQL representation of the query would be:

```
SELECT title
FROM movie
WHERE year = 1987;
```

## Multiple Data Patterns with Multiple Variables

If we want to query: "Who starred in the movie *Lethal Weapon*?", we can write the following query:

```
[:find ?name
 :where
 [?m :movie/title "Lethal Weapon"]
 [?m :movie/cast ?p]
 [?p :person/name ?name]]
```

The data patterns can be interpreted as follows:

- `[?m :movie/title "Lethal Weapon"]`  
  - Matches the movie title "Lethal Weapon", allowing us to determine the entity ID (`?m`) for the movie.
- `[?m :movie/cast ?p]`  
  - Given the value of `?m`, this determines the entity ID (`?p`) for a person.
- `[?p :person/name ?name]`  
  - Given the value of `?p`, this determines the name of the person (`?name`).

The corresponding SQL representation of this query would be:

```
SELECT person.name
FROM movie 
INNER JOIN person 
ON movie.cast = person.id
WHERE movie.title = "Lethal Weapon";
```

Notice how the `INNER JOIN` happens implicitly in the query.

### Practice

- [ ] Write about [Data Patterns](https://www.learndatalogtoday.org/chapter/2).

Note:

1. The term **placeholder** may feel abstract; the following example provides clarity.

The query below aims to find the entity ID (`?p`) of a person named "Clinton Eastwood" who has acted in a movie:

```
[:find ?p
 :where
 [_ :movie/cast ?p]
 [?p :person/name "Clinton Eastwood"]]
```

Since we need to ensure this person has acted in a movie, we include `[_ :movie/cast ?p]`. However, we don't care which movie it is; as long as one exists, we use an underscore to **occupy** the **position** of `E` in this data pattern.
