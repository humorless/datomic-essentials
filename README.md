# datomic-essentials

A series of Datomic introduction articles, using SQL to explain certain abstract ideas

## Datomic: A Database with Built-in Event Sourcing

Once, I came across a story shared by a teacher who taught DDD (domain-driven design). He mentioned that after conducting a course on event sourcing, some participants told him, “Teacher, this event sourcing thing... it seems hard to apply in practice.”

There are other similar stories. During my time at a particular organization, I asked some colleagues if they had ever implemented event sourcing. One colleague replied, “Yes, but that project ended in disaster. Someone even got so frustrated that he quitted.”

Strangely enough, when I use event sourcing, it works perfectly fine—no difficulties at all.

You might think I’m about to tell you that I’m some kind of 10x engineer. That’s not the case. What I’m really here to share is my secret: I use a database with built-in event sourcing—Datomic.


## Acknowledgments

The completion of this article series is largely inspired by my work at [Gaiwan](https://gaiwan.co/) and [LambdaIsland](https://lambdaisland.com/). Additionally, while attending [Heart of Clojure](https://2024.heartofclojure.eu/), I learned that non-Chinese readers were also interested, so I translated the original [Traditional Chinese version](https://ithelp.ithome.com.tw/users/20161869/ironman/7432) into English.  

If you have any thoughts after reading this series, feel free to reach out to [me](https://replware.dev/).

## Table of Contents

* [Preface: My Clients Call Me a Heretic](./day01-my-clients-call-me-a-heretic.md)
  * Event Sourcing
  * OLTP
* [Event Sourcing](./day02-event-sourcing.md)
  * Advantages
  * Implementation Challenges
* [Common Flaws in SQL Databases](./day03-common-flaws-in-sql-databases.md)
  * Inability to Rewind Database State
  * String-Based Query Languages
* [Common Flaws of SQL Databases - Continued](./day04-common-flaws-in-sql-databases-continued.md)
  * Locking and Isolation Issues
  * Schemas Are Hard to Query
  * Rigid and Fixed Schemas
  * Impedance Mismatch
* [The Database Category of Datomic](./day05-the-database-category-of-datomic.md)
  * The Clojure Language
* [Starting with Datalog — Part 1](./day06-datalog-part1.md)
  * Learn Datalog Today
* [Starting with Datalog - Part 2](./day07-datalog-part-2.md)
  * Datomic's Information Model
  * Basic Datalog Queries
* [Starting with Datalog - Part 3](./day08-datalog-part-3.md)
  * Multiple Data Patterns Sharing a Variable
  * Multiple Data Patterns with Multiple Variables
* [Starting with Datalog - Part 4](./day09-datalog-part-4.md)
  * Parameterized Queries in Datalog
* [Starting with Datalog - Part 5](./day10-datalog-part-5.md)
  * Binding Forms in Datalog
* [Starting with Datalog - Part 6 (More Queries)](./day11-datalog-part-6-more-queries.md)
  * Querying Attributes
  * Querying Transactions
* [Starting with Datalog -- Part 7 (More Joins)](./day12-datalog-part-7-more-joins.md)
  * Using Datalog to Perform Various SQL Joins
* [Starting with Datalog -- Part 8 (Predicates and Transformation Functions)](./day13-datalog-part-8-predicates-and-transformation-functions.md)
  * Five Types of Clauses
  * Predicate Expressions
  * Function Expressions
  * Runtime Environment
* [Starting with Datalog -- Part 9 (Aggregates and the With Clause)](./day14-datalog-part-9-aggregates-and-the-with-clause.md)
  * Built-in Aggregate Functions in Datalog
  * The With Clause
* [Starting with Datalog -- Part 10 (Find Specifications)](./day15-datalog-part-10-find-specifications.md)
  * The Four Common Find Specifications in Datalog
* [Starting with Datalog - Part 11 (Rules)](./day16-datalog-part-11-rules.md)
  * Using Datalog Rules
  * Characteristics of Rules
* [Starting with Datalog -- part 12 (Cross-Database Join)](./day17-datalog-part-12-cross-database-join.md)
  * Cross-Database Queries in Datalog
* [Starting with Datalog -- part 13 (SQL Window Function)](./day18-datalog-part-13-sql-window-function.md)
  * Top N per Group
* [Starting with Datalog -- part 14 (Subqueries)](./day19-datalog-part-14-subqueries.md)
* [Pull API](./day20-pull-api.md)
  * Recursive Queries
  * All Attributes
  * Reverse Queries
* [Impedance Mismatch and Datomic's Solution](./day21-impedance-mismatch-and-datomic-solution.md)
  * Common Solutions
  * Analyzing the Impedance
  * Datomic's Solution
* [as-of Query](./day22-as-of-query.md)
  * Code and Data Examples
  * Database Filters
* [Column Schema](./day23-column-schema.md)
  * No Need for NULL Values
  * Shared Column Schema
  * No Need for Bridge Tables
* [Writing to Datomic](./day24-writing-to-datomic.md)
  * Defining a Schema
  * Data Types for
  * Writing Actual Data
  * Format of Transaction Data ()
  * Unique Transaction Semantics in Datomic
* [Primary Key and Entity ID](./day25-primary-key-and-entity-id.md)
  * Datomic Makes the Primary Key Decision for You
  * Enum Types and
* [Constraints](./day26-constraints.md)
  * Common Applications
  * Advanced Applications
* [Indexes and Performance](./day27-indexes-and-performance.md)
  * Internal Structure of Indexes
* [Performance Optimization -- part 1](./day28-performance-optimization-part-1.md)
  * Guidelines
  * Measurement Tools
* [Performance Optimization -- Part 2](./day29-performance-optimization-part-2.md)
  * Direct Access to Datomic's Indexes
  * For OLAP Queries, Use Change Data Capture (CDC)
* [The Business Value of Datomic: A New Database Paradigm](./day30-the-business-value-of-datomic.md)
  * Event Sourcing
  * Highly Expressive Query Language
  * Unique Distributed Architecture
  * Conclusion

## License

Copyright @ Laurence Chen

Licensed under the term of [the Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/
).
