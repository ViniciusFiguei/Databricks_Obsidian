Before we deep in the content, let's see an explanation about what is Delta and how it works behind. Basically a Delta Table is composed by parquet files, working together a delta log that captures and save all the transactions we do.

![[Pasted image 20250520175359.png]]
https://delta-io.github.io/delta-rs/how-delta-lake-works/architecture-of-delta-table/

To illustrate, below I have run `%fs ls` over a Delta file saved in volumes, and as we can see there was 19 partitions with parquet files under the delta. (We see 21 rows but we have the delta log folder and a archive called SUCCESS).

![[Pasted image 20250521090807.png]]

Inside the `_delta_log` folder of a Delta Table, there are three main types of files:
#### 1. **JSON Files**

- Each JSON file represents a **transaction** performed on the Delta Table.
- They are **versioned** and named incrementally, such as:
    ```
    00000000000000000000.json
    00000000000000000001.json
    ```
- These files contain detailed records of actions like:
    
    - Add/remove files
    - Metadata updates
    - Transaction-level information

>  These are the core files used to track the full history of the table.
### 2. **CRC Files**

- CRC files are **checksum files** used to:
    - Ensure data **integrity**
    - Detect possible **corruption** in the corresponding JSON files

>  They play a key role in validating the consistency of transaction logs.
### 3. **Checkpoint Files (**`**.checkpoint.parquet**`**)**

- Created **every N transactions** (by default, every 10 JSON files).
- Stored in **Parquet format** for efficient reading.
#### ✅ Purposes of checkpoint files:

- **Boost performance** when reading the Delta table state (no need to process all JSON files from the beginning)
- **Serve as a starting point** to reconstruct the latest state of the table

---
### Benefits of Using Delta Tables

Delta Lake brings ACID transactions and reliability to big data. Here are the key advantages:
#### 1. **ACID Transactions**
- Ensures **atomicity**, **consistency**, **isolation**, and **durability** for all data operations.
- Avoids issues from partial writes or concurrent updates.
#### 2. **Schema Evolution**
- Supports automatic schema updates using options like `mergeSchema`.
- Flexible for evolving datasets.
#### 3. **Time Travel**
- Query previous versions of data with `VERSION AS OF` or `TIMESTAMP AS OF`.
- Enables rollback and auditing.
#### 4. **Data Lineage and Auditability**
- Maintains a full history of changes (with metadata).
- Supports GDPR compliance and debugging.
#### 5. **Scalable Metadata Handling**
- Handles petabyte-scale tables efficiently.
- No performance degradation with many partitions/files.
#### 6. **Efficient Upserts and Deletes**
- Supports `MERGE INTO`, `UPDATE`, `DELETE`, and `INSERT` natively.
- Great for CDC and streaming use cases.
#### 7. **Streaming and Batch Unification**
- Seamlessly supports both streaming and batch reads/writes on the same table.
#### 8. **Performance Optimization**
- Leverages **data skipping**, **Z-ordering**, and **file compaction (OPTIMIZE)** for faster queries.
---
## Some topics covered here about Delta Tables:

