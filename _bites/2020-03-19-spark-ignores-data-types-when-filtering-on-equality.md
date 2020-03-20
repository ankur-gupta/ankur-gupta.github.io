---
layout: post
identifier: spark-ignores-data-types-when-filtering-on-equality
title: "Spark ignores data types when filtering on equality"
date: 2020-03-19
comments: true
---
This may be surprising to regular pandas users and may lead to unexpected or silent errors.

### The problem
Let's construct an example dataframe to demonstrate the problem. The following dataframe
has two columns -- column `x` has type integer, column `y` has type string. Column `x`
has an element `1` (integer) and column `y` has an element `"1"` (string).
```python
df = spark.createDataFrame([(1, 'a'), (2, 'b'), (3, '1')], 'x INT, y STRING')
df.dtypes
# [('x', 'int'), ('y', 'string')]
df.show()
# +---+---+
# |  x|  y|
# +---+---+
# |  1|  a|
# |  2|  b|
# |  3|  1|
# +---+---+
```
Let's say we want to count how many `1` (integer) values are in each column. We should get back
these results --  column `x` should have one instance of `1` (integer) and column `y` should have
zero instances of `1` (integer). But only one of these happen.
```python
df[df['x'].isin(1)].count()  # 1 (correct)
df[df['y'].isin(1)].count()  # 1 (incorrect)
```
We may suspect that the
[`pyspark.sql.Column.isin` method](https://github.com/apache/spark/blob/master/python/pyspark/sql/column.py#L429)
method has a bug. Let's check another way without using `.isin` method.
```python
df[df['x'] == 1].count()  # 1 (correct)
df[df['y'] == 1].count()  # 1 (incorrect)
```
We still get the same partially incorrect results. Let's check something else -- filter
for `"1"` (string) instead.
```python
df[df['x'].isin("1")].count()  # 1 (incorrect)
df[df['y'].isin("1")].count()  # 1 (correct)

df[df['x'] == "1"].count()  # 1 (incorrect)
df[df['y'] == "1"].count()  # 1 (correct)
```
It appears that (1) `.isin` is not the reason and (2) spark does not respect data types when
filtering. Python is a dynamically typed language (though it now supports optional type checking).
Perhaps that is causing this. Let's just use SQL (via python) instead and verify.
```python
df.createOrReplaceTempView('dummy')
spark.sql('SELECT * FROM dummy WHERE x = 1').count()  # 1 (correct)
spark.sql('SELECT * FROM dummy WHERE y = 1').count()  # 1 (incorrect)
spark.sql('SELECT * FROM dummy WHERE x = "1"').count()  # 1 (incorrect)
spark.sql('SELECT * FROM dummy WHERE y = "1"').count()  # 1 (correct)
```
We get the same results back -- data types are not respected.

### Let's just use Scala
Let's investigate if this happens in a statically typed language like Scala.
```scala
import spark.implicits._

val df = Seq(
    (1, "a"),
    (2, "b"),
    (3, "1")
).toDF("x", "y")

df.dtypes
// Array[(String, String)] = Array((x,IntegerType), (y,StringType))

df.filter($"x" === 1).count()    // 1 (correct)
df.filter($"y" === 1).count()    // 1 (incorrect)
df.filter($"x" === "1").count()  // 1 (incorrect)
df.filter($"y" === 1).count()    // 1 (correct)
```
Scala has the same problem!

### Plain-old SQL has the same behavior
Turns out we see the same behavior in at least some SQL systems. Let's use SQL (using `sqlite3`)
without using spark at all.
```python
import sqlite3
conn = sqlite3.connect(':memory:')
c = conn.cursor()
c.execute('CREATE TABLE dummy (x integer, y string)')
c.execute('INSERT INTO dummy VALUES (1, "a")')
c.execute('INSERT INTO dummy VALUES (2, "b")')
c.execute('INSERT INTO dummy VALUES (3, "1")')
conn.commit()
list(c.execute('SELECT * FROM dummy'))
list(c.execute('SELECT * FROM dummy WHERE x=1'))   # [(1, 'a')]  (correct)
list(c.execute('SELECT * FROM dummy WHERE y=1'))   # [(3, 1)]    (incorrect)
list(c.execute('SELECT * FROM dummy WHERE x="1"')) # [(1, 'a')]  (incorrect)
list(c.execute('SELECT * FROM dummy WHERE y="1"')) # [(3, 1)]    (correct)
conn.close()
```
Turns out SQL does not respect data types either!

### It gets worse! Even `pandas` has this bug.
Turns out that even `pandas` has this problem. For pyspark and SQL,
this problem appears to be a consistent design issue but for `pandas` this problem appears to be
bug instead of a design choice. I used `pandas` version `0.25.3` for this experiment.
```python
pdf['x'].isin([1]).sum()  # 1 (correct)
pdf['y'].isin([1]).sum()  # 0 (correct)
pdf['x'].isin(["1"]).sum()  # 1 (incorrect)
pdf['y'].isin(["1"]).sum()  # 1 (correct)
```
Note how _only_ the third line in the above snippet returns incorrect results. This indicates
that `pandas` ignores the data type sometimes but not always. Further, in the case of `pandas`
the problem is only in the `.isin` method and not in general (like in SQL and pyspark), as
shown by the following example that does not use `.isin`.
```python
# Results are correct when we don't use pd.Series.isin() method
pdf['x'].apply(lambda x: x == 1).sum()   # 1 (correct)
pdf['y'].apply(lambda y: y == 1).sum()   # 0 (correct)
pdf['x'].apply(lambda x: x == "1").sum() # 0 (correct)
pdf['y'].apply(lambda y: y == "1").sum() # 1 (correct)

(pdf['x'] == 1).sum()    # 1 (correct)
(pdf['y'] == 1).sum()    # 0 (correct)
(pdf['x'] == "1").sum()  # 0 (correct)
(pdf['y'] == "1").sum()  # 1 (correct)
```
Looking even deeper into the `pandas.Series.isin` method, we see that it relies upon the
[`pandas.core.algorithms.isin` function](https://github.com/pandas-dev/pandas/blob/v0.25.3/pandas/core/algorithms.py#L413).
[Lines 452-453](https://github.com/pandas-dev/pandas/blob/v0.25.3/pandas/core/algorithms.py#L452)
(in `pandas` version `0.25.3`) contain the following code (comments are mine):
```python
# Lines 452-453 in pandas 0.25.3
comps, dtype, _ = _ensure_data(comps)              # comps = elements of the column
values, _, _ = _ensure_data(values, dtype=dtype)   # values = list of values passed to `.isin`
```
As you can see, `values` get type casted into the `dtype` of the column. We can verify this
by actually running the code ourselves.
```python
from pandas.core.algorithms import _ensure_data
comps = pdf['x']
values = ["1"]  # str
comps, dtype, _ = _ensure_data(comps)
print(dtype)  # int64
values, _, _ = _ensure_data(values, dtype=dtype)
print(values)  # [1]
print(values.dtype)  # int64
```
The reason why this does not affect column `y` in `pdf` above is because `_ensure_data` returns
`dtype=object` for column `y`. Searching `pandas` GitHub issues, we see that
[this bug](https://github.com/pandas-dev/pandas/issues/22160)
has been brought up before but was somehow ignored.

### Takeaway
Spark and SQL ignore data types when filtering on equality. This seems to be a design issue and
is consistent throughout (or at least as far as I can see). Pandas, on the other hand, exhibits
this problem inconsistently which may lead to complacency.

For end users, the best way to prevent this mistake is to always manually ensure that the data
types of the column match the data type of every value in a list values used for equality
filtering.
