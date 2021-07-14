## TiDB Doris connector



#### Summary

TiDB and Doris are both decent opensource relational data database. TiDB is a fusion distributed database product, while Doris is more focused on the OLAP field. If TiDB can hand over part of the OLAP analysis task to Doris, it can realize efficient large-scale OLAP analysis with the help of Doris's physical and chemical view, aggregation model and other advantages. Doris design documentation and related content can be found on the [official website of Doris](https://doris.apache.org/).


#### EXPLANATION OF THE DORIS VOCABULARY

- Be: Doris' Backend process, responsible for query and import task plan execution, as well as data storage.
- FE: Frontend process of the Doris, which is responsible for query and import task plan generation, and metadata management and storage.


#### MAIN MODULES
In Doris, the streaming import functionality is mainly divided into the following layers

##### Parsing
This layer includes the HTTP request parsing on BE and the generation portion of the import plan.

##### Executive
This layer includes the execution process of the import plan on BE, including the process of reading, converting and distributing data.

##### Transaction management
Any streaming import is treated as an atomic transaction in Doris. This layer is mainly responsible for the start, commit, and rollback of the import transaction to ensure the atomic of the import.

##### Storage 
This layer includes the steps to store the imported data after BE receives it, which will not be specifically described in this article.


#### DATA MODEL
In Doris, data is logically described in the form of tables. A table contains rows and columns. A Row is a Row of data for the user. Column is used to describe the different fields in a Row of data.

Column can be divided into two broad categories: Key and Value. From a business perspective, Key and Value can correspond to dimension columns and metric columns, respectively.

The data model of Doris is mainly divided into three categories:
##### Aggregate

Aggregation model. When we import the data, the same rows for the Key column are aggregated into one row, while the Value column is aggregated according to the set AggregationType. There are currently four types in AggregationType:

- SUM: adding up the Value of multiple rows.
- Replace: where the Value in the next batch of data replaces the Value in the previously imported row.
- Max: keeping the maximum value.
- MIN: keeping the minimum.

##### Uniq

In some multi-dimensional analysis scenarios, users are more concerned about how to ensure the uniqueness of Key, that is, how to obtain the uniqueness constraint of Primary Key. Therefore, we introduce the data model of UNIQ. This model is essentially a special case of the aggregate model and a simplified representation of the table structure.

##### Duplicate
The data has neither primary keys nor aggregation requirements.


#### IMPORT DATA

There are many ways to import data in Doris. There are two ways that are directly related to this Hackathon: Stream Load and RoutineLoad.

- Stream load
The user submits the request over the HTTP protocol and creates the import with the raw data. Mainly used to quickly import data from a local file or data stream into the Doris. The import command returns the import results synchronously. Reference documentation

- Routine load
The user submits a routine import job via the MySQL protocol, spawns a permanent thread that continuously reads data from a data source (such as Kafka) and imports it into Doris. Reference documentation

In a variety of storage formats, Aggregate and Duplicate only support INSERT and DELETE, while UNIQ supports batch addition and update through import, which can realize data modification in another way

The data source can choose TIDB binlog and read the binlog to synchronize the data incrementally into the Doris.


#### DESIGN

When importing data, you can choose the storage model of the table according to the actual situation. If only the newly added table can be any model, and if the data is modified or deleted, it is recommended to use the UNIQ model, and use the Batch Delete function to add a flag column to identify the operation type of the data. You can refer to the documentation for more detailed information.

There are many kinds of implementation schemes. The following design ideas are given.

Stream Load Scheme
This method is relatively simple to implement and design. It can start or design an independent service in TIDB, read the TIDB binlog file on time and regularly, parse the contents of the binlog, assemble the data rows into the CSV format supported by Doris, and import them into Doris through stream load.

Routine Load scheme
This approach uses TIDB's Drainer to synchronize the binlog to Kafka, and a new TiDB binlog data format is added to Doris to synchronize the data.

TIDB native protocol synchronization scheme
The synchronization protocol of TIDB replica is implemented in Doris, which disguises Doris as a node of TIDB cluster, takes over the data synchronization request from TIDB, parses the data format, and writes the data into Doris


### Your RFC/Proposal?
<!-- A link to the RFC/Proposal. -->