- [[#Vacuum]]
- [[#Time travel]]
- [[#Convert Parquet to Delta]]
- [[#Delta Table schema validation]]
- [[#Schema evolution]]
- [[#REORG and DROP COLUMN]]
- [REGORF](#REORG-and-DROP-COLUMN)
## Vacuum

No **Databricks**, [VACUUM](https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/delta-vacuum) é um comando usado para limpar arquivos obsoletos de tabelas **Delta Lake**, liberando espaço em disco.

Quando uma tabela Delta é modificada (inserções, atualizações ou deleções), os arquivos antigos não são imediatamente removidos. Isso acontece porque o Delta Lake mantém um histórico de versões da tabela para suporte a **Time Travel** e recuperação de dados.

O comando `VACUUM` remove arquivos de versões antigas que não são mais necessários, reduzindo o armazenamento usado.

- Sintaxe - Using `DRY RUN` you will only see the files that will be dropped, if you are sure remove the DRY RUN like the next command.

```sql
VACUUM catalog.schema.table RETAIN 0 HOURS DRY RUN;
```

- **`RETAIN`** → Define quantas horas de histórico manter antes de apagar os arquivos (mínimo: **168 horas = 7 dias**).
- **`DRY RUN`** → Exibe os arquivos que seriam apagados, sem removê-los.

Exemplo:
 
Remover arquivos antigos de uma tabela Delta, mantendo apenas os últimos 7 dias:

```sql
VACUUM minha_tabela RETAIN 168;
```

Ou verificar antes de apagar:

```sql
VACUUM minha_tabela RETAIN 168 DRY RUN;
```

**Warning** ⚠️
- O **Delta Lake** exige um mínimo de **7 dias** para retenção por padrão. Para reduzir esse valor, é necessário configurar `spark.databricks.delta.retentionDurationCheck.enabled` como `false`.

```sql
SET spark.databricks.delta.retentionDurationCheck.enabled = false
```

![[Pasted image 20250521105307.png]]
- Se executar `VACUUM` com um tempo de retenção muito baixo, pode impedir o **Time Travel** e comprometer a recuperação de dados antigos.

## Time travel

For all operations that we do in Delta Tables, they will maintain versions and all the transaction information. With time travel we can travel across the time time and restore or select old versions.

- How to see the versions? Using **DESCRIBE HISTORY**

It is a command in Databricks is used to retrieve provenance information for each write operation performed on a Delta table. This includes details such as the operation type, the user who performed the operation, and the timestamp of the operation. The table history is retained for 30 days by default.

After added some data into my table, I ran the query and this is the result:

![[Pasted image 20250207104313.png]]
After identifying the version you want look for, you can run like:
```sql
SELECT * FROM catalog.schema.table@V0;
-- Where V0 is the version, you just have to add @V+ the version number!
```
![[Pasted image 20250207104830.png]]



- Using PySpark (Table or Path) with **timestamp**

![[Pasted image 20250211215955.png]]
- Also PySpark using **version**

![[Pasted image 20250211220134.png]]

- Another way using SQL with **version**

```
-- SQL SYNTAX
SELECT * FROM catalog.schema.table VERSION AS OF 1
```

![[Pasted image 20250211221031.png]]

- Using SQL with **timestamp**

```
-- SQL SYNTAX 
SELECT * FROM catalog.schema.table TIMESTAMP AS OF "YOUR_TIME_STAMP_DATE"
```

![[Pasted image 20250211221006.png]]

Also, SQL using path with timestamp, also can be used version:

![[Pasted image 20250211220848.png]]

Now, knowing the version, we can restore a version doing this:
```sql
-- SINTAX
RESTORE TABLE catalog.schema.table TO VERSION AS OF 5;
```

![[Pasted image 20250213112211.png]]
After run the command above, we use **DESCRIBE HISTORY** to see the status. As you can see, the command restore does not delete the old versions, it just create a new version with operation called *RESTORE*. This means that you still are able to query old versions!  

![[Pasted image 20250213112745.png]]
Using Spark:

```python
spark.sql("RESTORE TABLE catalog.schema.table TO VERSION AS OF 5")
```
__Note__: We can not restore using dataframe API, but we can use the lib *DeltaTable* like that:
```python
from delta import DeltaTable

people_dt = DeltaTable.forName(spark, "dev.demo_db.people")

people_dt.restoreToVersion(1)

people_dt.restoreToTimestamp('YOUR_DATE_HERE')
```
## Convert Parquet to Delta
In Databricks, if we want to convert a Parquet file to Delta, we don’t need to read and write the file again. We can simply convert it, because this is a feature of Delta Lake — a Delta table is backed by Parquet files under the hood.

How to do that? First, take a look with the parquet table stored in Volumes:

![[Pasted image 20250520094707.png]]
To convert:
![[Pasted image 20250520094843.png]]
```sql
%sql
CONVERT TO DELTA parquet.`/Volumes/dev/demo_db/files/fire_calls_tbl`
PARTITIONED BY (Year INT) -- If the table is partitioned, we have to type
```
Running the ``%fs ls`` another time, now we can see the delta log file.
![[Pasted image 20250520095000.png]]
![[Pasted image 20250520095347.png]]
## Delta Table schema validation

For schema validation we will have to understand few basic things before we talk about schema validation. What is schema validation? 

When you try to insert records in an existing delta table, the delta table will validate the data type, column names, all those schema details of incoming data.

So if the incoming data is schema of the incoming data matches with the target table wherever you are trying to insert, it will allow you to insert new records.

But if there are any discrepancies, if a schema of the incoming data does not match with the target
table, it should raise an schema validation error. And the point is that schema validation comes into play when you try to insert records into an existing delta table. That's the time when schema validation triggers, and how many ways we can insert data into a delta table?

 ___Statements:___
1. INSERT
2. OVERWRITE
3. MERGE
4. DataFrame Append

 ___Validation Scenarions:___
1. Column matching approach
2. New Columns
3. Data Type Mismatch (Not allowed in any case)

Before deep into examples, I have created the following table to exemplify:

__dev.demo_db.people_tbl:__

|-- id: integer (nullable = true) 
|-- firstName: string (nullable = true) 
|-- lastName: string (nullable = true)
### 1. INSERT - Column matching by position (matching names not mandatory)

Here we can see that I have used different names on the `insert` statement, but the position is correct according the target table, it worked correctly. To conclude, columns are matched by position, not by the column names. So matching names are not mandatory, but this has a potential problem. For example, if I pass the gender instead the first name, as long as the data type match, it will allow you to insert the data. Then, we have to be aware with that.

![[Pasted image 20250520111059.png]]
### 2.INSERT - New columns are not allowed

From the same table, I have tried to insert the same columns from the previous example, adding a new column `dob`. How we can se below, I got a `schema mismatch`, because that is not allowed.

![[Pasted image 20250520111954.png]]
### 3.OVERWRITE - New columns are not allowed.

What if I try to overwrite this table and insert four columns instead of three old columns? The table has three columns and I want to insert four columns, I do not care about existing data. Is that allowed? No. This is a good thing, because delta table maintains different versions of the data. So overwrite will just create a new version of the same table, we are not dropping the table and recreating from the scratch. It is just overwrite the table with a new version, and that is why we have a schema validation for this operation also. 

![[Pasted image 20250520112915.png]]
### 4.MERGE - Column matching by name (matching by position not allowed)

Here I have tried to do a merge using the columns names exactly in the source table, that has different column names. There is a schema validation in place, and we got the message `Cannot resolve firstName in INSERT clause given columns`, that is because in merge the column names have to match.

![[Pasted image 20250520115310.png]]
### 5.MERGE - New columns are silently ignored 

Similarly, what happens if we try to add new columns in the merge statement? Now to do the merge I have fixed the column names according the target. So, as you can see below, after fix the names, I was able to merge and got no error. 

![[Pasted image 20250520120021.png]]
But, after query the table we can see that the column `dob` is not present, I still have three columns, because merge statement silently ignore the new column.  

![[Pasted image 20250520120321.png]]
### 6.DATAFRAME Append - Column matching by name (by position not allowed)

Now, I have used the Dataframe API to insert new records into a existing table, using different column names. I read a json file and have tried to append using spark write. As we can see, that does not work. If you try using dataframe API to insert or append new records in a existing table you have to match by name.

![[Pasted image 20250520140106.png]]
### 7.DATAFRAME Append - New columns also not allowed

To finish, if we try to insert a new column using API, also we will get an error, because the operation is not allowed.

![[Pasted image 20250520140129.png]]

## Schema evolution

We have learned previously that Delta Tables come with some type of validation in place. We have an option to allow the schema evolution automatically when we write new data into a Delta table. Without that, we see that Spark launch some erros with different schemas. How to use that?
### 1. Automatic schema evolution 

Using the same previous example, I have only added a new option `option("mergeSchema", "true")` and now, Spark allows me to write the data adding a new column, and very important and interesting point, the historic will be preserved. Here I am using the schema evolution at table level.

![[Pasted image 20250520141121.png]]
Querying the previous and the latest version:

![[Pasted image 20250520141536.png]]

Another way is to set it at the Spark configuration level in the session, using Python or SQL like this:

```python
spark.conf.set("spark.databricks.delta.schema.autoMerge.enabled", "true")
```

```sql
SET spark.databricks.delta.schema.autoMerge.enabled = true
```

***Keep in mind! The changes will be applied only in the current session!*** 

To check we can run also in Python or SQL:

```python
spark.conf.get("spark.databricks.delta.schema.autoMerge.enabled")
```

```sql
SET spark.databricks.delta.schema.autoMerge.enabled
```

![[Pasted image 20250520153604.png]]

Once we have this option set to true, we will be able to add new columns without any other statements. I used SQL but it will work also with Dataframe API, actually in a better way because API comes with a solid schema validation.   

![[Pasted image 20250520153922.png]]
### 1. Manual schema evolution 

If I want to do that manually, we can just create a new column using the command `ALTER TABLE`. I think I can say that is a better approach, because we will have more control about the pipline and data ingestion.
- __New column at the end__

Below we can see a table with three columns. I ran a query to add a new column and then, I ran a `INSERT INTO` with a new column, it has worked successfully. 

![[Pasted image 20250520144949.png]]
- __New column at the middle__:

In the previous example, just adding a new column, it will be created at the end of the table. If you want to set a position, we can run the following query:

```sql
ALTER TABLE catalog.schema.table 
ADD COLUMNS (new_column STRING after existing_column);
```

![[Pasted image 20250520145348.png]]
And be able to add data:

![[Pasted image 20250520145522.png]]

## REORG and DROP COLUMN

Before explain about REORG let's do something, try to drop some columns in our Delta Table. So, to do that we simply use the DDL command `ALTER TABLE DROP COLUMN`. 

As you can see in the following screenshot, we are not allowed to drop columns, but why? Drop columns are not enabled by default. To remove columns directly using this command, we have to have the property `columnMapping.mode` set up equal `name`. 

![[Pasted image 20250521112841.png]]
So to resolve this situation we can the following statement to change:

![[Pasted image 20250521113250.png]]
_Hint: We can see the protocols of the table using `DESCRIBE EXTENDED` command, and at the end of the table we have the column `Table Properties`. _

```sql
ALTER TABLE catalog.schema.table SET TBLPROPERTIES (
  'delta.columnMapping.mode' = 'name',
  'delta.minReaderVersion' = '2',
  'delta.minWriterVersion' = '5');
```
After doing that, we are able to drop columns as you can see:

![[Pasted image 20250521113354.png]]

**Warning** ⚠️

Doing that, the files will not be dropped, Spark just will mark the columns as deleted and only delete on the metadata! And the physical files will not be deleted. May you have huge parquet files in your table, to drop in fact we have to use `REORG`.

After drop the columns, if we check the history we can see a record where the deleted columns are registered:

![[Pasted image 20250521142703.png]]
### REORG

If you want to really delete the columns dropped earlier, we have to use reorg. It will look to the delta table or delta files searching for all the things are marked for deletion later. Running reorg it will go and physically remove everything that was marked for deletion. It will read those files, remove the deleted columns, values and rewrite those files and optimizations.

Before I run the `REORG` command I saved that following screenshot, here we can see nine parquet files in the folder: 

![[Pasted image 20250521143937.png]]
After run, we can see in the metrics: 9 files added and 9 files removed, but why? What happened here? This indicates that Databricks did a file layout optimization, probably rewriting the data. It just recreate the files with the new changings.

```sql
REORG TABLE catalog.schema.table APPLY(PURGE)
```

![[Pasted image 20250521144223.png]]
![[Pasted image 20250521144332.png]]

Also we get a new version:

![[Pasted image 20250521144845.png]]
https://docs.databricks.com/aws/en/sql/language-manual/delta-reorg-table

## Optimize and Zorder

Basically when we are ingesting or collecting data maybe in batch or streaming, we are also saving and writing the data into a delta table. We also know, there are different frequencies to do that, maybe every minute, every 30 minutes, every hour, etc... So maybe this situation can create a weird kind of files, small files. For example, suppose that I have an ingestion triggered every 10 minutes, ingesting small chunks of data, for each ingestion small files will be created into my delta table, and we know small files are a problem for Spark. Our files should not be very small and the recommended size is around 128mb, that is the default setting in some Spark configuration.

If you have created small or uneven files in your delta table due some reasons, we can optimize it using the command `OPTIMIZE` used to compress small files into larger, more efficient files.

Just to illustrate, I have created a table using the following statement, to create a new table only with players from England.

```sql
CREATE TABLE dev.fifa.players
AS SELECT * FROM fifa.silver.players WHERE nacionalidade_player = 'England'
```

![[Pasted image 20250522105928.png]]
Looking at ADSL, my table got only one parquet file:

![[Pasted image 20250522110057.png]]
After ran eight insert commands, I got eight new files into my table, totaling nine:

![[Pasted image 20250522110134.png]]
![[Pasted image 20250522110339.png]]
After run the optimize, we can see _"numFilesAdded"_ = 1 and "_numFilesRemoved"_ = 9, because the nine files were deleted in generated a new unique file according the size.

![[Pasted image 20250522110930.png]]

**Warning** ⚠️

By doing that, the files will not be deleted immediately, first optimize will just create a new version rewriting the data, and the old files will kept. To delete those files, we have to run `VACUUM` command.

![[Pasted image 20250522112545.png]]

### ZORDER

Zorder its a part of `OPTIMIZE`. 
