O **catálogo do Databricks** é um recurso que organiza e gerencia os metadados relacionados a tabelas, bancos de dados e outros objetos de dados armazenados no ambiente Databricks. Ele é parte integrante da estrutura do Databricks e serve como um **sistema de gerenciamento de metadados** para facilitar o acesso, a governança e a descoberta de dados.

- [[#Storage credential|Storage credential]]
- [[#External Location|External Location]]
- [[#Manage Azure Databricks account and Metastore|Manage Azure Databricks account and Metastore]]
- [[#How to create a Unity Catalog|How to create a Unity Catalog]]
- [[#How to create catalogs, schema and tables]]
	- [[#Catalog]]
	- [[#Schema]]
	- [[#Tables]]
		- [[#Managed table]]
		- [[#External table]]
- [[#Views]]
## Storage credential

It is a managed entity within the Unity Catalog that centralizes the configuration of storage access credentials, in a secure and reusable way.

Storage credentials in Databricks are security features that provide a secure way to authenticate and authorize access to data storage systems (S3, Azure Blob Storage, etc.).

Represents an authentication and authorization mechanism for accessing data stored on your cloud. Each storage credential is subject to Unity Catalog access-control policies that control which users and groups can access the credential. If a user does not have access to a storage credential in Unity Catalog, the request fails and Unity Catalog does not attempt to authenticate to your cloud tenant on the user’s behalf.

To create storage credentials, you must be a Databricks account admin. The account admin who creates the storage credential can delegate ownership to another user or group to manage permissions on it. These credentials are used to ensure that only authorized users and clusters can access data, as defined by the organization's security policies.

Before create a storage credential, we have a process to follow that is: Create an **Access Connector for Azure Databricks** and grant **Storage Blob** to Access Connector for Azure Databricks.

A good video about this: https://youtu.be/kRfNXFh9T3U
## External location
An external location is an object that combines a cloud storage path with a storage credential that authorizes access to the cloud storage path. Each external location is subject to Unity Catalog access-control policies that control which users and groups can access the credential. If a user does not have access to an external location in Unity Catalog, the request fails and Unity Catalog does not attempt to authenticate to your cloud tenant on the user’s behalf.

Here, we can configure a different blob storage to interact with Databricks once you have created an external credential. To do this, we have to pass the storage credential and the URL.

---
## Manage Azure Databricks account and Metastore

Azure Databricks accounts are managed both through the Azure Databricks [account console](https://accounts.azuredatabricks.net/) and the Azure Portal. In the account console, account admins manage [Unity Catalog metastores](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/), [users and groups](https://learn.microsoft.com/en-us/azure/databricks/admin/users-groups/), and various account-level settings. [Microsoft Learn](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/)

The Azure Portal is where users with the Azure Contributor or Owner role can create workspaces, manage their subscriptions, and configure diagnostic logging. To enable access to the account console, you must establish your account’s first account admin following [Establish your first account admin](https://learn.microsoft.com/en-us/azure/databricks/admin/#establish-first-account-admin). In short, it is an administrative interface for managing your Databricks account at the organization level (not just a specific workspace).

The Unity Catalog Metastore organizes data like this:

Metastore (account level)
  ├── Catalogs
  │    ├──  (Schemas)
  │    │    ├──  Tables
  │    │    ├──  Views

Once inside the account console, we face that screen below and several options to manage, like workspaces, catalogs, etc. 

 - Create, delete and view all workspaces associated in the account.
 
![[Pasted image 20250131100741.png]]

- Create and manage the __metastore__ from Unity Catalog, set permissions for catalogs, schemas and tables. Enable centralized data governance for multiple workspaces.

![[Pasted image 20250131100842.png]]

- Manage users and groups like add, remove and assign roles. Set permissions to access workspaces and resources.

![[Pasted image 20250131110030.png]]
## How to create a Unity Catalog

The first step is to create a new *metastore*, that is the top-level container for Unity Catalog. Let's go to (accounts.azuredatabricks.net) once in the page, go to **create metastore**, and here we can set a name, region and an ADLS GEN 2 PATH (optional) this location will be used as a default place to storage managed tables across all catalogs in the metastore, if you do not pass this, Databricks will ask you for a external location when you try to create a new catalog. The last option, is the Access Connector ID, responsible to allow the connection with Azure Storage Account. After fill all the field, we have to attach this metastore with a workspace, this is very simple and intuitive! Completing all the steps, we will be able to create a new catalog inside Unity Catalog.

![[Pasted image 20250202173809.png]]
## How to create catalogs, schema and tables

Once we have a Unity Catalog configured, now we can create in order of the three-level hierarchy that is what Databricks uses to organize database objects. Here, I will show the SQL commands but we can do the same by user interface! Here a good explation: [link](https://www.youtube.com/watch?v=Ys9obOxJaOM&list=PL2IsFZBGM_IGiAvVZWAEKX8gg1ItnxEEb&index=11)
### Catalog
First of all, we need a catalog. The first option is with no path, then the catalog will be created under the standard path defined in Unity Catalog, ensure that you set up this, if no Databricks will ask you to specify a location. Also we can pass a external location after configure one. Below, I show both examples:
```sql
CREATE CATALOG IF NOT EXISTS SANDBOX_VINICIUS
COMMENT 'My Databricks catalog';


CREATE CATALOG IF NOT EXISTS SANDBOX_VINICIUS_EXT
MANAGED LOCATION 'abfss://adb@lakevinicius.dfs.core.windows.net/sandbox_ext';

-- BONUS, HOW TO DROP CATALOG? 
DROP CATALOG MY_CATALOG CASCADE
-- Cascade statement will drop all objects under the catalog. 
```
Once created, we are able to check the catalog with the command:
```sql
DESCRIBE CATALOG EXTENDED catalog_name
```

The result:

![[Pasted image 20250205075915.png]]
Also we can run the command below to view all catalogs available, and personalize the query to look for a specific catalog, very helpful in case you have a bunch! 
```sql
-- In this example, I am looking for all catalogs that starts with sand, no matter whats comes next
SHOW CATALOGS LIKE 'sand*'
```
_Tip: We can also use the like operator for tables and schemas._
### Schema
The next step is to create the schema. It is so easy to create as a catalog, and also we can create a schema by default or with an external location. However, keep in mind, the things here works by hierarchy, what it means? Before I created the catalog _SANDBOX_VINICIUS_ without an external location, now if a create a schema under this catalog using another location, the content under the schema will be stored on the location defined on my schema, maybe it is confusing at first. A schema belongs to a catalog, and tables belong to a schema.
```sql
USE CATALOG sandbox_vinicius;
CREATE SCHEMA IF NOT EXISTS lab_schema
-- WITH MANAGED LOCATION
MANAGED LOCATION 'abfss://adb@lakevinicius.dfs.core.windows.net/schemas'
COMMENT 'My schema'
```

After done, let's use the describe command: 

![[Pasted image 20250205081459.png]]
So, early I created this catalog in the standard location configured in Unity Catalog, now under that I created a schema declaring to use a external location, when I create a table within this schema, it will be stored there, unless I set up another location to my table. 
### Tables
TERMINAR
#### Managed table

Here, Databricks is the one who manages everything about the table, both metadata and the physical data itself. Files are automatically stored in the default directory configured in Databricks (usually where it is associated, S3, Azure Data Lake, or GCS). If you delete the tables, the underlying data there in the storage will also be deleted!

Should be used when Databricks is responsible for managing the entire data lifecycle. Scenarios in which we want simplicity in management, without worrying about manually organizing files. We can create files of different types such as Parquet, CSV, JSON... But of course, this will lose the functionality of the Delta file!
#### Creating a managed table:
```sql
CREATE TABLE IF NOT EXISTS sandbox_vinicius.lab_schema.teams (
  id INT,
  team STRING,
  city STRING
);

INSERT INTO sandbox_vinicius.lab_schema.teams VALUES 
(1, 'Real Madrid', 'Madrid'), (2, 'Cruzeiro', 'BH')
```

PySpark:
```python
df.write.format("delta").saveAsTable("catalog.schema.table")
```
After created:

![[Pasted image 20250205145817.png]]
By UI:

![[Pasted image 20250205145937.png]]

---
#### External table

In Databricks, an External Table is a table whose schema and metadata are managed by Unity Catalog or Hive Metastore, but the physical data remains stored outside the control managed by Databricks, typically in a Data Lake as: Azure Data Lake Storage Gen2 (ADLS Gen2) or Amazon S3.

https://learn.microsoft.com/en-us/azure/databricks/tables/external
#### Creating a external table:
```sql
CREATE TABLE IF NOT EXISTS sandbox_vinicius.lab_schema.teams_ext (
  id INT,
  team STRING,
  city STRING
)
LOCATION 'abfss://adb@lakevinicius.dfs.core.windows.net/adb/tables';

INSERT INTO sandbox_vinicius.lab_schema.teams_ext VALUES
(1, 'Barcelona', 'Barcelona'), (2, 'PSG', 'Paris')
```
After created we can see the **type** and **location** using DESCRIBE: 

![[Pasted image 20250205153529.png]]
Looking by UI:

![[Pasted image 20250205153629.png]]

So, after creating two tables, now I will drop them to look about the command UNDROP.

![[Pasted image 20250205151741.png]]

### Plus:

How to check if a table exists:
```python
## Python
spark.catalog.tableExists("catalog.schema.table")
```
### UNDROP
What is this? UNDROP is a feature that allow us to recover dropped tables within 7 days. When we drop a table with Unity Catalog, the data files immediately does not get removed, it will be removed within seven days. If you have dropped a table by mistake, you can go ahead and undrop it. Following, I explain how to do that. More about this here: [Link](https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/sql-ref-syntax-ddl-undrop-table)

*Before, I just checked at blob storage, and my data still is there! For both tables, managed and external, you can back above and check the table ID with the path!*

![[Pasted image 20250205153749.png]]
![[Pasted image 20250205154134.png]]
Back on Databricks, we have to query:

```sql
SHOW TABLES DROPPED IN schema_name
```

Here, we see a lot of informations about tables that can be revovered, ok, it is cool! But how to recover the table?

![[Pasted image 20250205154744.png]]
TERMINAR

## Views
In Databricks, views are logical representations of data that can be queried like regular tables. They are read-only objects composed of one or more tables and views in a metastore, allowing users to structure data efficiently and simplify SQL queries.

- Views can be temporary or permanent and are useful for transforming, filtering, and aggregating data.
### Temporary view

This type of views are only visible to the session that created them and are discarded when the session ends.

```sql
CREATE TEMPORARY VIEW teams_madrid_temp_view -- View name
AS
SELECT * FROM sandbox_vinicius.lab_schema.teams_ext 
WHERE CITY = 'Madrid';
```
We can check the view running SQL or PySpark:

![[Pasted image 20250207105746.png]]
![[Pasted image 20250207105832.png]]
### Permanent view
A permanent view is a **logical object** that allows you to create a reusable query as a virtual table. Unlike temporary views, permanent views are stored in **Unity Catalog** or a specified **schema** and are accessible by multiple users with appropriate permissions. So, to create this kind of view you have to set a catalog and schema to storage that.

_Tip: Once you have a view, in the future if you want to modify query parameters, just run the new using **CREATE OR REPLACE VIEW**_

![[Pasted image 20250207111503.png]]





### Plus:

 - **How to check if a table exists using Python:**
```python
spark.catalog.tableExists("catalog.schema.table")
```
- Ways to copy a table

There are different ways to create copies of tables for various use cases, such as **CTAS (CREATE TABLE AS SELECT)**, **Shallow Clone**, and **Deep Clone**. Each method serves different purposes depending on you need a simple copy, a metadata reference, or a full independent dataset.

- **CTAS**
Is used to create a **new table with copied data** from an existing table. The new table is independent of the original and does not inherit properties like Delta history.
```sql
CREATE TABLE lab_schema.teams_ext2
AS
SELECT * FROM sandbox_vinicius.lab_schema.teams_ext;
-- The table will be created in a different location that the original
```

![[Pasted image 20250207114519.png]]
Note: The table starts from version 0! You can see the history and the operation:

![[Pasted image 20250207114819.png]]

- **DEEP CLONE**
 Copies all data and metadata from the source to the target. This includes the data, schema, partitioning information, invariants, and nullability. Deep clones are independent of the source, but they are costly to make. In case of delta tables, deep clone is a better option than CTAS.

```sql
CREATE TABLE lab_schema.teams_ext_deep
DEEP CLONE lab_schema.teams_ext
```

![[Pasted image 20250207115400.png]]
Looking at history:

![[Pasted image 20250207115532.png]]
- **SHALLOW CLONE**
Copies only the metadata from the source to the target. This includes the schema, partitioning information, invariants, and nullability. Shallow clones are cheaper to create, but they share references to nested objects.

```sql
CREATE TABLE lab_schema.teams_shallow
SHALLOW CLONE lab_schema_managed.teams;
```

Note: Shallow clone does not work with external tables!
![[Pasted image 20250207120021.png]]
![[Pasted image 20250207120539.png]]
Looking at history:

![[Pasted image 20250207120648.png]]

## UPSERT into a Delta table using MERGE
TERMINAR
