# Preface: My Clients Call Me a Heretic

> Most thought leaders have, at one time or another, been considered heretics.
>
> -- Alan Weiss (*Million Dollar Consulting*)

I am an IT consultant. While helping clients complete their projects, I often receive feedback from them. This year, one client made the following comment:

> Teacher, you actually managed to win our bank's business. That is practically impossible. Our bank is steeped in conservative mainstream thinking. Compared to us, you're like some kind of cult leader...

Hearing such praise, I almost blurted out the much-maligned phrase, "The peach tree does not speak, yet its path is naturally formed" — a self-aggrandizing nonsense my former boss once angrily criticized. All I did was suggest replacing shell scripts and stored procedures with the modern data stack.

After receiving such feedback for the third time, I decided to share my thoughts on the pinnacle of "heresy" — the Datomic database.

## Event Sourcing

In 2024, Datomic ranked 143 overall on the [db-engines](https://db-engines.com/en/system/Datomic) website and 67th among relational databases (Note 1). However, db-engines categorizes databases into various niche types, such as Event Stores, Graph DBMS, and Time Series DBMS. For reasons unknown, Datomic is not classified as an Event Store on the site. If it were, I believe Datomic could easily rank #1 in the Event Stores category.

Since I assert that Datomic qualifies as an Event Store, it naturally supports features like event sourcing.

## OLTP

Readers might assume my recommendation is: "Consider Datomic if you’re building a large-scale distributed system or implementing event sourcing." That’s not the case. My suggestion is: *“Why not consider using Datomic to replace a traditional relational database in your next project, even if it’s just a standard OLTP application?”* (Note 2)

Switching databases is challenging. Many companies only attempt to implement features like event sourcing or scale-out capabilities after their systems are mostly established. If you anticipate needing to build distributed systems in the future, why not lay a solid foundation from the start?

Of course, selecting a database involves several important considerations:

- How is its query language? Is it easy to use?
- Does it have industrial-grade robustness?
- Is it beginner-friendly?
- How is its machine performance?
- Are major companies using it?
- …

We’ll explore these aspects in future articles.

### Notes:

1. Whether Datomic counts as a relational database is debatable. Since Datomic uses Datalog as its query language, which differs from SQL, there are some distinctions. However, its expressive power is at least equivalent to SQL.
2. If your project is OLAP-focused, feel free to check out my article from last year: [Modern Data Engineering and Data Analytics](https://ithelp.ithome.com.tw/users/20161869/ironman/6057).
