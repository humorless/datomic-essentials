# Performance Optimization -- part 1

Datomic, like many databases, tends to slow down as data volume increases, even for typical queries. When optimizing Datomic’s performance, two approaches can be applied:

1. Guidelines
2. Measurement Tools

## Guidelines

The guidelines are derived from Datomic's design principles and practical development experience.

### Place Clauses with Fewer Unbound Variables First (Join Along)

The more unbound variables a clause has, the more data atoms it is likely to match. Therefore, placing clauses with fewer unbound variables first can minimize the total number of data atoms a query needs to match.

### Place the Most Selective Clause First

Datomic's `:where` clauses are executed sequentially. Placing the most selective clause first can minimize the total number of data atoms matched.

For example, if `[?e :only/matches ?few]` matches fewer data atoms than `[?e :will/match ?many]`, this ordering is likely to yield better performance.

```
:where [?e :only/matches ?few]
       [?e :will/match ?many]
```

### Moderately Avoid N+1 Query Patterns

Because Datomic internally implements event sourcing and provides value-oriented programming semantics, it ensures data consistency. For Datomic, the entire database corresponds to different values at any point in time and can be accessed like a snapshot, much like Git.

This convenience often leads engineers to write N+1 queries without worrying about data inconsistencies. While there are no consistency issues, performance problems can arise.

Hence, for queries that are known to match a large number of data atoms, special attention should be paid to whether they use an N+1 pattern. If so, modifying the query to reduce the number of executions can significantly decrease I/O time.

## Measurement Tools

Datomic provides two measurement tools: one for logical optimization at the query level, **Query Stats**, and another closely related to deployment and architectural optimization, **I/O Stats**.

### Query Stats

Example:

```
(d/query {:query '[:find (count ?a)
                   :in $ ?aname
                   :where [?a :artist/name ?aname]]
          :args [db "The Beatles"]
          :query-stats true})
```

Query Stats provide the exact number of data atoms matched by each clause. What we intuitively guess as the "most selective clause" might not always be the case. Query Stats allow us to obtain precise data.

### I/O Stats

Example:

```
(d/query {:query '[:find (count ?a)
                   :in $ ?aname
                   :where [?a :artist/name ?aname]]
          :args [db "The Beatles"]
          :io-context :artist/by-name})
```

Datomic's architecture uses multiple layers of caching. Observing the I/O information from these caches can guide us in adjusting cache deployment.

The layers of Datomic's cache, from closest to furthest, are as follows:

| Cache Name                 | Cached Object Description                             |
|----------------------------|------------------------------------------------------|
| :ocache                    | Number of segments accessed through in-memory object caching (Java objects) |
| :valcache                  | Number of segments accessed through Valcache (SSD caching) |
| :memcached                 | Number of segments accessed through Memcached        |
| (storage of record) :ddb/:sql/:cass etc. | Number of segments accessed through Datomic storage (`:ddb/:sql/:cass` represents the current storage protocol) |
| :inflight-lookup-ms        | Total time spent on parallel requests waiting for the same segment |

## References

1. [Datomic Query Improver](https://github.com/ParkerICI/datomic-query-improver)
2. [Datomic Query Stats](https://docs.datomic.com/reference/query-stats.html)
3. [Datomic I/O Stats](https://docs.datomic.com/reference/io-stats.html)
