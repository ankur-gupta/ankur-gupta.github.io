---
layout: post
identifier: spark-catalog-api
title: "Spark Catalog API"
date: 2019-10-17
comments: true
---
Spark Catalog API (`spark.catalog.*`) provides good information about what tables are cached
and which UDF functions are available.
```python
from pyspark.sql.functions import udf
my_square = udf(lambda x: x * x, 'double')
spark.udf.register('my_square', my_square)
spark.catalog.listFunctions()
# ...
# Function(name='my_square', description=None, className=None, isTemporary=True),
# ...
```
You can see a list of tables and see which ones are cached.
```python
df = spark.createDataFrame([(x, x) for x in range(10000)], 'a INT, b INT')
df.is_cached
# False
df.createOrReplaceTempView('dummy')
spark.catalog.listTables()
# [Table(name='dummy', database=None, description=None, tableType='TEMPORARY', isTemporary=True)]
spark.catalog.isCached('dummy')
# False
df.cache()
spark.catalog.isCached('dummy')
# True
```
Other functions such as `spark.catalog.clearCache()` could also be very useful.