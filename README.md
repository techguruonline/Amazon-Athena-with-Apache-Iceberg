# Amazon-Athena-with-Apache-Iceberg
Insert / Update / Delete on S3 With Amazon Athena and Apache Iceberg

Athena ACID transactions powered by Apache Iceberg. Open Speccification, Snapshot-based table format for huge analytic datasets


# Introduction:
##### Iceberg is a high-performance format for huge analytic tables. Iceberg brings the reliability and simplicity of SQL tables to big data, while making it possible for engines like Spark, Trino, Flink, Presto, Hive and Impala to safely work with the same tables, at the same time.

*Create a new Database*

    CREATE DATABASE ATHENA_ICEBERG;

*Let's create a new table*

    CREATE TABLE CUSTOMER (
    CUST_ID INT,
    FIRST_NAME STRING,
    LAST_NAME STRING,
    PHONE_NO INT,
    JOIN_DT DATE
    )
    LOCATION 's3://emr-studio-demo-s3bucket-5he9azv0zpew/athena-iceberg/tables/'
    TBLPROPERTIES(
    'table_type' = 'ICEBERG'
    'format' = 'parquet'
    ));

*Insert data into the table*

    INSERT INTO CUSTOMER VALUES (1, 'Prasad', 'Nadig', 1234567890, '2020-01-01');\
    INSERT INTO CUSTOMER VALUES (2, 'Jeff', 'Bezos', 0987654321, '2000-01-01');\
    INSERT INTO CUSTOMER VALUES (3, 'Bit', 'Coin', 1357908642, '2020-01-01');\
    INSERT INTO CUSTOMER VALUES (4, 'Ethereum', 'Eth', 0864213579, '2020-01-01');\
    INSERT INTO CUSTOMER VALUES (5, 'Cardano', 'Ada', 6784567543, '2020-01-01');\
    INSERT INTO CUSTOMER VALUES (6, 'Cosmos', 'Atom', 6789054321, '2020-01-01');\
    INSERT INTO CUSTOMER VALUES (7, 'Polygon', 'Matic', 5647382910, '2020-01-01');\

--Select the data to validate inserts
SELECT * FROM CUSTOMER;

--Even though the data resides on S3, you can treat this like a traditional RDBMS i.e, you can run ACID transactions like UPDATE, INSERT, DELETE etc
--Now let's UPDATE a record
UPDATE CUSTOMER
SET PHONE_NO = 2468013579
WHERE CUST_ID = 2;

--Select the data to validate update
SELECT * FROM CUSTOMER WHERE CUST_ID = 2;

--Iceberg table provides powerful features like Timetravel which allows you to go back in time and see the data at that point in itme.
-- Let's try out the time travel to check what was the Phone number of Customer with Cust_id = 2, 5 mins back
SELECT * FROM CUSTOMER FOR SYSTEM_TIME AS OF (CURRENT_TIMESTAMP - INTERVAL '5' MINUTE) WHERE CUST_ID = 2;


-- You can also check the status of the table at a specific point in time
SELECT * FROM CUSTOMER FOR SYSTEM_TIME AS OF TIMESTAMP '2022-10-24 00:00:00' WHERE CUST_ID = 2;

-- Now let's delete couple of records
DELETE FROM CUSTOMER WHERE CUST_ID IN (6,7);

--Select the data to validate if the records were deleted
SELECT * FROM CUSTOMER;

--Again let's go back in time to check how the table looks like 5 mins back
SELECT * FROM CUSTOMER FOR SYSTEM_TIME AS OF (CURRENT_TIMESTAMP - INTERVAL '5' MINUTE);

