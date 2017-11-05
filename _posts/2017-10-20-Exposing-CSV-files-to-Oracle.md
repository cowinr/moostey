---
layout: default
title: "Exposing CSV files to Oracle"
date: 2017-10-20
---

You can use SQL with a CSV file by exposing it to Oracle using a `DIRECTORY` object.
 
```sql 
  CREATE OR REPLACE DIRECTORY RC_HOME AS '/home/username/Oracle';
  GRANT READ, WRITE ON DIRECTORY RC_HOME TO MY_DB_USER;
```
 
This allows Oracle to access CSV files in my home area. The directory only needs to be reachable from the database server.
 
Typically there are few users that have the ability to create a `DIRECTORY`, so you may need to run the above as a more privileged user.
 
After placing a file, e.g. `test.csv`, in `/home/username/Oracle` you need to map it to an external `TABLE`:
 
```sql 
  CREATE TABLE TEST_CSV
  (
    Broker           VARCHAR2(100),
    Company          VARCHAR2(30),
    Created          VARCHAR2(10),
    Premium          VARCHAR2(20)
  )
  ORGANIZATION EXTERNAL
  (
    TYPE ORACLE_LOADER
    DEFAULT DIRECTORY RC_HOME
    ACCESS PARAMETERS
    (
      RECORDS DELIMITED BY NEWLINE
      SKIP 1
      FIELDS TERMINATED BY ','
      OPTIONALLY ENCLOSED BY '"'
      LRTRIM
      MISSING FIELD VALUES ARE NULL
    )
  LOCATION ('test.csv')
  );

  CREATE PUBLIC SYNONYM TEST_CSV FOR TEST_CSV;
  GRANT SELECT ON TEST_CSV TO MY_DB_USER;
``` 
 
The table and columns can be named whatever you want, and are mapped to the columns in the CSV file in order.
 
There are a number of options that can be set in the `ACCESS PARAMETERS` section, including data type mapping from `VARCHAR2` to `DATE` for example. Lots of detail in the docs.
 
Once the data is in the database as an external table, it can be mostly used like any other table.
