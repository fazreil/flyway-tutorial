# Flyway tutorial

![](images/Flyway_logo.svg.png)

## Set up database
For this exercise we are going to work with a database called `tutorial`.

```sql
use tutorial;

SELECT *
FROM INFORMATION_SCHEMA.TABLES;  
```
will show:
```
TABLE_CATALOG|TABLE_SCHEMA|TABLE_NAME|TABLE_TYPE|
-------------+------------+----------+----------+
```


## Setting up flyway connection

Create a file called flyway.conf with this content
``` java-properties
flyway.url=jdbc:sqlserver://172.17.0.2:1433;databaseName=tutorial
flyway.user=sa
flyway.password=RHBr0cks!
```
When we do `flyway -configFiles=flyway.conf info` the information of the database will appear, indicating a successful connection.
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf info                                                                                                                                                                       
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: << Empty Schema >>

+----------+---------+-------------+------+--------------+-------+
| Category | Version | Description | Type | Installed On | State |
+----------+---------+-------------+------+--------------+-------+
| No migrations found                                            |
+----------+---------+-------------+------+--------------+-------+
```

We instruct flyway to pickup the configuration file by defining option `-configFiles=flyway.conf` in our command.

## First Change

Then we create our first change in the database. We are going to store all of the changes in a directory called `sql/`

We create a file in the said folder and we call it `sql/V1__Create_person_table.sql`
``` sql
create table PERSON (
    ID int not null,
    NAME varchar(100) not null
);
```
Running `flyway info` this time around will give you the following output
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf info                                                                                                                                              
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: << Empty Schema >>

+-----------+---------+---------------------+------+--------------+---------+
| Category  | Version | Description         | Type | Installed On | State   |
+-----------+---------+---------------------+------+--------------+---------+
| Versioned | 1       | Create person table | SQL  |              | Pending |
+-----------+---------+---------------------+------+--------------+---------+
```

note that the State at the end says `Pending`

Let's apply the change into our database. We do that by running `migrate` command:

```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf migrate                                                                                                                                           
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully validated 1 migration (execution time 00:00.046s)
Creating Schema History table [tutorial].[dbo].[flyway_schema_history] ...
Current version of schema [dbo]: << Empty Schema >>
Migrating schema [dbo] to version "1 - Create person table"
Successfully applied 1 migration to schema [dbo], now at version v1 (execution time 00:00.073s)
```
The last line above says `Successfully applied 1 migration to schema [dbo], now at version v1`, indicating the migration/change has been applied successfully.

Let's check what `flyway info` says now:
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf info                                                                                                                                              
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: 1

+-----------+---------+---------------------+------+---------------------+---------+
| Category  | Version | Description         | Type | Installed On        | State   |
+-----------+---------+---------------------+------+---------------------+---------+
| Versioned | 1       | Create person table | SQL  | 2021-06-28 16:55:49 | Success |
+-----------+---------+---------------------+------+---------------------+---------+

```
The last line says that changes version 1 has been migrated successfully.

Let's check out what happen to the content of the tutorial database:

``` sql
use tutorial;

SELECT *
FROM INFORMATION_SCHEMA.TABLES;  
```

```
TABLE_CATALOG|TABLE_SCHEMA|TABLE_NAME           |TABLE_TYPE|
-------------+------------+---------------------+----------+
tutorial     |dbo         |flyway_schema_history|BASE TABLE|
tutorial     |dbo         |PERSON               |BASE TABLE|
```

## flyway_schema_history
The database shows two new table, PERSON and flyway_schema_history. The PERSON table is intended to be created. flyway_schema_history in the other hand stores the history of flyway execution. The content of the flyway_schema_history shows as below:

```sql
use tutorial;

SELECT *
FROM dbo.flyway_schema_history;  
```
```
installed_rank|version|description        |type|script                     |checksum  |installed_by|installed_on       |execution_time|success|
--------------+-------+-------------------+----+---------------------------+----------+------------+-------------------+--------------+-------+
             1|1      |Create person table|SQL |V1__Create_person_table.sql|1715188512|sa          |2021-06-28 17:24:40|             7|      1|
```

## What if we rerun the migration script

Out of curiousity, what happen if we perform migrate again, without any new migration script:
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf migrate                                                                                                                                           
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully validated 1 migration (execution time 00:00.049s)
Current version of schema [dbo]: 1
Schema [dbo] is up to date. No migration necessary.
```
Flyway will report saying that the schema is up to date. There are no changes being done.

NOTE: **Repeatable migrations** have a description and a checksum, but no version. Instead of being run just once, they are **(re-)applied every time their checksum changes**.

## Clearing off to square one
There is a way to purge all changes we had done so far by calling the `flyway clean` command. However I don't think we are going to use this in production database.

I am showing this command just to show that it is possible to start fresh. Useful especially when doing test and development.

```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf clean                                                                                                                                             
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully dropped pre-schema database level objects (execution time 00:00.002s)
Successfully cleaned schema [dbo] (execution time 00:00.259s)
Successfully dropped post-schema database level objects (execution time 00:00.037s)
```
