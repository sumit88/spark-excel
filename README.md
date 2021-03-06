# Spark Excel Library

A library for querying Excel files with Apache Spark, for Spark SQL and DataFrames.

[![Build Status](https://travis-ci.org/crealytics/spark-excel.svg?branch=master)](https://travis-ci.org/crealytics/spark-excel)
[![Maven Central](https://img.shields.io/maven-central/v/com.crealytics/spark-excel_2.11.svg)](https://search.maven.org/artifact/com.crealytics/spark-excel_2.11/)


## Requirements

This library requires Spark 2.0+

## Linking
You can link against this library in your program at the following coordinates:

### Scala 2.11
```
groupId: com.crealytics
artifactId: spark-excel_2.11
version: 0.10.0
```

## Using with Spark shell
This package can be added to  Spark using the `--packages` command line option.  For example, to include it when starting the spark shell:

### Spark compiled with Scala 2.11
```
$SPARK_HOME/bin/spark-shell --packages com.crealytics:spark-excel_2.11:0.10.0
```

## Features
This package allows querying Excel spreadsheets as [Spark DataFrames](https://spark.apache.org/docs/latest/sql-programming-guide.html).

### Scala API
__Spark 2.0+:__


#### Create a DataFrame from an Excel file

```scala
import org.apache.spark.sql._

val spark: SparkSession = ???
val df = spark.read
    .format("com.crealytics.spark.excel")
    .option("sheetName", "Daily") // Required
    .option("useHeader", "true") // Required
    .option("treatEmptyValuesAsNulls", "false") // Optional, default: true
    .option("inferSchema", "false") // Optional, default: false
    .option("addColorColumns", "true") // Optional, default: false
    .option("startColumn", 0) // Optional, default: 0
    .option("endColumn", 99) // Optional, default: Int.MaxValue
    .option("timestampFormat", "MM-dd-yyyy HH:mm:ss") // Optional, default: yyyy-mm-dd hh:mm:ss[.fffffffff]
    .option("maxRowsInMemory", 20) // Optional, default None. If set, uses a streaming reader which can help with big files
    .option("excerptSize", 10) // Optional, default: 10. If set and if schema inferred, number of rows to infer schema from
    .option("skipFirstRows", 5) // Optional, default None. If set skips the first n rows and checks for headers in row n+1
    .option("workbookPassword", "pass") // Optional, default None. Requires unlimited strength JCE for older JVMs
    .schema(myCustomSchema) // Optional, default: Either inferred schema, or all columns are Strings
    .load("Worktime.xlsx")
```

For convenience, there is an implicit that wraps the `DataFrameReader` returned by `spark.read`
and provides a `.excel` method which accepts all possible options and provides default values:

```scala
import org.apache.spark.sql._
import com.crealytics.spark.excel._

val spark: SparkSession = ???
val df = spark.read.excel(
    sheetName = "Daily",  // Required
    useHeader = "true",  // Required
    treatEmptyValuesAsNulls = "false",  // Optional, default: true
    inferSchema = "false",  // Optional, default: false
    addColorColumns = "true",  // Optional, default: false
    startColumn = 0,  // Optional, default: 0
    endColumn = 99,  // Optional, default: Int.MaxValue
    timestampFormat = "MM-dd-yyyy HH:mm:ss",  // Optional, default: yyyy-mm-dd hh:mm:ss[.fffffffff]
    maxRowsInMemory = 20,  // Optional, default None. If set, uses a streaming reader which can help with big files
    excerptSize = 10,  // Optional, default: 10. If set and if schema inferred, number of rows to infer schema from
    skipFirstRows = 5,  // Optional, default None. If set skips the first n rows and checks for headers in row n+1
    workbookPassword = "pass"  // Optional, default None. Requires unlimited strength JCE for older JVMs
).schema(myCustomSchema) // Optional, default: Either inferred schema, or all columns are Strings
 .load("Worktime.xlsx")
```

#### Create a DataFrame from an Excel file using custom schema
```scala
import org.apache.spark.sql._
import org.apache.spark.sql.types._

val peopleSchema = StructType(Array(
    StructField("Name", StringType, nullable = false),
    StructField("Age", DoubleType, nullable = false),
    StructField("Occupation", StringType, nullable = false),
    StructField("Date of birth", StringType, nullable = false)))

val spark: SparkSession = ???
val df = spark.read
    .format("com.crealytics.spark.excel")
    .option("sheetName", "Info")
    .option("useHeader", "true")
    .schema(peopleSchema)
    .load("People.xlsx")
```

#### Write a DataFrame to an Excel file
```scala
import org.apache.spark.sql._

val df: DataFrame = ???
df.write
  .format("com.crealytics.spark.excel")
  .option("sheetName", "Daily")
  .option("preHeader", "Pre-header\tin\tcells\nand\trows") // Optional, default None. If set adds rows (split by "\n") and cells (split by "\t") before the column headers.
  .option("useHeader", "true")
  .option("dateFormat", "yy-mmm-d") // Optional, default: yy-m-d h:mm
  .option("timestampFormat", "mm-dd-yyyy hh:mm:ss") // Optional, default: yyyy-mm-dd hh:mm:ss.000
  .mode("overwrite")
  .save("Worktime2.xlsx")
```

## Building From Source
This library is built with [SBT](http://www.scala-sbt.org/0.13/docs/Command-Line-Reference.html).
To build a JAR file simply run `sbt assembly` from the project root.
The build configuration includes support for Scala 2.11.
