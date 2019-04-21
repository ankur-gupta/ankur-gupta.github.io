---
layout: post
title: ARRAYs and STRUCTs in BigQuery
date: 2017-10-31
logo: database
---

### NESTED and REPEATED Fields

Google BigQuery (BQ) allows nested and repeated fields. The ability to store nested and repeated fields
is useful to store data that has more complicated data structure than a simple tabular structure.
One example is a JSON that contains arrays within a deeply nested field.
Tabularizing (or flattening) such a JSON is difficult and often requires making suitable
assumptions or setting convetions.
BQ allows us to store this JSON without the need to flatten or tabularize it.
In fact, you can create a new table in BQ from a JSON file and download the result of a query
as a JSON.

There is one drawback of storing data that has nested and repeated structure -- it's difficult to
query it. Storing nested and repeated field structure in BQ shifts the burden of
tabularizing (or flattening) the data from the person who is adding data to BQ to
the person who is going to query the data.


Let's take the following example dataset which has such a nested and repeated structure.
You can find the JSON version of this dataset [here](/assets/arrays-and-structs-in-google-bigquery/example-dataset-1.json). Feel free to load this JSON into BQ and play along.

**Example Dataset 1** ([JSON](/assets/arrays-and-structs-in-google-bigquery/example-dataset-1.json))
![Example Dataset 1](/assets/arrays-and-structs-in-google-bigquery/example-dataset-1.png)
FIXME: Dataset has a different meaning in BQ. Change this.

### Standard and Legacy SQL

As of writing this post, BQ has two flavors of SQL -- standard SQL and legacy SQL.
Fow querying a table that has nested and repeated fields, such as the dataset shown in Example 1,
standard SQL is not necessarily better than legacy SQL. To illustrate this, let's try to
write the simplest query in both standard and legacy dialects.

```sql
-- standardSQL
SELECT
  *
FROM
  `example-project-184704.example_dataset.example_dataset_1`
-- works fine
```

```sql
-- legacySQL (with Flatten Results checked)
SELECT
  *
FROM
  [example-project-184704:example_dataset.example_dataset_1]
-- Error: Cannot output multiple independently repeated fields at the same time. Found array_of_integers and array_of_floats
```

Standard SQL query works fine and returns the entire dataset as expected.
Legacy SQL, with `Flatten Results` option turned on[^1], fails with a reason that has something to
do with the repeated fields. In real world, the dataset is typically huge and the presence of
repeated fields makes it difficult to write simple queries in legacy SQL. This makes legacy SQL
annoying to use.

However, there may be one benefit to using legacy SQL over standard SQL in the case of nested
fields.

```sql
-- standardSQL
SELECT
  array_of_nested_structs.float_within_struct
FROM
  `example-project-184704.example_dataset.example_dataset_1`
-- Error: Cannot access field float_within_struct on a value with type ARRAY<STRUCT<struct_within_struct STRUCT<character_within_struct STRING>, float_within_struct FLOAT64>> at [3:27]
```

```sql
-- legacySQL
SELECT
  array_of_nested_structs.float_within_struct
FROM
  [example-project-184704:example_dataset.example_dataset_1]
-- works fine (but returns a flattened result)
```

Legacy SQL allows us to ["dot" into a nested repeated field](https://cloud.google.com/bigquery/docs/reference/standard-sql/migrating-from-legacy-sql) without having to explicitly unnest anything.
The same query in standard SQL fails with a complicated error message but it essentially means
that standard SQL does not unnest the field implicitly.

So, whether you're using legacy or standard SQL, nested and repeated fields are difficult to
query. In the rest of this post, we will **focus only on standard SQL** because (1) standard SQL
is more powerful overall and (2) legacy SQL will probably be phased out.

### THE PROBLEM
As we see in the example query above, naively querying a nested field using standard SQL
throws an error that looks something like this:

```
Error: Cannot access field float_within_struct on a value with type ARRAY<STRUCT<struct_within_struct STRUCT<character_within_struct STRING>, float_within_struct FLOAT64>> at [3:27]
```

There are [various](https://stackoverflow.com/questions/39109817/cannot-access-field-in-big-query-with-type-arraystructhitnumber-int64-time-in)
[pages](https://stackoverflow.com/questions/41112178/google-big-query-gives-cannot-access-field-page-on-a-value-with-type-array)
on [Stackoverflow](https://stackoverflow.com/questions/38234894/working-with-structs-within-arrays-for-new-bigquery-standard-sql)
that solve specific cases of this error. While these pages are very helpful,
they do not fully explain what happens under the hood.
In this post, we will attempt to understand what exactly is causing this error and how would
we solve it.

Ideally, we would look at the code or the storage structure of the database itself to answer such
questions. Since we cannot do that, we are going to try a different approach.
Instead of looking at correct answers that solve our specific problems, we are going to attempt
to recreate the Example Dataset 1 using a (standard) SQL query. I hope that by performing this
exercise, you will understand how the nested and repeated fields are stored and why the error
is justified.

Our aim at the end of this post is to be able to query over nested and repeated fields
in standard SQL with ease and confidence.

### Data Types
Like most programming languages, BQ has [data types](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#array-type) -- both simple and complex.
Integers are an example of a simple data type.
[Arrays](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#array-type)
and
[structs](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#struct-type)
 are examples of complex data types.
 In the standard SQL flavor, arrays and structs enable the nested and repeated fields structure
in a BQ table. This is why the error message you see above has the words `ARRAY` and `STRUCT` in it.

Before we begin our exercise to recreate Example Dataset 1, let's familiarize ourselves with
`ARRAY`s and `STRUCT`s.






#### Footnotes
[^1]: `Flatten Results` with Legacy SQL.

    It's possible to turn the `Flatten Results` option off but it requires to send the result to
    a table. Often, a data scientist/analyst needs to incrementally build a SQL query, starting with
    a simple query and the refining the query based on the results of the simpler query.
    Sending the result (which can often be very large) to a table is tedious and slow.