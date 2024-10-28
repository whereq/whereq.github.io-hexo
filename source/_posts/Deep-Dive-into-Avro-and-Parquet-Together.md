---
title: Deep Dive into Avro and Parquet Together
date: 2024-10-28 17:33:14
categories:
- Deep Dive
- Avro
- Parquet
tags:
- Deep Dive
- Avro
- Parquet
---

- [Introduction](#introduction)
- [Architecture Overview](#architecture-overview)
- [Data Pipeline Design](#data-pipeline-design)
  - [Ingesting Data from Messaging System](#ingesting-data-from-messaging-system)
  - [Processing and Converting Avro to Parquet](#processing-and-converting-avro-to-parquet)
  - [Saving to Big Data Storage (S3)](#saving-to-big-data-storage-s3)
- [Understanding the Structural Differences](#understanding-the-structural-differences)
  - [Avro to Parquet Conversion](#avro-to-parquet-conversion)
  - [Reading from Parquet and Converting Back to Avro](#reading-from-parquet-and-converting-back-to-avro)
- [Parquet’s Row Groups and Column Chunks](#parquets-row-groups-and-column-chunks)
  - [ Ensuring Data Belongs to the Same Record](#-ensuring-data-belongs-to-the-same-record)
  - [Reading from Parquet and Converting Back to Avro](#reading-from-parquet-and-converting-back-to-avro-1)
- [What if Null Values in Parquet](#what-if-null-values-in-parquet)
  - [1. **Null Values in Columns**](#1-null-values-in-columns)
  - [2. **Encodings for Efficient Storage**](#2-encodings-for-efficient-storage)
  - [3. **Metadata Tracking and Optional Fields**](#3-metadata-tracking-and-optional-fields)
  - [Example Scenario](#example-scenario)
  - [Example in Spark](#example-in-spark)
- [Real-Life Production Sample Code in Spark](#real-life-production-sample-code-in-spark)
  - [ Reading from Kafka](#-reading-from-kafka)
  - [ Converting Avro to Parquet](#-converting-avro-to-parquet)
  - [ Writing to S3](#-writing-to-s3)
- [Conclusion](#conclusion)

---

<a name="introduction"></a>
## Introduction

In modern data architectures, Avro data is often used in messaging systems like Apache Kafka due to its compact binary format and schema evolution support. However, in data lakes or data warehouses, Parquet is preferred for storage due to its columnar format and efficient read performance. This article covers the architecture, pipeline design, and real-life sample code for implementing a pipeline that ingests Avro data from a messaging system and saves it in Parquet format in a big data storage solution like Amazon S3.

---

<a name="architecture-overview"></a>
## Architecture Overview

This section provides an overview of the components in a typical architecture that ingests Avro from a messaging system and saves it as Parquet in big data storage.

**Diagram: Avro to Parquet Pipeline Architecture**
```
    Messaging System (Kafka)  ---->  Stream Processing (Spark)  ---->  S3 Storage (Parquet Format)
```

<a name="avro-and-parquet-formats"></a>###  Avro and Parquet Formats

- **Avro**: A row-based binary format, commonly used for serializing data in messaging systems due to its compact size and schema evolution capabilities.
- **Parquet**: A columnar storage format optimized for reading specific columns, making it highly efficient for analytics and big data storage solutions.

<a name="role-of-messaging-systems"></a>###  Role of Messaging Systems

Messaging systems like **Apache Kafka** are widely used to transport streaming data across systems in real time. Kafka provides fault-tolerance and durability, ensuring data is safely delivered to downstream applications for processing.

<a name="storage-in-big-data-storage-solutions"></a>###  Storage in Big Data Storage Solutions

Big data storage solutions, such as **Amazon S3**, offer durable and cost-effective storage for large-scale data. Storing data in Parquet format on S3 enables easy integration with data analytics frameworks like Spark, Presto, and Hive.

---

<a name="data-pipeline-design"></a>
## Data Pipeline Design

The pipeline includes the following key stages:

1. **Data Ingestion**: Consuming Avro-encoded messages from Kafka.
2. **Processing and Transformation**: Converting Avro data to Parquet format.
3. **Storage**: Saving the transformed Parquet files to Amazon S3.

<a name="ingesting-data-from-messaging-system"></a>
### Ingesting Data from Messaging System

The first step in the pipeline is ingesting data from a messaging system like Kafka. Spark Structured Streaming can be used to consume Avro-encoded messages from Kafka topics in real-time.

**Diagram: Ingesting Avro Data from Kafka**
```
Kafka Topic (Avro Messages)  --->  Spark Structured Streaming  --->  Data Transformation
```

<a name="processing-and-converting-avro-to-parquet"></a>
### Processing and Converting Avro to Parquet

Spark Structured Streaming allows us to process and transform the incoming Avro data. To convert Avro to Parquet, the schema needs to be mapped, and the data is then serialized to the Parquet format.

**Diagram: Converting Avro to Parquet in Spark**
```
Avro Data (Spark DataFrame)  --->  Schema Mapping  --->  Parquet Data
```

<a name="saving-to-big-data-storage-s3"></a>
### Saving to Big Data Storage (S3)

Once converted to Parquet, the data can be written to Amazon S3 in a structured format. Parquet files in S3 are organized by partitions (e.g., by date or another logical grouping) to optimize retrieval and querying performance.

**Diagram: Saving to S3**
```
Parquet Data (Partitioned by Date)  --->  Amazon S3 Bucket  --->  Analytics-ready Storage
```

---

<a name="understanding-the-structural-differences"></a>
## Understanding the Structural Differences

In Avro and Parquet formats, the **data organization** is fundamentally different:

- **Avro**: A row-based format where each record is serialized as a single entity, storing all fields of each row together.
- **Parquet**: A columnar storage format where data is grouped by columns rather than rows, optimizing for efficient reads on specific columns (ideal for analytical workloads).

To illustrate, let’s consider an Avro record with two attributes:
```json
{
  "name": "John Doe",
  "age": 30
}
```

In Avro, the above record would be stored as a single contiguous block of binary data, with fields appearing in the order defined by the schema.

When converting this record to **Parquet**, the storage layout changes to a **column-oriented format**. Parquet organizes data by columns:
- Each column (e.g., "name" and "age") is stored separately in **column chunks**.
- Parquet organizes records into **row groups**, where each row group has column chunks for each column defined in the schema.

**Diagram: Avro to Parquet Conversion for Record `{ "name": "John Doe", "age": 30 }`**

| **Parquet Column Chunk** | **Data Stored**     |
|--------------------------|---------------------|
| Name Column Chunk        | John Doe           |
| Age Column Chunk         | 30                 |

In a more extensive dataset with thousands of records, Parquet’s layout significantly optimizes read performance because each column chunk can be accessed independently. For instance, if only the "age" column is required for analysis, Parquet reads only the relevant chunk, skipping over unrelated columns like "name."

---

<a name="avro-to-parquet-conversion"></a>
### Avro to Parquet Conversion

The steps for converting Avro to Parquet in a system like Apache Spark generally follow this sequence:

1. **Avro Data Loading**:
   - Avro-encoded data is parsed and loaded into an intermediate in-memory structure, such as a DataFrame in Spark.
   - Avro schema is either read from the Avro file (or topic in case of Kafka) or supplied separately.

2. **Schema Mapping**:
   - Spark maps the Avro schema to an equivalent internal schema (e.g., Spark schema), which is essential to support Parquet’s columnar structure.
   - Fields in the Avro schema (like "name" and "age") are mapped to equivalent columns.

3. **Data Conversion**:
   - Each Avro record is then serialized into a format compatible with Parquet’s column-oriented layout.
   - Each attribute is transformed to be written in column chunks rather than row-by-row.

4. **Data Writing (Parquet)**:
   - Parquet organizes data into **row groups**, and within each row group, column data is stored in contiguous chunks.
   - Additional metadata (e.g., min, max, dictionary encoding) is recorded for each column chunk, which is useful for optimizations in querying.

**Example**:
In Spark, a sample code to convert Avro to Parquet may look like this:

```java
// Load Avro data into a DataFrame
Dataset<Row> avroDF = spark.read().format("avro").load("path/to/avro/file");

// Write data to Parquet format
avroDF.write().parquet("path/to/output/parquet");
```

This process reorganizes the data from a row-based layout (Avro) to a columnar layout (Parquet).

---

<a name="reading-from-parquet-and-converting-back-to-avro"></a>
### Reading from Parquet and Converting Back to Avro

When reading Parquet data and converting it back to Avro, the system reverses the process, transforming columnar data back into row-based data.

**Steps**:
1. **Load Parquet Data**:
   - Parquet files are read into memory. In a Spark-based setup, this would involve loading Parquet files into a DataFrame.

2. **Column Extraction**:
   - Parquet retrieves data by accessing each relevant column chunk. Parquet's metadata allows efficient navigation to the specific data of interest.

3. **Row Assembly**:
   - As each column’s data is accessed, the system assembles rows by gathering data from each column chunk.
   - The row-based structure compatible with Avro is recreated by combining values from each column for every row.

4. **Schema Mapping**:
   - Parquet columns are mapped to Avro fields based on a schema. This schema alignment ensures that columns match Avro’s row-oriented requirements.
   - The DataFrame (or equivalent data structure) is serialized back into the Avro format.

5. **Write to Avro**:
   - The row-based structure is serialized in Avro’s binary format, and the schema is either embedded or referenced as needed.

**Example Code: Converting Parquet to Avro in Spark**

```java
// Load Parquet data into a DataFrame
Dataset<Row> parquetDF = spark.read().parquet("path/to/parquet/file");

// Write data back to Avro format
parquetDF.write().format("avro").save("path/to/output/avro");
```

This code converts the columnar Parquet format back into Avro’s row-based format.

---

<a name="parquets-row-groups-and-column-chunks"></a>
## Parquet’s Row Groups and Column Chunks

In Parquet, data is stored in **row groups**. Each row group contains data for a set number of rows, and within each row group, data is stored in **column chunks** (one for each column). The row group is Parquet’s key structural element that ensures the association between column data across multiple rows:

- **Row Group**: A set of rows is organized as a contiguous data block. Each row group in Parquet represents a batch of rows from the original dataset.
- **Column Chunk**: Within each row group, each column’s data is stored in a separate column chunk.

For example, if we have a dataset with 10,000 records, it might be divided into several row groups (say 5,000 rows per row group). Each row group then contains the data for the columns `name` and `age` in its respective column chunks.

### <a name="ensuring-data-belongs-to-the-same-record"></a> Ensuring Data Belongs to the Same Record

To ensure data from each column belongs to the same record, Parquet relies on **row ordering** within each row group. The ordering within each column chunk of a row group matches across all columns:

- **Ordering**: Each row in a row group corresponds to the same index across all column chunks within that row group. For instance, the first value in the `name` column chunk of a row group corresponds to the first value in the `age` column chunk for the same row group.
- **Offset Metadata**: Parquet uses metadata about the offsets within each column chunk to ensure each row can be reconstructed correctly when reading.

So, in your example, the row group might look like this:

| Row Index | Name Column Chunk | Age Column Chunk |
|-----------|--------------------|------------------|
| Row 1     | "John Doe"        | 30              |
| Row 2     | "Jane Smith"      | 25              |
| Row 3     | "Alice Brown"     | 28              |

Each entry in the `name` and `age` column chunks corresponds by index. Parquet uses metadata within the row group and column chunks to indicate where each row begins and ends, making sure that during reads, each row’s data across columns is aligned.

<a name="reading-from-parquet-and-converting-back-to-avro"></a>
###  Reading from Parquet and Converting Back to Avro

When reading Parquet data back into a row-based format (like Avro), the system iterates over each row group:

- **Row Assembly**: For each row in a row group, Parquet retrieves the respective values from each column chunk based on their row index. This reassembles each row’s fields together to form the original record.
- **Schema Mapping**: The column metadata stored in Parquet includes the schema, allowing the correct assignment of values back to each field in the record.

<a name="what-if-null-values-in-parquet"></a>
## What if Null Values in Parquet
In Parquet, each **column chunk** within a **row group** holds data in columnar format, and Parquet efficiently handles cases where some rows may lack a value for a given column by using specific encoding and metadata, rather than empty placeholders. Here’s how it manages the scenario:

### 1. **Null Values in Columns**
   If a column lacks a value for a particular row, Parquet does not insert an empty placeholder. Instead:
   - **Definition Levels** are used to indicate whether a value is present or null. Each level specifies if a field has a value or is missing at that row.
   - **Repetition Levels** track nested structures within rows, so nested nulls and repeated fields can be handled in one scan.

### 2. **Encodings for Efficient Storage**
   Parquet uses encodings like **Run-Length Encoding (RLE)** and **Dictionary Encoding** to represent repeated or missing values compactly:
   - **Run-Length Encoding** groups consecutive null values (or any repeating values) into a single representation, saving space when large chunks are missing in a column.
   - **Dictionary Encoding** can efficiently map common values (like repeated nulls or default values) using dictionary lookup tables.

### 3. **Metadata Tracking and Optional Fields**
   - If a field is defined as `optional` in the schema, it allows a row to be saved without a value for that field. Parquet only includes these rows in the row group metadata, which helps query engines like Spark identify missing values.

### Example Scenario

Let’s say we have a Parquet file with three rows and three columns, where one column lacks values for some rows.

| Row Group | Column A | Column B | Column C |
|-----------|----------|----------|----------|
| Row 1     | Value 1  | Null     | Value 3  |
| Row 2     | Value 4  | Value 5  | Null     |
| Row 3     | Null     | Value 6  | Value 7  |

In this case:
- **Definition Levels** for `Column B` and `Column C` would show which rows contain nulls.
- The data structure itself would store only the non-null values, reducing storage without explicit placeholders.

This approach is particularly useful when handling sparse data, as it minimizes the storage footprint and increases reading efficiency.

### Example in Spark

Here’s how Spark handles this under the hood when reading Parquet data and converting it back to Avro:

```java
// Reading from Parquet
Dataset<Row> parquetDF = spark.read().parquet("path/to/parquet/file");

// Writing back to Avro, reassembling rows
parquetDF.write().format("avro").save("path/to/output/avro");
```

During this process, Spark ensures that each row’s values across columns are correctly aligned by leveraging Parquet’s row group and column chunk metadata. This alignment allows it to convert back to Avro’s row-based format without losing the association between `name` and `age` in each record.

---

<a name="real-life-production-sample-code-in-spark"></a>
## Real-Life Production Sample Code in Spark

The following sample code demonstrates how to implement this pipeline in **Apache Spark** with **Structured Streaming**.

### <a name="reading-from-kafka"></a> Reading from Kafka

Use Spark to read Avro data from a Kafka topic in real-time.

```java
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.avro.functions.from_avro;

public class AvroToParquetPipeline {

    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
                .appName("Avro to Parquet Pipeline")
                .getOrCreate();

        String kafkaBootstrapServers = "kafka-broker1:9092,kafka-broker2:9092";
        String kafkaTopic = "avro-topic";

        Dataset<Row> avroDF = spark
                .readStream()
                .format("kafka")
                .option("kafka.bootstrap.servers", kafkaBootstrapServers)
                .option("subscribe", kafkaTopic)
                .load()
                .selectExpr("CAST(value AS BINARY) as avroData");

        // Define the Avro schema (can be fetched from Schema Registry if available)
        String avroSchema = "{ \"type\": \"record\", \"name\": \"User\", \"fields\": [ {\"name\": \"id\", \"type\": \"int\"}, {\"name\": \"name\", \"type\": \"string\"}, {\"name\": \"age\", \"type\": \"int\"} ] }";

        Dataset<Row> parsedDF = avroDF
                .select(from_avro(avroDF.col("avroData"), avroSchema).as("data"))
                .select("data.*");

        parsedDF.printSchema(); // Debugging schema after parsing Avro
    }
}
```

### <a name="converting-avro-to-parquet"></a> Converting Avro to Parquet

Once we have parsed the Avro data into a DataFrame, we can convert it into Parquet format.

```java
Dataset<Row> parquetDF = parsedDF.repartition(1); // Optional: Customize partitioning as needed
```

### <a name="writing-to-s3"></a> Writing to S3

Writing the data to Amazon S3 requires specifying the destination path and configuring partitioning if necessary.

```java
import org.apache.spark.sql.streaming.Trigger;

String s3Path = "s3a://your-bucket/path/to/destination/";

parquetDF.writeStream()
        .format("parquet")
        .option("path", s3Path)
        .option("checkpointLocation", "s3a://your-bucket/path/to/checkpoint/")
        .partitionBy("date") // Optional: Partitioning by a specific column (e.g., date)
        .trigger(Trigger.ProcessingTime("5 minutes")) // Controls batch frequency
        .start()
        .awaitTermination();
```

---

<a name="conclusion"></a>
## Conclusion

This article covered an architecture for ingesting Avro data from a messaging system, converting it to Parquet, and saving it in Amazon S3. Using Spark Structured Streaming and Amazon S3 for storage, this pipeline is a robust solution for managing Avro-to-Parquet transformations in big data applications. Understanding the structural differences between Avro and Parquet, as well as the conversion process, enables data engineers to design efficient and scalable data pipelines.