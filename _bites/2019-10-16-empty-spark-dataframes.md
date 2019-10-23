---
layout: post
identifier: empty-spark-dataframes
title: "Empty spark dataframes"
date: 2019-10-16
comments: true
---
Creating an empty spark dataframe is a bit tricky. Let's see some examples.
First, let's create a `SparkSession` object to use.
```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName('my_app').getOrCreate()
```
The following command fails because the schema cannot be inferred.
We can make it work by specifying the schema as a string.
```python
spark.createDataFrame([])  # fails!
# ...
# ValueError: can not infer schema from empty dataset

df = spark.createDataFrame([], 'a INT, b DOUBLE, c STRING')  # works!
df.dtypes
# [('a', 'int'), ('b', 'double'), ('c', 'string')]
```
Things get a little bit more interesting when we create a spark dataframe from a pandas
dataframe.
```python
x = pd.DataFrame({'a': [1.0], 'b': [1.0]})  # works as expected
print(x.dtypes)  # pandas dataframe has a schema
# a    float64
# b    float64
# dtype: object
spark.createDataFrame(x).dtypes  # no need to specify schema because it can be inferred
# [('a', 'double'), ('b', 'double')]
```
An empty pandas dataframe has a schema but spark is unable to infer it.
```python
y = pd.DataFrame({'a': [], 'b': []})
print(y.dtypes)  # default dtype is float64
# a    float64
# b    float64
# dtype: object
spark.createDataFrame(y)  # fails!
# ...
# ValueError: can not infer schema from empty dataset
```
Now, funnily enough, spark completely ignores an empty pandas dataframe's schema.
```python
# works as expected
spark.createDataFrame(pd.DataFrame({'a': [], 'b': []}), 'a INT, b DOUBLE').dtypes
# [('a', 'int'), ('b', 'double')]

# also works!
spark.createDataFrame(pd.DataFrame({'a': [], 'b': []}), 'a INT').dtypes
# [('a', 'int')]

# also works!
spark.createDataFrame(pd.DataFrame({'a': [], 'b': []}), 'b INT').dtypes
# [('b', 'int')]

# still works!
spark.createDataFrame(pd.DataFrame({'a': [], 'b': []}), 'c INT').dtypes
# [('c', 'int')]
```