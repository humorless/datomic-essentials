# Writing to Datomic

Before diving in, let’s clarify some terminology:

1. In Datomic’s official documentation, writing to the database is referred to as a **transaction**.
2. In the same documentation, a schema field is referred to as an **attribute**.

To write data to a Datomic database, the first step is to define the schema. Unlike SQL databases, Datomic does **not** have a dedicated Data Definition Language (DDL), as Datomic’s schema is itself data, so it can be written using the same syntax.

## Defining a Schema

To define a schema, you need to use Datomic’s built-in attributes. Each attribute must include three required properties:

For example, let’s define a schema for `:movie/title`. This attribute can then be used to store movie titles. When defining it, you must specify three built-in attributes:

- `:db/ident`: Defines the name of the attribute.
- `:db/valueType`: Defines the data type of the attribute.
- `:db/cardinality`: Specifies whether the attribute corresponds to a single value or multiple values.

These three built-in attributes (`:db/ident`, `:db/valueType`, `:db/cardinality`) are required. Additionally, `:db/doc` is an optional built-in attribute. Note that all built-in attributes follow a consistent naming convention, starting with `:db/`.

```
{:db/ident :movie/title
 :db/valueType :db.type/string
 :db/cardinality :db.cardinality/one
 :db/doc "The title of the movie"}
```

## Data Types for `:db/valueType`

The following data types can be used with `:db/valueType`:

```
(':db.type/bigdec'  | ':db.type/bigint'  | ':db.type/boolean' |
 ':db.type/bytes'   | ':db.type/double'  | ':db.type/float'   |
 ':db.type/instant' | ':db.type/keyword' | ':db.type/long'    |
 ':db.type/ref'     | ':db.type/string'  | ':db.type/symbol'  |
 ':db.type/tuple'   | ':db.type/uuid'    | ':db.type/uri')
```

Among these, the following 8 types are most commonly used:

- `:db.type/instant`: Represents a date and time, corresponding to `java.util.Date`.
- `:db.type/uuid`: Represents a UUID, corresponding to `java.util.UUID`.
- `:db.type/ref`: Represents a reference to another entity.
- `:db.type/string`: Represents a string, corresponding to `java.lang.String`.
- `:db.type/long`: Represents a long integer.
- `:db.type/float`: Represents a floating-point number.
- `:db.type/double`: Represents a double-precision floating-point number.
- `:db.type/boolean`: Represents a boolean value.

**Note:** The `:db.type/bytes` type has been deprecated due to design concerns.

## Writing Actual Data

Let’s walk through an example of defining movie attributes, writing these attributes, and then writing movie data.

### Define Attributes

```
(def movie-schema [{:db/ident :movie/title
                    :db/valueType :db.type/string
                    :db/cardinality :db.cardinality/one
                    :db/doc "The title of the movie"}

                   {:db/ident :movie/genre
                    :db/valueType :db.type/string
                    :db/cardinality :db.cardinality/one
                    :db/doc "The genre of the movie"}

                   {:db/ident :movie/release-year
                    :db/valueType :db.type/long
                    :db/cardinality :db.cardinality/one
                    :db/doc "The year the movie was released in theaters"}])
```

This defines three **custom attributes**: `:movie/title`, `:movie/genre`, and `:movie/release-year`. Once these are defined, they can be used to describe the data being written to the database.

### Write Attributes

```
@(d/transact conn movie-schema)
```

Here, `conn` is the database connection, `d/transact` is the API for writing to the database, and `movie-schema` contains the defined attributes.

### Write Movie Data

```
(def first-movies [{:movie/title "The Goonies"
                    :movie/genre "action/adventure"
                    :movie/release-year 1985}
                   {:movie/title "Commando"
                    :movie/genre "action/adventure"
                    :movie/release-year 1985}
                   {:movie/title "Repo Man"
                    :movie/genre "punk dystopia"
                    :movie/release-year 1984}])

@(d/transact conn first-movies)
```

Here, `first-movies` is a variable containing the movie data. Each movie entry is described using the previously defined **custom attributes**.

**Note:** Datomic has three versions: Datomic Cloud, Datomic Pro, and Datomic Local. Additionally, its API comes in two forms: Peer API and Client API. Explaining the differences and selection criteria is beyond the scope of this article, which focuses on **Datomic Pro** and **Peer API**.

## Format of Transaction Data (`tx-data`)

The second argument of the `d/transact` API is a **vector of tx-data**. Think of transaction data as the smallest unit that corresponds to individual data atoms (datoms). Typically, multiple datoms are written at once, so the argument is a vector of tx-data.

Transaction data can be in two forms: map or list. These are equivalent, with the map form being a syntactic sugar for the list form. To fully understand transaction data, let’s start with the list form.

### Basic Examples of List Form Transaction Data

* Set an entity-id’s attribute to a value:

```
[:db/add entity-id attribute value]
```

* Retract an entity-id’s attribute value:

```
[:db/retract entity-id attribute value?]
```

### Equivalent Representations

* Map form of transaction data with explicit `:db/id`:

```
[{:db/id item-1-id
  :line-item/product chocolate
  :line-item/quantity 1}
 {:db/id item-2-id
  :line-item/product whisky
  :line-item/quantity 2}]
```

* List form of transaction data:

```
[[:db/add item-1-id :line-item/product chocolate]
 [:db/add item-1-id :line-item/quantity 1]
 [:db/add item-2-id :line-item/product whisky]
 [:db/add item-2-id :line-item/quantity 2]]
```

### Temporary IDs (Tempids)

A common question arises: *“When data hasn’t been written to the database yet, how do we specify the entity IDs like `item-1-id` or `item-2-id`?”*

The answer is simple: use a temporary string, such as `"temp-item-1"`. These temporary strings are referred to as **tempids** in Datomic’s documentation.

## Unique Transaction Semantics in Datomic

Let’s compare Datomic’s transaction semantics with those of SQL databases.

Most SQL databases treat a transaction as a series of updates, each modifying the database state. Depending on the level of isolation, a transaction may see interleaved updates from other transactions:

```
BEGIN TRANSACTION
UPDATE ... ;; db state a -> db state a'
UPDATE ... ;; db state b -> db state b'
UPDATE ... ;; db state c -> db state c'
COMMIT
```

In contrast, a Datomic transaction is defined as appending a set of datoms to the database.

## References

1. [Datomic Schema Reference](https://docs.datomic.com/schema/schema-reference.html)
2. [Datomic Transaction Tutorial](https://docs.datomic.com/peer-tutorial/transact-schema.html)
3. [Datomic Transaction Data Reference](https://docs.datomic.com/transactions/transaction-data-reference.html)
4. [Comparison with Updating Transactions](https://docs.datomic.com/tech-notes/comparison-with-updating-transactions.html)
