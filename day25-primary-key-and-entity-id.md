# Primary Key and Entity ID

Have you ever been puzzled about how to design the primary key for an SQL table?

Whenever I face this problem, the first question to consider is:

1. Should I use a **natural key** as the primary key? A natural key is one or more attributes derived from the business rules. This approach is convenient in the beginning.
2. Should I use a **surrogate key** as the primary key? A surrogate key decouples the primary key from business meaning, offering greater flexibility for future modifications. Additionally, some tables, being a subset of a data entity, lack attributes that can serve as a natural key, requiring the use of a surrogate key.

Once you decide to use a surrogate key, the next question is determining its data type:

- Auto-incrementing integers
- Strings (UUID)

Then, you must consider whether to guard against **enumeration attacks**, whether performance is sufficient, and whether to choose integers to mitigate enumeration attacks or UUIDs for enhanced performance.

## Datomic Makes the Primary Key Decision for You

In the Datomic world, the primary key is simply the entity ID. It’s that straightforward.

Having a simple default choice is **a huge relief** for me. It’s akin to switching from C++ to Java and no longer worrying about memory `new/delete`, or transitioning from JavaScript to Clojure, where all collections are persistent, and there’s no need to think about when to deep clone. When developing application software, understanding business rules is already exhausting enough. If the tools you use bombard you with five or six design combinations for performance and security, it leads to severe decision fatigue, making you want to pick one at random.

### Best Practices

Since Datomic provides a unified entity ID type for every data entity, some best practices can be derived:

1. When querying, use entity IDs for joins whenever possible for optimal performance.
2. Entity IDs should not be exposed externally. For instance, an API path should not directly use entity IDs.
3. Keys for external use should have a **unique identity constraint**.

Note: See [Day 26](./day26-constraints.md) for details on constraints.

## Enum Types and `:db/ident`

In [Day 24](./day24-writing-to-datomic.md), we listed the data types that attributes in Datomic can use. Compared to traditional SQL databases, one notable omission is enum types.

However, Datomic has a unique best practice for handling enums, which consists of three key points:

1. Define enum types using `:db/ident`. An enum type itself is a data entity.
2. Set the attribute pointing to the enum type with the data type `:db.type/ref`.
3. The name of the enum type can automatically convert to its entity ID.

### Example of Using Enum Types

* Defining attributes and enum types:

```
[
;; Define enum type :country/CA
{:db/ident :country/CA}

;; Define enum type :country/JP
{:db/ident :country/JP}

;; Define attribute :artist/country
;; Allow this attribute to point to an entity representing an enum type
{:db/ident :artist/country
 :db/valueType :db.type/ref
 :db/cardinality :db.cardinality/one
 :db/doc "An artist's country of residence"}
]
```

* Writing actual data:

```
[{:artist/name "Leonard Cohen"
  :artist/country :country/CA}]
```

When writing real data, since `:artist/country` has the data type `:db.type/ref`, its value should correspond to an entity ID. However, the above code is correct because in Datomic, any enum type defined with `:db/ident` can automatically convert to its entity ID.

## References

1. [Data Modeling](https://docs.datomic.com/schema/schema-modeling.html#enums)
