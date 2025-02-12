# Starting with Datalog -- Part 10 (Find Specifications)

In previous examples introducing Datalog, you might have encountered some special syntax that could be quite confusing. This section provides a detailed explanation.

- `[variable/expression ...]` in the `:find` clause, which appeared in [Day12](./day12.md)

```
[:find [(pull ?e [:person/name 
                  :movie/_cast]) ...]
 :where 
 [?e :person/name]]
```

In this query, the `[(pull variable pattern) ...]` part includes the `[variable/expression ...]` construct, which is a type of `:find` clause syntax, also called a **find specification**.

- `variable/expression .` in the `:find` clause, which appeared in [Day14](./day14.md)

```
(d/q '[:find (sum ?heads) .
       :in [[_ ?heads]]]
     monsters)
```

Similarly, the `variable/expression .` construct in this query is also a type of find specification.

## The Four Common Find Specifications in Datalog

In SQL databases, data is always exchanged in **one single format: tables**. Since SQL query outputs are standardized as tables, assembling data using subqueries becomes straightforward.

For example, if we want to generate a series of numbers from 1 to 10 using SQL, the output is not an array but a table with a single column.

```
SELECT generate_series(1, 10) AS num_series;

;; => 
 num_series
------------
          1
          2
          3
          4
          5
          6
          7
          8
          9
         10
```

In Datalog, both [input](./day10.md) and output data can take one of four forms:  
- **Scalar**  
- **Tuple**  
- **Collection**  
- **Relation**  

Below are examples of each find specification: (Note 1)

### Scalar Find Specification

```
;; single scalar find spec
(d/q '[:find ?v .
       :where [0 :db/ident ?v]]
     db)

;; Output: a single value
;; => :db.part/db
```

### Tuple Find Specification

```
;; single tuple find spec
(d/q '[:find [?e ?ident]
       :where [?e :db/ident ?ident]]
     db)
     
;; Output: a single tuple
;; => [106 :session/published?]
```

### Collection Find Specification

```
;; collection find spec
(d/q '[:find [?v ...]
       :where [_ :db/ident ?v]]
     db)

;; Output: a collection
;; => 
[:db.sys/reId
 :db/code
 :tito.registration/reference
 :db.entity/preds
 :db/fn
 ...]
```

### Relation Find Specification

```
;; relation find spec
(d/q '[:find ?e ?v
       :where [?e :db/ident ?v]]
     db)
     
;; Output: a relation (array of tuples)
;; => 
#{[106 :session/published?] 
  [17592186045418 :location.type/depot]
  [121 :tito.ticket/email]
  [12 :db.install/valueType]
  [45 :db/noHistory]
 ...}
```

---

**Note:**  
1. [Find Spec Examples from Day of Datomic](https://github.com/Datomic/day-of-datomic/blob/daa457f766e16f55243a95513e759573b8827329/tutorial/find_specification_examples.clj)
