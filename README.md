# Snowflake Learn

## Snowflake Step
. Create Account & Activate
. Login to Snowflake
. Change the role to accountadmin
. start the default warehour
. create database
.


## Create Warehouse (Compute Engine/Server)

```
CREATE WAREHOUSE LAB_WAREHOUSE WITH WAREHOUSE_SIZE = 'XSMALL' WAREHOUSE_TYPE = 'STANDARD' AUTO_SUSPEND = 600 AUTO_RESUME = TRUE COMMENT = 'Lab Virtual Warehouse ';
```

## Create Database

    Create a Permanent Database
    Create a Transient Database
    Create a Database Replication


```
CREATE DATABASE LAB_DB DATA_RETENTION_TIME_IN_DAYS = 1;
create transient database db_transient_1;
create database mydb1
  as replica of aws_us_west_2.myaccount.mydb1
  auto_refresh_materialized_views_on_secondary = true;
```

## Switch 2 Database
```
use LAB_DB;
```

## Create Schema

```
Create schema schema-name;
```

## Create Tables

```
create table LAB_DB.lab_schema.users (
  id integer autoincrement,
  name varchar(100) not null,
  gender varchar(10),
  active boolean default true,
  created_timestamp timestamp,
  updated_timestamp timestamp
);

create table users (
  id integer autoincrement, -- auto incrementing IDs
  name varchar (100),  -- variable string column
  preferences variant, -- column used to store JSON type of data
  active boolean default true,
  created_at timestamp
);

create temporary table inactive_users (
  id integer autoincrement,
  name varchar(100) not null,
  active boolean default false
);


```

## Insert Data into Tables

```
insert into LAB_DB.lab_schema.users (id, name, gender, active, created_timestamp, updated_timestamp) 
values 
(1, 'Name1', null, 'True', current_timestamp(), current_timestamp()), 
(2, 'Name2', null, 'false', current_timestamp(), current_timestamp());
```
