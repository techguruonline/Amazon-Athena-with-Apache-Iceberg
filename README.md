# Amazon-Athena-with-Apache-Iceberg (Athena Engine V2)
Insert / Update / Delete on S3 With Amazon Athena and Apache Iceberg

Athena ACID transactions powered by Apache Iceberg. Open Speccification, Snapshot-based table format for huge analytic datasets


# Introduction:
##### Iceberg is a high-performance format for huge analytic tables. Iceberg brings the reliability and simplicity of SQL tables to big data, while making it possible for engines like Spark, Trino, Flink, Presto, Hive and Impala to safely work with the same tables, at the same time. 
##### Iceberg supports flexible SQL commands to merge new data, update existing rows, and perform targeted deletes. Iceberg can eagerly rewrite data files for read performance, or it can use delete deltas for faster updates.
##### Supports Full Schema Evolution and Time travel / Rollback features
##### For more information, please visit https://iceberg.apache.org/ 


## Let's start with creating a database in Athena

*Create a new Database*

    CREATE DATABASE IF NOT EXISTS ATHENA_ICEBERG;

*Let's create a new table, notice Athena natively supports ICEBERG as the table type*

    CREATE TABLE IF NOT EXISTS CUSTOMER (
    CUST_ID INT,
    FIRST_NAME STRING,
    LAST_NAME STRING,
    PHONE_NO INT,
    JOIN_DT DATE
    )
    LOCATION 's3://misc-experimental/athena-iceberg/tables/'
    TBLPROPERTIES(
    'table_type' = 'ICEBERG',
    'format' = 'parquet'
    );

### Insert / Update / Delete Operations

*Insert data into the table*
    
    INSERT INTO CUSTOMER VALUES (1, 'Prasad', 'Nadig', 1234567890, current_date);
    INSERT INTO CUSTOMER VALUES (2, 'Jeff', 'Bezos', 0987654321, current_date);
    INSERT INTO CUSTOMER VALUES (3, 'Bit', 'Coin', 1357908642, current_date);
    INSERT INTO CUSTOMER VALUES (4, 'Ethereum', 'Eth', 0864213579, current_date);
    INSERT INTO CUSTOMER VALUES (5, 'Cardano', 'Ada', 678456754, current_date);
    INSERT INTO CUSTOMER VALUES (6, 'Cosmos', 'Atom', 678905432, current_date);
    INSERT INTO CUSTOMER VALUES (7, 'Polygon', 'Matic', 564738291, current_date);

*Select the data to validate inserts*
    
    SELECT * FROM customer ORDER BY cust_id;

Even though the data resides on S3, you can treat this like a traditional RDBMS i.e, you can run ACID transactions like UPDATE, INSERT, DELETE etc
Now let's UPDATE a record
    
    UPDATE customer
    SET phone_no = 246801357
    WHERE cust_id = 2;

*Select the data to validate update*
    
    SELECT * FROM customer WHERE CUST_ID = 2;

Iceberg table provides powerful features like Timetravel which allows you to go back in time and see the data at that point in itme.
Let's try out the time travel to check what was the Phone number of Customer with Cust_id = 2, 5 mins back <br>
    
    SELECT * FROM customer FOR SYSTEM_TIME AS OF (CURRENT_TIMESTAMP - INTERVAL '5' MINUTE) WHERE CUST_ID = 2;


You can also check the status of the table at a specific point in time
Get the current timestanp

    SELECT current_timestamp;  

    SELECT * FROM customer FOR SYSTEM_TIME AS OF TIMESTAMP '2022-10-24 00:00:00' WHERE CUST_ID = 2;

Now let's DELETE couple of records

    DELETE FROM customer WHERE CUST_ID IN (6,7);

Select the data to validate if the records were deleted

    SELECT * FROM customer ORDER BY cust_id;

Again let's go back in time to check how the table looks like 2 mins back

    SELECT * FROM customer FOR SYSTEM_TIME AS OF (CURRENT_TIMESTAMP - INTERVAL '2' MINUTE) ORDER BY cust_id;

### Schema Evolution

##### Iceberg schema updates are metadata-only changes. No data files are changed when you perform a schema update.

*Add a new column to Customer table*

    ALTER TABLE customer ADD COLUMNS (loc string);

*Insert new record including the newly add column*

    INSERT INTO customer VALUES (7, 'Solana', 'Sol', 564738291, current_date, 'New York');

*Select the table to check the data*

    SELECT * FROM customer ORDER BY cust_id;

*Rename a column*

    ALTER TABLE customer CHANGE COLUMN loc state string;

*show the columns in a table*

    SHOW COLUMNS FROM customer;

*Drop column*

    ALTER TABLE customer DROP COLUMN state;

*Select the table to check the data*

    SELECT * FROM customer ORDER BY cust_id;

    SHOW COLUMNS FROM customer;


# Managing Iceberg Tables
Athena supports the following table DDL operations for Iceberg tables.

*Rename table*
    
    ALTER TABLE customer RENAME TO customers;

*Set table properties for an existing table, Add properties to an Iceberg table and sets their assigned values.*
    
    ALTER TABLE customers SET TBLPROPERTIES (
    'write_target_data_file_size_bytes'='536870912', 
    'optimize_rewrite_delete_file_threshold'='10'
    )

*Show table properties*

    SHOW TBLPROPERTIES customers;


*Unset table properties*
    
    ALTER TABLE customers UNSET TBLPROPERTIES ('write_target_data_file_size_bytes');

*Run the show table properties again to verify*

    SHOW TBLPROPERTIES customers;

*Describe table, when the FORMATTED option is specified, the output displays additional information such as table location and properties.*
    
    DESCRIBE customers;
    DESCRIBE FORMATTED customers;

*Show create table, displays a CREATE TABLE DDL statement that can be used to recreate the Iceberg table*
    
    SHOW CREATE TABLE customers;

*Finally DROP table, drop table not only drops the table metadata, it also deletes the underlying data on S3*
    
    DROP TABLE customers;


# Consideration and Limitations

Tables with Amazon Glue catalog only – Only Iceberg tables created against the Amazon Glue catalog based on specifications defined by the open source glue catalog implementation are supported from Athena.

Table locking support by Amazon Glue only – Unlike the open source Glue catalog implementation, which supports plug-in custom locking, Athena supports Amazon Glue optimistic locking only. Using Athena to modify an Iceberg table with any other lock implementation will cause potential data loss and break transactions.

Parquet files only – Currently, Athena supports Iceberg tables in Parquet file format only. ORC and AVRO are not supported.

Lake Formation – Integration with Amazon Lake Formation is not supported.

Unsupported operations – The following Athena operations are not supported for Iceberg tables.

    CREATE TABLE AS
    ALTER TABLE SET LOCATION
    CREATE VIEW
    SHOW CREATE VIEW
    DROP VIEW
    DESCRIBE VIEW
