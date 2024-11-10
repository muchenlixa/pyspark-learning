## What is Spark anyway?

Spark is a platform for cluster computing.

Spark lets you spread data and computations over clusters with multiple nodes (each of node is a separate computer).

Splitting yo the data makes it easier to work with very large datasets because each node only works with a small amount of data.

---

As each node works on its own subset of the total data, it also carries out a part of the total calculations requried, so that both data processing and computation are performed in *parallel* over the cluster. It's a fact that parallel computation can make certain types of programming tasks much faster. 

However, with greater computing power comes greater complexity.

## Using Spark in Python

The first setp in using Spark is connecting to a cluster. 

In practice, the cluster will be hosted on a remote machine that's connected to all other nodes. There will be one computer, which called *master* that manages splitting up the data and the computations. The master is connected to the rest of the computers in the cluster, which are called *worker*. The master sends the workers data and calculations to run, and they send their results back to the master. 

## Examining the SparkContext

Spark is some serious software. It takes more time to start up than you might be used to. 

Running simpler computations might take longer than expected. This is because all the optimizations that Spark has under its hood are designed for complicated operations with big data sets. 

For simple or small problems, Spark may actually perform worse than some other solutions!

## Using DataFrames

Spark's core data structure is the Resillient Distributed Dataset (RDD). This is a low level object that lets Spark work its magic by splitting data across multiple nodes in the cluster. However, RDDs are hard to work with directly, so it might be easier to using the Spark DataFrame abstraction built on top of RDDs. 

Spark DataFrame was designed to behave a lot like a SQL table. Not only are they easier to understand, DataFrames are also more optimized for complicated operations than RDDs. 

---

To working with Spark DataFrames, you first have to create a `SparkSession` object from your `SparkContext`. 

You can think of the `SparkContext` as your connection to the cluster and the `SparkSession` as your interface with the connection. 

## Creating a SparkSession

Creating multiple `SparkSessions` and `SparkContexts` can cause issues, so **it's best practice to use the SparkSession.builder.getOrCreate() method**. This returns an existing SparkSession if there's already one in the environment, or creates a new one if necessary!

```python
from pyspark.sql import SparkSession

# Create my_spark
my_spark = SparkSession.builder.getOrCreate()

# Print my_spark
print(my_spark)
```

## Viewing tables

Once you've created a `SparkSession`, you can start poking around to see what data is in your cluster!

Your `SparkSession` has an attribute called `catalog` which lists all the data inside the cluster. This attribute has a few methods for extracting different pieces of information. 

One of the most useful is the `.listTables()` method, which returns the names of all the tables in your cluster as a list.

```python
# Print the tables in the catalog
print(spark.catalog.listTables())
```

## Write SQL queries

One of te advantages of the DataFrame interface is that you can run SQL on the tables in your Spark cluster. 

Running a query on table is as easy as using the `.sql()` method on your `SparkSession`. This method takes a string containing the query and then returns a DataFrame with the results!

```python
query = "SELECT * FROM flights LIMIT 10"

# Get the first 10 rows of flights
flights10 = spark.sql(query)

# Show the results
flights10.show()
```

## Pandafy a Spark DataFrame

Suppose you've run a query on your huge dataset and aggregated it down to something a little more manageable. 

Sometimes it makes sense to then take that table and work iwth it locally using `pandas`. Spark DataFrames make that easy with `.toPandas()` method. Calling this method on Spark DataFrame returns the corresponding `pandas` DataFrame.

```python
query = "SELECT origin, dest, COUNT(*) as N FROM flights GROUP BY origin, dest"

# Run the query
flight_counts = spark.sql(query)

# Convert the results to a pandas DataFrame
pd_counts = flight_counts.toPandas()

# Print the head of pd_counts
print(pd_counts.head())
```

## Put some Spark in your data

You can also go the other direction by puting a `pandas` DataFrame into a Spark cluster! The `SparkSession` class has a method for this as well. 

The `.createDataFrame()` method takes a `pandas` DataFrame and returns a Spark DataFrame. 

---

**Remember:** the output of this method is stored locally not in the `SparkSession` catalog. This means that you can use all Spark DataFrame methods on it, but you can't access the data in other contexts. The SQL query (using `.sql()` method) that references your DataFrame will throw an error. To access the data in this way, you have to save it as a temporary table.

You can use `.createTempView()` or `.createOrReplaceTempView()` Spark DataFrame methods to handle the temporary table. These methods register the DataFrame as a table in the catalog, but as this table is temporary, it can only be accessed from the specific `SparkSession` used to create the Spark DataFrame.

![spark-figure](/spark_figure.png)

```python
# Create pd_temp
pd_temp = pd.DataFrame(np.random.random(10))

# Create spark_temp from pd_temp
spark_temp = spark.createDataFrame(pd_temp)

# Examine the tables in the catalog
print(spark.catalog.listTables())

# Add spark_temp to the catalog
spark_temp.createOrReplaceTempView("temp")

# Examine the tables in the catalog again
print(spark.catalog.listTables())
```

## Dropping the middle man

It's possible to work with `pandas` and load the data to Spark, but why deal with `pandas` at all?

The `SparkSession` has a `.read` attribute which has several methods for reading different data sources into SparkDataFrames. Using these you can create a DataFrame from data files just like with regular `pandas` DataFrames!

```python
# Don't change this file path
file_path = "/usr/local/share/datasets/airports.csv"

# Read in the airports data
airports = spark.read.csv(file_path, header=True)

# Show the data
airports.show()
```