---
title: Deep Dive into Avro and Parquet Formats
date: 2024-10-28 16:10:12
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
- [Understanding Avro Format](#understanding-avro-format)
  - [Avro Schema](#avro-schema)
  - [Avro Schema in Depth](#avro-schema-in-depth)
    - [1. **Schema Basics**](#1-schema-basics)
    - [2. **Primitive Data Types**](#2-primitive-data-types)
    - [3. **Complex Data Types**](#3-complex-data-types)
    - [4. **Union Types**](#4-union-types)
    - [5. **Default Values**](#5-default-values)
    - [6. **Schema Evolution**](#6-schema-evolution)
    - [7. **Named Types and Namespaces**](#7-named-types-and-namespaces)
    - [8. **Schema Example with Namespaces**](#8-schema-example-with-namespaces)
  - [Schema Registry](#schema-registry)
  - [Storage Architecture for Avro in HDFS/S3](#storage-architecture-for-avro-in-hdfss3)
  - [Sample Code in Spark](#sample-code-in-spark)
- [Exploring Parquet Format](#exploring-parquet-format)
  - [Parquet Schema and Compression](#parquet-schema-and-compression)
  - [Parquet Schema](#parquet-schema)
    - [1. **Schema Hierarchy and Structure**](#1-schema-hierarchy-and-structure)
    - [2. **Column Definition: Required, Optional, Repeated**](#2-column-definition-required-optional-repeated)
    - [3. **Definition and Repetition Levels**](#3-definition-and-repetition-levels)
      - [Example Parquet Schema](#example-parquet-schema)
      - [Parquet Structure: Definition and Repetition Levels](#parquet-structure-definition-and-repetition-levels)
      - [1. Field: `name`](#1-field-name)
      - [2. Field: `contacts`](#2-field-contacts)
      - [3. Fields Inside `contacts`: `type`, `address`, `number`](#3-fields-inside-contacts-type-address-number)
      - [4. Field: `addresses`](#4-field-addresses)
      - [5. Fields Inside `addresses`: `street`, `city`](#5-fields-inside-addresses-street-city)
      - [Example Encoding with Levels](#example-encoding-with-levels)
    - [4. **Logical Types and Annotations**](#4-logical-types-and-annotations)
    - [5. **Column Index and Row Group Metadata**](#5-column-index-and-row-group-metadata)
    - [Example of a Parquet Schema in JSON Representation](#example-of-a-parquet-schema-in-json-representation)
  - [Storage Architecture for Parquet in HDFS/S3](#storage-architecture-for-parquet-in-hdfss3)
  - [Sample Code in Spark](#sample-code-in-spark-1)
- [Conclusion](#conclusion)

---

<a name="introduction"></a>
## Introduction

As data systems grow in complexity and scale, choosing efficient data serialization formats becomes crucial. This article provides a deep dive into **Apache Avro** and **Apache Parquet**, two popular formats in the **Big Data** ecosystem. Both formats have distinct characteristics, making them suitable for different types of data and use cases.

---

<a name="understanding-avro-format"></a>
## Understanding Avro Format

**Apache Avro** is a data serialization format that facilitates the encoding of complex data structures into a compact, binary format, making it highly efficient for streaming and storage.

<a name="avro-schema"></a>
### Avro Schema

The Avro schema describes the structure of data stored in Avro format and is defined using **JSON**. Avro schemas allow the specification of nested fields, and each schema comprises two parts:
- **Primitive types**: `null`, `boolean`, `int`, `long`, `float`, `double`, `bytes`, and `string`.
- **Complex types**: `record`, `enum`, `array`, `map`, `union`, and `fixed`.

**Example Avro Schema**:
```json
{
  "type": "record",
  "name": "Employee",
  "fields": [
    {"name": "id", "type": "int"},
    {"name": "name", "type": "string"},
    {"name": "email", "type": ["null", "string"], "default": null},
    {"name": "salary", "type": "double"}
  ]
}
```

In this schema:
- The `Employee` record has fields `id`, `name`, `email`, and `salary`.
- The `email` field is optional, allowing `null` values.

<a name="avro-schema-in-depth"></a>
### Avro Schema in Depth
Apache Avro is a data serialization framework with a focus on compact storage, schema evolution, and interoperability, especially popular in data pipelines with tools like Apache Kafka, Hadoop, and Spark. Avro schema is a JSON-based definition that describes the structure, types, and constraints of the data. This schema allows for efficient serialization and deserialization, making it versatile for data storage and streaming applications.

Avro’s schema format is highly adaptable, supporting both simple and complex data structures and allowing schema evolution without disrupting data compatibility. This makes Avro particularly suited for big data pipelines and environments where schemas may need frequent updates. Its JSON-based structure is easy to read and interpret while enabling efficient storage and retrieval in binary format, ideal for high-performance data processing.

Here’s an in-depth look at Avro schemas:

<a name="1-schema-basics"></a>
#### 1. **Schema Basics**
   Avro schemas are written in JSON and define the data types and structure. A schema specifies:
   - **Type**: The data type, like `string`, `int`, `float`, or complex types like `record` and `array`.
   - **Name**: The name of the field (only required for named types like `record`).
   - **Namespace**: Optional, used to provide a unique context for named types, helpful in managing type names in larger systems.
   - **Fields**: Defines each field within complex types, like `record`.

   Example of a simple Avro schema:
   ```json
   {
     "type": "record",
     "name": "User",
     "fields": [
       {"name": "id", "type": "int"},
       {"name": "name", "type": "string"},
       {"name": "email", "type": "string"}
     ]
   }
   ```
   This schema defines a `User` record with three fields: `id`, `name`, and `email`.

<a name="2-primitive-data-types"></a>
#### 2. **Primitive Data Types**
   Avro supports several primitive data types:
   - **Null**: Represents a null value.
   - **Boolean**: Represents `true` or `false`.
   - **Int**: 32-bit signed integer.
   - **Long**: 64-bit signed integer.
   - **Float**: 32-bit IEEE floating-point number.
   - **Double**: 64-bit IEEE floating-point number.
   - **Bytes**: Sequence of bytes, useful for binary data.
   - **String**: Unicode character sequence.

<a name="3-complex-data-types"></a>
#### 3. **Complex Data Types**
   Avro supports complex data types to handle more advanced data structures:
   - **Record**: Similar to a structured data object or table row; contains fields, each with its own name and type.
   - **Enum**: Defines a list of allowed values.
   - **Array**: An ordered collection of values of a specified type.
   - **Map**: A collection of key-value pairs with `string` keys and values of a specified type.
   - **Union**: Allows fields to have multiple types, which supports optional fields and complex structures.
   - **Fixed**: Represents a fixed-size binary value, useful for things like fixed-length IDs.

   Example of complex types:
   ```json
   {
     "type": "record",
     "name": "Person",
     "fields": [
       {"name": "name", "type": "string"},
       {"name": "age", "type": ["null", "int"], "default": null},
       {"name": "emails", "type": {"type": "array", "items": "string"}},
       {"name": "address", "type": {
         "type": "record",
         "name": "Address",
         "fields": [
           {"name": "street", "type": "string"},
           {"name": "city", "type": "string"}
         ]
       }}
     ]
   }
   ```
   This schema defines a `Person` record with an optional `age`, a list of `emails`, and a nested `Address` record.

<a name="4-union-types"></a>
#### 4. **Union Types**
   Unions in Avro schemas allow a field to hold one of multiple types, helping to represent optional fields:
   - A union is represented by an array of types, such as `["null", "int"]` for a nullable integer.
   - The first type in the union is considered the default.

   Example:
   ```json
   {"name": "birthdate", "type": ["null", "string"], "default": null}
   ```
   Here, `birthdate` is either `null` or a `string`, allowing it to be optional.

<a name="5-default-values"></a>
#### 5. **Default Values**
   - Fields can have default values if they are missing in the data, which is particularly useful when evolving schemas.
   - Default values must match the field type and are defined within the schema.

   Example:
   ```json
   {"name": "isActive", "type": "boolean", "default": true}
   ```

<a name="6-schema-evolution"></a>
#### 6. **Schema Evolution**
   Avro is designed to handle schema evolution, meaning new schemas can be applied to existing data files without breaking compatibility:
   - **Backward Compatibility**: New schema can read old data (e.g., adding new fields with default values).
   - **Forward Compatibility**: Old schema can read new data (e.g., adding optional fields).
   - **Full Compatibility**: Both backward and forward compatible.

   Schema evolution rules include:
   - Adding a new field with a default value is backward-compatible.
   - Removing a field is backward-compatible if it is not required by consumers.
   - Changing a field type requires careful consideration to ensure compatibility.

<a name="7-named-types-and-namespaces"></a>
#### 7. **Named Types and Namespaces**
   In Avro, named types like `record` and `enum` require unique names, often scoped by namespaces:
   - **Name**: The unique identifier for a type.
   - **Namespace**: Provides context to prevent naming conflicts, similar to a package in Java or a module in Python.

<a name="8-schema-example-with-namespaces"></a>
#### 8. **Schema Example with Namespaces**
   Example using namespace for managing complex data:
   ```json
   {
     "namespace": "com.example",
     "type": "record",
     "name": "Customer",
     "fields": [
       {"name": "id", "type": "int"},
       {"name": "contact", "type": {
         "type": "record",
         "name": "ContactInfo",
         "fields": [
           {"name": "email", "type": "string"},
           {"name": "phone", "type": ["null", "string"], "default": null}
         ]
       }}
     ]
   }
   ```

<a name="schema-registry"></a>
### Schema Registry

**Schema Registry** is a crucial component that allows the storage and retrieval of schemas used by producers and consumers of data. It ensures that producers and consumers agree on the data structure, supporting **schema evolution** and **compatibility checks**.

**Diagram: Schema Registry Workflow**
```
        +-----------+             +--------------------+
        | Producer  |   Write     |  Schema Registry   |
        |           | ----------> |                    |
        +-----------+             +--------------------+
            |
            |
            |    Write
            v
        +-----------+                  +-----------------+
        |  Data     |    Read/Write    |    Consumer     |
        |  Storage  | <--------------> |                 |
        +-----------+                  +-----------------+
```

- **Producers** register the schema before sending data to storage.
- **Consumers** fetch the schema from the registry to deserialize and understand the data.

<a name="storage-architecture-for-avro-in-hdfs-s3"></a>
### Storage Architecture for Avro in HDFS/S3

When storing Avro data in **HDFS** or **S3**, Avro files contain both the schema and data, allowing each file to be fully self-contained and readable independently.

**Diagram: Avro Storage in HDFS/S3**
```
        +------------------------+
        |   HDFS or S3 Storage   |
        +------------------------+
                  |
        +------------------------+
        |       Avro File        |
        |      + Schema          |
        +------------------------+
                 |
        +------------------------+
        |       Blocks           |
        +------------------------+
```

- **Each Avro file** contains schema and data blocks, making it easily portable across storage platforms.
- **Data Access**: Avro files can be read without external dependencies, as each file includes the required schema.

<a name="sample-code-in-spark-avro"></a>
### Sample Code in Spark

Below is a Spark example that reads data from an Avro file, processes it, and writes it back to HDFS as Avro format.

```scala
import org.apache.spark.sql.{DataFrame, SparkSession}
import org.apache.spark.sql.avro._

// Initialize Spark session
val spark = SparkSession.builder
    .appName("AvroExample")
    .getOrCreate()

// Define the Avro schema as JSON
val avroSchema = """
{
  "type": "record",
  "name": "Employee",
  "fields": [
    {"name": "id", "type": "int"},
    {"name": "name", "type": "string"},
    {"name": "email", "type": ["null", "string"], "default": null},
    {"name": "salary", "type": "double"}
  ]
}
"""

// Read Avro file from HDFS
val df: DataFrame = spark.read
    .format("avro")
    .load("hdfs://path/to/input/avro/file")

// Process the DataFrame (example: filter employees with salary > 5000)
val filteredDf = df.filter("salary > 5000")

// Write back to HDFS in Avro format
filteredDf.write
    .format("avro")
    .option("avroSchema", avroSchema)
    .save("hdfs://path/to/output/avro/file")
```

---

<a name="exploring-parquet-format"></a>
## Exploring Parquet Format

**Apache Parquet** is a columnar storage format optimized for complex nested data structures, making it an ideal choice for **big data processing**.

<a name="parquet-schema-and-compression"></a>
### Parquet Schema and Compression

Parquet organizes data in columns, allowing for efficient compression and encoding. This format is particularly beneficial for queries that scan specific columns.

**Compression Techniques**:
- **Dictionary Encoding**: Reduces data size by replacing duplicate values with codes.
- **Bit Packing**: Optimizes storage by packing multiple values into smaller data units.

**Diagram: Parquet File Structure**
```
+------------------------------+
|           Parquet            |
+------------------------------+
|      Row Group 1             |
|    +--------------------+    |
|    | Column 1           |    |
|    | Column 2           |    |
|    | ...                |    |
|    +--------------------+    |
|                              |
|      Row Group 2             |
|    +--------------------+    |
|    | Column 1           |    |
|    | Column 2           |    |
|    | ...                |    |
|    +--------------------+    |
+------------------------------+
```

- Each **Row Group** contains columns stored together, optimizing compression and query performance.
- **Columnar storage** allows quick access to specific fields, reducing read and write times.

<a name="parquet-schema"></a>
### Parquet Schema
Parquet’s schema defines the structure, types, and layout of the data stored in the file, making it highly efficient for analytics and queries. This schema is stored in the file’s metadata and allows query engines to interpret the data without needing to read the entire file, which speeds up operations significantly.

The Parquet schema plays a crucial role in its columnar storage efficiency, managing data sparsity, and enabling rich, nested data structures with low storage overhead. It facilitates schema evolution, where additional columns or fields can be added without rewriting the entire file, and supports advanced filtering and optimizations for large data analytics tasks.

Here’s an overview of the main components of the Parquet schema:

<a name="1-schema-hierarchy-and-structure"></a>
#### 1. **Schema Hierarchy and Structure**
   Parquet schema is organized hierarchically and supports complex nested structures:
   - **Message Type**: This is the root element in a Parquet schema, defining the data structure at the highest level.
   - **Fields**: Each field in the schema has a name, data type, and potentially nested subfields if the data structure is complex. Parquet supports:
     - **Primitive Types**: such as `int32`, `int64`, `boolean`, `float`, `double`, and `binary`.
     - **Nested Types**: Parquet can store complex structures like lists, maps, and structs, which are stored as nested types in the schema.

   For example, a Parquet schema could look like:
   ```plaintext
   message schema {
       required int32 id;
       optional group address {
           required binary street (UTF8);
           optional int32 zip_code;
       }
   }
   ```
   This schema describes a record with an `id` and an optional nested `address` group, which contains a `street` and an optional `zip_code`.

<a name="2-column-definition-required-optional-repeated"></a>
#### 2. **Column Definition: Required, Optional, Repeated**
   Each field in the Parquet schema has a **repetition level**:
   - **Required**: Every row must contain a value for this field.
   - **Optional**: A field that can be present or absent in each row.
   - **Repeated**: Used to model lists or repeated elements, allowing multiple values for the field in a row.

<a name="3-definition-and-repetition-levels"></a>
#### 3. **Definition and Repetition Levels**
  Parquet’s schema model is hierarchical and optimized for efficient storage and retrieval in columnar format. It uses **Definition Levels** and **Repetition Levels** to handle nullability, optional fields, and nested data structures. Let’s walk through an example to show how these levels work.

   To manage complex data structures and sparsity, Parquet uses **definition levels** and **repetition levels**:
   - **Definition Levels** track whether each level in the hierarchy has data or is `null`, it tells us how deep in the structure a field is defined.
   - **Repetition Levels** track whether an element is the first instance or a repeated instance in nested lists or repeated fields, it tells us if the field is part of a repeated structure and, if so, how deep the repeat goes.

   For instance, if a row has a list of addresses, the repetition levels can indicate the position of each address within the list, while definition levels show whether each element is present or missing.


##### Example Parquet Schema

Consider the following JSON structure, which could be a nested data schema for a Parquet file:
```json
{
  "name": "Alice",
  "contacts": [
    {
      "type": "email",
      "address": "alice@example.com"
    },
    {
      "type": "phone",
      "number": null
    }
  ],
  "addresses": [
    {
      "street": "123 Main St",
      "city": "Wonderland"
    }
  ]
}
```

Here’s the corresponding Parquet schema:
```plaintext
message Person {
  required binary name (UTF8);
  repeated group contacts {
    required binary type (UTF8);
    optional binary address (UTF8);
    optional binary number (UTF8);
  }
  repeated group addresses {
    required binary street (UTF8);
    required binary city (UTF8);
  }
}
```

##### Parquet Structure: Definition and Repetition Levels

1. **Definition Level**: Indicates the presence or absence of values at each level in a schema. This level is used to define nullability and optional fields.
2. **Repetition Level**: Indicates if a field is part of a repeated structure (e.g., a list or an array). This is useful for nested or repeated structures, telling the reader if a given level repeats within the hierarchy.

Let’s break down how these levels work for each field in the example above:

##### 1. Field: `name`
   - **Definition Level**: 1 (required field, so it's always defined)
   - **Repetition Level**: 0 (not repeated)

##### 2. Field: `contacts`
   - **Definition Level**: 1 to 2, depending on the presence of `address` and `number` fields.
     - If both `address` and `number` are `null`, the definition level is lower.
   - **Repetition Level**: 1 (this field is repeated since `contacts` is a list)

##### 3. Fields Inside `contacts`: `type`, `address`, `number`
   - **type**
     - **Definition Level**: 2 (required field within the repeated group)
     - **Repetition Level**: 1
   - **address** and **number**
     - **Definition Level**: 2 or 3 (optional fields)
       - A definition level of 2 means the `contacts` object is present but `address` or `number` is `null`.
       - A definition level of 3 indicates both fields are filled.
     - **Repetition Level**: 1 (repeated inside `contacts`)

##### 4. Field: `addresses`
   - **Definition Level**: 1 to 2
   - **Repetition Level**: 1 (repeated field)

##### 5. Fields Inside `addresses`: `street`, `city`
   - Both `street` and `city` are required within the repeated `addresses` group.
   - **Definition Level**: 2
   - **Repetition Level**: 1 (both are repeated with each `addresses` group entry)

##### Example Encoding with Levels

Assuming data with these values:
```json
{
  "name": "Alice",
  "contacts": [
    {
      "type": "email",
      "address": "alice@example.com",
      "number": null
    },
    {
      "type": "phone",
      "number": "123-456-7890"
    }
  ],
  "addresses": [
    {
      "street": "123 Main St",
      "city": "Wonderland"
    }
  ]
}
```

The encoded levels might look like this:

| Field        | Value            | Definition Level | Repetition Level |
|--------------|------------------|------------------|------------------|
| name         | "Alice"          | 1                | 0                |
| contacts[0].type | "email"         | 2                | 1                |
| contacts[0].address | "alice@example.com" | 3              | 1                |
| contacts[0].number | null             | 2                | 1                |
| contacts[1].type | "phone"         | 2                | 1                |
| contacts[1].address | null             | 2                | 1                |
| contacts[1].number | "123-456-7890" | 3              | 1                |
| addresses[0].street | "123 Main St" | 2              | 1                |
| addresses[0].city | "Wonderland"   | 2              | 1                |


These levels help Parquet efficiently handle complex nested and repeated data structures, enabling better data compression and faster querying by only storing relevant levels and values for each entry.


<a name="4-logical-types-and-annotations"></a>
#### 4. **Logical Types and Annotations**
   Parquet supports logical types that provide additional context for primitive types:
   - **Logical Types**: These are type annotations like `UTF8` for text strings, `DATE` for dates, and `DECIMAL` for high-precision decimals.
   - Logical types help map data from systems that use richer data types (like SQL databases) to Parquet’s primitive types.

   Example:
   ```plaintext
   optional int32 birthdate (DATE);
   ```

<a name="5-column-index-and-row-group-metadata"></a>
#### 5. **Column Index and Row Group Metadata**
   - **Column Index**: Parquet files include metadata about each column, such as min/max values, null counts, and distinct counts. This allows query engines to skip entire blocks if they don't match the filter criteria.
   - **Row Group Metadata**: Parquet stores data in row groups (blocks of rows), each containing column data for that group. Each row group stores metadata like the number of rows and data size, which helps with efficient reads.

<a name="example-of-a-parquet-schema-in-json-representation"></a>
#### Example of a Parquet Schema in JSON Representation
A Parquet schema can also be represented in JSON format to show the hierarchy and structure clearly. For instance:
```json
{
  "name": "schema",
  "type": "message",
  "fields": [
    {
      "name": "id",
      "type": "INT32",
      "repetition_type": "REQUIRED"
    },
    {
      "name": "address",
      "type": "group",
      "repetition_type": "OPTIONAL",
      "fields": [
        {
          "name": "street",
          "type": "BINARY",
          "logicalType": "UTF8",
          "repetition_type": "REQUIRED"
        },
        {
          "name": "zip_code",
          "type": "INT32",
          "repetition_type": "OPTIONAL"
        }
      ]
    }
  ]
}
```

In this JSON schema:
- The `id` field is required.
- The `address` field is optional and has two nested fields: a required `street` and an optional `zip_code`.

<a name="storage-architecture-for-parquet-in-hdfs-s3"></a>
### Storage Architecture for Parquet in HDFS/S3

In HDFS/S3 storage, Parquet files are organized in row groups with each column stored independently within those groups.

**Diagram: Parquet Storage in HDFS/S3**
```
        +------------------------+
        |   HDFS or S3 Storage   |
        +------------------------+
                  |
        +------------------------+
        |     Parquet File       |
        +------------------------+
                 |
        +------------------------+
        |   Row Groups           |
        +------------------------+
                 |
        +------------------------+
        | Columns within Groups  |
        +------------------------+
```

- **Row Groups**: Partitioned sections that allow for distributed reads.
- **Columns**: Stored separately within each row group for efficient access.

<a name="sample-code-in-spark-parquet"></a>
### Sample Code in Spark

Below is a Spark example that reads data from a Parquet file, performs transformations, and writes it back to HDFS as Parquet format.

```scala
import org.apache.spark.sql.{DataFrame, SparkSession}

// Initialize Spark session
val spark = SparkSession.builder
    .appName("ParquetExample")
    .getOrCreate()

// Read Parquet file from HDFS
val df: DataFrame = spark.read
    .parquet("hdfs://path/to/input/parquet/file")

// Process the DataFrame (example: select employees with salary > 5000)
val filteredDf = df.filter("salary > 5000")

// Write back to HDFS in Parquet format
filteredDf.write
    .parquet("hdfs://path/to/output/parquet/file")
```

---

<a name="conclusion"></a>
## Conclusion

This article provided an in-depth look at **Avro** and **Parquet** formats, exploring their unique characteristics, schema management, and storage architectures. With examples in **Spark**, we demonstrated practical implementations in a **real-world** production environment, highlighting the versatility of both formats for big data solutions.
