---
layout: post
identifier: handling-avro-files-in-python
title: Handling Avro files in Python
author: Ankur Gupta
date: 2019-11-29
comments: true
---

[Apache Avro](https://avro.apache.org/docs/current/) is a data serialization format.
We can store data as `.avro` files on disk. Avro files are typically used with Spark but Spark
is completely independent of Avro. Avro is a row-based format that is suitable for evolving
data schemas. One benefit of using Avro is that schema and metadata travels with the data.
If you have an `.avro` file, you have the schema of the data as well.
The [Apache Avro Specification](https://avro.apache.org/docs/current/spec.html) provides
easy-to-read yet detailed information.

Sadly, using Avro files with python is unnecessarily error-prone, especially for a beginner.
In this post, we will describe the common errors and their solutions.


## Official Avro Packages
### Python 2 _vs_ Python 3
Python 2 is [end-of-life](https://pythonclock.org/). You should _not_ be writing Python 2 code.
However, the official Avro
[Getting Started (Python) Guide](https://avro.apache.org/docs/current/gettingstartedpython.html)
is written for Python 2 and will fail with Python 3. The problem goes deeper than merely
outdated official documentation.

There are two official python packages for handling Avro, one for Python 2 and one for Python 3.
The packages have different names, which is unusual for the python ecosystem[^1].

| Python Version | Package Name                                             | Example           |
|----------------|----------------------------------------------------------|-------------------|
| Python 2       | [`avro`](https://pypi.org/project/avro/)                 | avro.schema.parse |
| Python 3       | [`avro-python3`](https://pypi.org/project/avro-python3/) | avro.schema.Parse |

The first problem is that you can actually install `avro`, which is intended for Python 2, in a
Python 3 (virtual) environment. But, when you try to use `avro` in Python 3, it
[fails](https://stackoverflow.com/questions/41924355/syntaxerror-invalid-syntax-when-i-import-avro-in-python3).
```bash
# Inside a fresh Python 3 virtual environment
python  --version
# Python 3.7.3

# You can successfully install `avro` in a Python 3 virtualenv even though `avro` is not
# compatible with Python 3.
pip install avro

# Fails when you try to use it!
python -c "import avro.schema"
# Traceback (most recent call last):
#   File "<string>", line 1, in <module>
#   File "/Users/ankur/.virtualenvs/python3-test-env/lib/python3.7/site-packages/avro/schema.py", line 383
#     except Exception, e:
#                     ^
# SyntaxError: invalid syntax
```
Thankfully, the reverse problem is very unlikely. You should _not_ be able to install
`avro-python3`, which is intended for Python 3, within a Python 2 (virtual) environment, by default.
```bash
# Inside a fresh Python 2 virtual environment
python  --version
# Python 2.7.16

# Fails at the installation step itself, thankfully!
pip install avro-python3
# DEPRECATION: Python 2.7 will reach the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 won't be maintained after that date. A future version of pip will drop support for Python 2.7. More details about Python 2 support in pip, can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support
# Collecting avro-python3
#   Downloading https://files.pythonhosted.org/packages/d1/55/4c2e6fecf06cbaa68e0abaf12e1e965969872ed16da3674e6245cab0d5e2/avro-python3-1.9.0.tar.gz
# ERROR: Package 'avro-python3' requires a different Python: 2.7.16 not in '>=3.4'
```
Even if you install the correct Avro package for your Python environment, the API
[differs](https://stackoverflow.com/questions/41405729/module-avro-schema-has-no-attribute-parse)
between `avro` and `avro-python3`. As an example, for Python 2 (with `avro` package), you need to
use the function `avro.schema.parse` but for Python 3 (with `avro-python3` package), you need to use
the function `avro.schema.Parse`.

While the difference in API does _somewhat_ justify having different package names, this still
causes unnecessary confusion. The confusion is exacerbated because the
[official guide](https://avro.apache.org/docs/current/gettingstartedpython.html), which still
uses Python 2, never mentions that the instructions are _only_ applicable to Python 2.

In the rest of this post, we will only use Python 3 with `avro-python3` package because Python 2
is EOL.

### Working Example
This is an example usage of `avro-python3` in a Python 3 environment.
```python
# Python 3 with `avro-python3` package available
import copy
import json
import avro
from avro.datafile import DataFileWriter, DataFileReader
from avro.io import DatumWriter, DatumReader

# Note that we combined namespace and name to get "full name"
schema = {
    'name': 'avro.example.User',
    'type': 'record',
    'fields': [
        {'name': 'name', 'type': 'string'},
        {'name': 'age', 'type': 'int'}
    ]
}

# Parse the schema so we can use it to write the data
schema_parsed = avro.schema.Parse(json.dumps(schema))

# Write data to an avro file
with open('users.avro', 'wb') as f:
    writer = DataFileWriter(f, DatumWriter(), schema_parsed)
    writer.append({'name': 'Pierre-Simon Laplace', 'age': 77})
    writer.append({'name': 'John von Neumann', 'age': 53})
    writer.close()

# Read data from an avro file
with open('users.avro', 'rb') as f:
    reader = DataFileReader(f, DatumReader())
    metadata = copy.deepcopy(reader.meta)
    schema_from_file = json.loads(metadata['avro.schema'])
    users = [user for user in reader]
    reader.close()

print(f'Schema that we specified:\n {schema}')
print(f'Schema that we parsed:\n {schema_parsed}')
print(f'Schema from users.avro file:\n {schema_from_file}')
print(f'Users:\n {users}')

# Schema that we specified:
#  {'name': 'avro.example.User', 'type': 'record',
#   'fields': [{'name': 'name', 'type': 'string'}, {'name': 'age', 'type': 'int'}]}
# Schema that we parsed:
#  {"type": "record", "name": "User", "namespace": "avro.example",
#   "fields": [{"type": "string", "name": "name"}, {"type": "int", "name": "age"}]}
# Schema from users.avro file:
#  {'type': 'record', 'name': 'User', 'namespace': 'avro.example',
#   'fields': [{'type': 'string', 'name': 'name'}, {'type': 'int', 'name': 'age'}]}
# Users:
#  [{'name': 'Pierre-Simon Laplace', 'age': 77}, {'name': 'John von Neumann', 'age': 53}]
```

#### Issue with `name`, `namespace`, and _full name_
An interesting thing to note is what happens with the `name` and `namespace` fields.
The schema we specified has the full name of the schema that has both `name` and `namespace`
combined, _i.e._, `'name': 'avro.example.User'`. However, after parsing with `avro.schema.Parse()`,
the `name` and `namespace` are separated into individual fields. Further, when we read back the
schema from the `users.avro` file, we also get the `name` and `namespace` separated
into individual fields.

Avro specification, for some reason, uses the `name` field for both the _full name_ and the
_partial name_. In other words, the `name` field can either contain the full name or only the
partial name. Ideally, Avro specification should have kept `partial_name`, `namespace`, and
`full_name` as separate fields.

This _behind-the-scene separation_ and _in-place modification_ may cause unexpected errors if
your code depends on the exact value of `name`. One common use case is when you're handling lots of
different schemas and you want to identify/index/search by the schema name.

A **best practice** to guard against possible `name` errors is to always
parse a `dict` schema into a `avro.schema.RecordSchema` using `avro.schema.Parse()`. This will
generate the `namespace`, `fullname`, and `simple_name` (partial name), which you can then use
with peace of mind.
```python
print(type(schema_parsed))
# <class 'avro.schema.RecordSchema'>
print(schema_parsed.avro_name.fullname)
# avro.example.User
print(schema_parsed.avro_name.simple_name)
# User
print(schema_parsed.avro_name.namespace)
# avro.example
```

This problem of `name` and `namespace` deepens when we use a third-party package called
`fastavro`, as we will see in the next section.


## Third-party Avro Packages
### `fastavro`
While `avro-python3` is the official Avro package, it appears to be
[very slow](https://medium.com/@abrarsheikh/benchmarking-avro-and-fastavro-using-pytest-benchmark-tox-and-matplotlib-bd7a83964453).
This is because it is written in pure python.
In comparison, [`fastavro`](https://github.com/fastavro/fastavro) uses C extensions (with
regular CPython) making it much faster. Another benefit of using `fastavro` is that you can
install it the same way in both Python 2 and Python 3. `fastavro` API is also the
same[^2] for both Python 2 and 3.

We will use `fastavro 0.22.7` for the following discussion. First, let's use the
`fastavro.parse_schema()`. Unlike `avro.schema.Parse()`, `fastavro.parse_schema()` reads in a
schema `dict` and outputs another schema `dict`.
```python
import fastavro

# Namespace and name are combined to get "full name"
schema_together = {
    'name': 'avro.example.User',
    'type': 'record',
    'fields': [
        {'name': 'name', 'type': 'string'},
        {'name': 'age', 'type': 'int'}
    ]
}

# Namespace and name are separate
schema_separated = {
    'name': 'User',
    'namespace': 'avro.example',
    'type': 'record',
    'fields': [
        {'name': 'name', 'type': 'string'},
        {'name': 'age', 'type': 'int'}
    ]
}

# fastavro.parse_schema() accepts schema as a dict and returns parsed schema as another dict.
# The parsed schema combines name and namespace into "full name".
schema_together_parsed = fastavro.parse_schema(schema_together)
schema_separated_parsed = fastavro.parse_schema(schema_separated)
print(schema_separated_parsed == schema_together_parsed)
# True
print(schema_separated_parsed)
# {'type': 'record', 'name': 'avro.example.User',
# 'fields': [{'name': 'name', 'type': 'string'},
#            {'name': 'age', 'type': 'int'}],
# '__fastavro_parsed': True}
```

Parsing a schema `dict` is not really necessary to write data to disk. Parsing the schema
provides two benefits:
1. It helps us verify that the input schema `dict` is indeed valid
2. We get back a canonicalized (as per `fastavro`) schema `dict` as a result

But, there is one side effect. Parsed schema combines the `name` and `namespace`
into _full name_ and then stores the _full name_ in the `name` field. This is the opposite behavior
of `avro-python3` and just like with `avro-python3`, this _behind-the-scene, in-place modification_
can cause unexpected errors.
Finally, the schema that gets written to disk is whatever schema `dict`
we pass to the `fastavro.writer()`.
```python
# Continued from above

# User data to store.
users = [{'name': 'Pierre-Simon Laplace', 'age': 77},
         {'name': 'John von Neumann', 'age': 53}]

# Experiment 1: Write data using the schema with `name` and `namespace` combined.
with open('users.avro', 'wb') as f:
    fastavro.writer(f, schema_separated_parsed, users)

with open('users.avro', 'rb') as f:
    reader = fastavro.reader(f)
    users_read_back = [user for user in reader]
    metadata = copy.deepcopy(reader.metadata)
    writer_schema = copy.deepcopy(reader.writer_schema)
    schema_from_file = json.loads(metadata['avro.schema'])

print(writer_schema == schema_from_file)
# True
print(schema_from_file)
# {'type': 'record', 'name': 'avro.example.User',
#  'fields': [{'name': 'name', 'type': 'string'}, {'name': 'age', 'type': 'int'}]}
print(writer_schema)
# {'type': 'record', 'name': 'avro.example.User',
#  'fields': [{'name': 'name', 'type': 'string'}, {'name': 'age', 'type': 'int'}]}
```
`fastavro` seems to provide two fields that contain the schema -- `reader.writer_schema` and
`reader.metadata`. Metadata (`reader.metadata`) includes the schema as
`reader.metadata['avro.schema']`. In the above experiment,
both these sources of schema provide the exact same schema that has `name` and `namespace`
combined into a _full name_. But, this is not always the case, as we will see in the next
experiment.
```python
# Continued from above

# Experiment 2: Write data using the schema that has `name` and `namespace` separate.
# Use the unparsed schema that has name and namespace separate
with open('users.avro', 'wb') as f:
    fastavro.writer(f, schema_separated, users)

with open('users.avro', 'rb') as f:
    reader = fastavro.reader(f)
    users_read_back = [user for user in reader]
    metadata = copy.deepcopy(reader.metadata)
    writer_schema = copy.deepcopy(reader.writer_schema)
    schema_from_file = json.loads(metadata['avro.schema'])

print(writer_schema == schema_from_file)
# False
print(schema_from_file)
# {'name': 'User', 'namespace': 'avro.example', 'type': 'record',
#  'fields': [{'name': 'name', 'type': 'string'}, {'name': 'age', 'type': 'int'}]}
print(writer_schema)
# {'type': 'record', 'name': 'avro.example.User',
#  'fields': [{'name': 'name', 'type': 'string'}, {'name': 'age', 'type': 'int'}]}
```
The above experiment shows that `reader.writer_schema` and `reader.metadata['avro.schema']`
differ in whether or not `name` and `namespace` are combined together.

In both experiments, `reader.metadata['avro.schema']` is more faithful to the
schema `dict` we used to actually write the data. Therefore, it's a good practice to use
`reader.metadata['avro.schema']` instead of `reader.writer_schema` to get the schema.

#### `avro-python3` _vs_ `fastavro`
As we saw, `avro-python3` and `fastavro` have opposite behaviors when it comes to handling `name`,
`namespace`, and _full name_.

| Package        | Parsing function        | Behavior                         |
|----------------|-------------------------|----------------------------------|
| `avro-python3` | `avro.schema.Parse`     | Separates `name` and `namespace` |
| `fastavro`     | `fastavro.parse_schema` | Combines `name` and `namespace`  |

The handling of `name` and `namespace` may not bother you at all or you may find the above
difference in behavior unimportant. But, if your code expects either
exactly the _partial name_ or exactly the _full name_, then you may encounter errors.
One common error is when you prefix the `namespace` to an already fully qualified name in the
`name` field. I highly recommend exercising care when handling `name` and `namespace`
depending on the package you're using.

### Avro <> DataFrame
As we have seen above, Avro format simply requires a schema and a list of records. We don't need
a dataframe to handle Avro files. However, we can write a `pandas` dataframe into an Avro
file or read an Avro file into a `pandas` dataframe.
To begin with, we can always represent a dataframe as a list of records and vice-versa
1. List of records -- `pandas.DataFrame.from_records()` --> Dataframe
2. List of records <-- `pandas.DataFrame.to_dict(orient='records')` -- Dataframe

Using the two functions above in conjunction with `avro-python3` or `fastavro`, we can read/write
dataframes as Avro. The only additional work wewould need to do is to
inter-convert between `pandas` data types and
[Avro schema types](https://avro.apache.org/docs/current/spec.html#schema_primitive) ourselves.

An alternative solution is to use a third-party package called
[`pandavro`](https://github.com/ynqa/pandavro), which does some of this inter-conversion
for us[^3].
```python
import copy
import json
import pandas as pd
import pandavro as pdx
from avro.datafile import DataFileReader
from avro.io import DatumReader

# Data to be saved
users = [{'name': 'Pierre-Simon Laplace', 'age': 77},
         {'name': 'John von Neumann', 'age': 53}]
users_df = pd.DataFrame.from_records(users)
print(users_df)

# Save the data without any schema
pdx.to_avro('users.avro', users_df)

# Read the data back
users_df_redux = pdx.from_avro('users.avro')
print(type(users_df_redux))
# <class 'pandas.core.frame.DataFrame'>

# Check the schema for "users.avro"
with open('users.avro', 'rb') as f:
    reader = DataFileReader(f, DatumReader())
    metadata = copy.deepcopy(reader.meta)
    schema_from_file = json.loads(metadata['avro.schema'])
    reader.close()
print(schema_from_file)
# {'type': 'record', 'name': 'Root',
#  'fields': [{'name': 'name', 'type': ['null', 'string']},
#             {'name': 'age', 'type': ['null', 'long']}]}
```
In the above example, we didn't specify a schema ourselves and `pandavro` assigned the
`name` = `Root` to the schema. We can also provide a schema `dict` to `pandavro.to_avro()`
function, which will preserve the `name` and `namespace` faithfully.


## Avro with PySpark
Using Avro with PySpark is fraught with a sequence of issues. Let's see the common issues
step-by-step.

### Confusing official guide
The [official Spark documentation on Avro](https://spark.apache.org/docs/latest/sql-data-sources-avro.html)
contains two _seemingly_ contradictory claims. On one hand, the official documentation says

> Since Spark 2.4 release, Spark SQL provides built-in support for reading and writing Apache Avro data.

Then, in the next line, it says

> The `spark-avro` module is external and not included in `spark-submit` or `spark-shell` by default.

Perhaps, there is sufficient technical difference between the two claims to make them
consistent with each other. But, use of the word "built-in" is unnecessarily confusing.
I recommend that you disregard the first claim that mentions "built-in" support for Avro.
Only the second claim is true -- you need to provide the `spark-avro` package to Spark.
You can do this by providing the Maven coordinates in the form `groupId:artifactId:version`
as follows[^4],[^5]:
```shell
# Example 1
$SPARK_INSTALLATION/bin/pyspark --packages org.apache.spark:spark-avro_2.12:2.4.4

# Example 2
$SPARK_INSTALLATION/bin/pyspark --packages com.databricks:spark-avro_2.11:4.0.0
```
You can go to the [The Central Repository Search Engine](https://search.maven.org/search?q=spark-avro)
or [Maven Repository](https://mvnrepository.com/search?q=spark-avro) (recommended)
to find the versions. If you provide a Maven coordinate that doesn't exist on Maven, you will
get a dependency error.

You may need to clear the cache in `$HOME/.ivy2` to overcome some `unknown resolver null` issues,
as mentioned [here](https://github.com/databricks/spark-avro/issues/264) and
[here](https://discuss.cloudxlab.com/t/spark-launch-error-for-all-versions/1478/2).
You can delete `$HOME/.ivy2` folder completely to clear the cache but be aware that you will also
delete all other downloaded/installed dependencies if you do so.


### `spark-avro`: Databricks or Apache
The reason why we show two examples in the above snippet is because there are at least two
common instances of the `spark-avro` package.
It appears that the [original `spark-avro` package](https://github.com/databricks/spark-avro)
was written by Databricks and then donated to Apache Spark project. Spark 2.4.0 included support
for "built-in" for Avro and updated the `spark-avro` package to have new functionality and
better performance while still retaining
[backward API compatibility](https://databricks.com/blog/2018/11/30/apache-avro-as-a-built-in-data-source-in-apache-spark-2-4.html)
with the older Databricks' version of `spark-avro`.

Both versions of `spark-avro` are available to use. If you're on Spark 2.4.0 or higher,
you should use Apache Spark's `spark-avro`[^6]. If you're on Spark 2.4.0 or lower, you need to use
the Databricks' version.

There is still [one minor change](https://stackoverflow.com/questions/29759893/how-to-read-avro-file-in-pyspark)
you need to make to your code to switch between the Databricks' (older) and Apache Spark's (newer)
versions.
```python
# For Spark 2.4.0 and higher, use Apache Spark's version of spark-avro
df = spark.read.format('avro').load('path/to/avro/data')

# For lower than Spark 2.4.0, use Databricks's version of spark-avro
df = spark.read.format('com.databricks.spark.avro').load('path/to/avro/data')
```
If you don't use the correct string in `format()`, you may see an error like this.
```python
AnalysisException: 'Failed to find data source: avro.
Avro is built-in but external data source module since Spark 2.4.
Please deploy the application as per the deployment section of
"Apache Avro Data Source Guide".;'
```
Obviously, you also need to provide the corresponding `spark-avro` to Spark. As we saw, we can
simply provide the correct Maven coordinates to the intended `spark-avro` package. But, there is
one more glitch -- even if we provide valid Maven coordinates to `spark-avro` package that
installs successfully, we may see an error. Let's see this in the next section.

### Scala version is important
Note the Scala version in the Maven coordinates. In `org.apache.spark:spark-avro_2.12:2.4.4`,
the Scala version is 2.12 and in `com.databricks:spark-avro_2.11:4.0.0` the Scala version is
2.11. If you don't use the correct Scala version, you will find that the `spark-avro` package
installs correctly and the `pyspark` shell starts successfully but reading Avro data
[fails](https://stackoverflow.com/questions/55873023/how-to-use-spark-avro-package-to-read-avro-file-from-spark-shell).
```bash
# Run pyspark shell with Apache Spark's spark-avro package as mentioned in the official docs
$ $SPARK_INSTALLATION/bin/pyspark --packages org.apache.spark:spark-avro_2.12:2.4.4
# Everything successful!

# Inside the resulting pyspark shell
>>> df = spark.read.format("avro").load("users.avro")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  ...
py4j.protocol.Py4JJavaError: An error occurred while calling o35.load.
: java.util.ServiceConfigurationError: org.apache.spark.sql.sources.DataSourceRegister:
Provider org.apache.spark.sql.avro.AvroFileFormat could not be instantiated
    at java.util.ServiceLoader.fail(ServiceLoader.java:232)
    ...
# Failure only when you read the Avro data!
```
Turns out that the `pyspark` in the above example was built against Scala 2.11, as shown below.
But, we provided a `spark-avro` package that was built for Scala 2.12.
```bash
$ $SPARK_INSTALLATION/bin/pyspark --version
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.4
      /_/

Using Scala version 2.11.12, Java HotSpot(TM) 64-Bit Server VM, 1.8.0_221
Branch
Compiled by user  on 2019-08-27T21:21:38Z
Revision
Url
Type --help for more information.
```
This issue is especially egregious because the user is able to install `spark-avro` for the wrong
Scala version without any indication of error, only to fail at the last moment.
Once you know about this issue, it can be easily fixed by simply using the correct `spark-avro`
package for your `pyspark`'s Scala version.
```bash
# Run pyspark shell with the correct Scala version for spark-avro
$ $SPARK_INSTALLATION/bin/pyspark --packages org.apache.spark:spark-avro_2.11:2.4.4

# Inside the resulting pyspark shell
>>> df = spark.read.format("avro").load("users.avro")
>>> df.show()
+--------------------+---+
|                name|age|
+--------------------+---+
|Pierre-Simon Laplace| 77|
|    John von Neumann| 53|
+--------------------+---+
```

### Working example, finally!
For this example, we will use Scala 2.11, Spark 2.4.4, and Apache Spark's `spark-avro 2.4.4`
within a `pyspark` shell[^6].
```bash
$ $SPARK_INSTALLATION/bin/pyspark --packages org.apache.spark:spark-avro_2.11:2.4.4
```
Within the `pyspark` shell, we can run the following code to write and read Avro.
```python
# Data to store
users = [{'name': 'Pierre-Simon Laplace', 'age': 77},
         {'name': 'John von Neumann', 'age': 53}]

# Create a pyspark dataframe
users_df = spark.createDataFrame(users, 'name STRING, age INT')

# Write to a folder named users
users_df.write.format('avro').mode("overwrite").save('users-folder')

# Read the data back
users_df_redux = spark.read.format('avro').load('./users-folder')
```

## Conclusion
My experience in working with Avro format has been error-prone at every step of the way.
The `name` and `namespace` ambiguity lies in the Avro specification itself. This is further
exacerbated by the contrasting behavior of the two most common Avro packages
for python (without spark) -- `avro-python3` and `fastavro`. When trying to use the official
Avro package for python, the package name and API differences between Python 2 and Python 3
create unnecessary confusion. This makes it difficult to port code over from Python 2. And, even though
we should not be writing Python 2 code, the package name and API differences make it difficult
to write code that is both Python 2 and Python 3 compatible.

Using Avro with PySpark comes with its own sequence of issues that present themselves
unexpectedly. In contrast, using `parquet`, `json`, or `csv` with Spark is so much easier. There
is no need to install an external package to use these formats. In that sense,
support for `parquet`, `json`, or `csv` is truly _built-in_.

I wrote this post with the hope that it saves you some time, effort, and frustration.
I have tried to list all the issues and solutions that people encounter when using Avro with python.
If I missed something or if I made a mistake, please let me know in the comments. Please feel free
to share this post with others if they would find it useful.

## Footnotes
[^1]: You can perform `pip install pandas` for both Python 2 and Python 3. You don't have to change the name of the package from `pandas` to `pandas-python3` for Python 3.
[^2]: Based on my basic usage.
[^3]: [`pandavro`](https://github.com/ynqa/pandavro) makes some decisions while inferring schema such as making all columns nullable. This may not be what you want. For production systems, consider using a pre-determined, version-controlled Avro schema saved as a `.avsc` file or using a schema store.
[^4]: An alternative way to provide a list of packages to Spark is to set the environment variable `PYSPARK_SUBMIT_ARGS`, as mentioned [here](https://stackoverflow.com/questions/55947670/how-to-write-spark-dataframe-into-avro-file-format-in-jupyter-notebook). This may be more helpful with Jupyter.
[^5]: For Java or Scala, you can [list `spark-avro` as a dependency](https://stackoverflow.com/questions/53715347/spark-reading-avro-file).
[^6]: For Spark 2.4.0+, using the Databricks' version of `spark-avro` creates more problems. One common error is `java.lang.ClassNotFoundException: Failed to find data source: org.apache.spark.sql.avro.AvroFileFormat`.
