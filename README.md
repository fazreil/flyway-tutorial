# Flyway

![](images/Flyway_logo.svg.png)

---

## Who uses flyway?

According to Stackshare, companies that uses flyway diverse in nature and develop their applications using various programming languages.
source: https://stackshare.io/flyway

- https://www.insoft.lt/en
- https://www.doubleslash.de/en/company/company-portrait/
- 40 other companies identified by stackshare.io

---

## How do we do in non-prod now

### Collaborating on flyway script

There is a procedure being followed when developers prepare the SQL scripts (from now on referred as migration scripts). The script is being stored in one of our git repository.
ref: http://172.30.81.68:3000/DCP/dcp-be-flyway

This way all of our changes in the script is being recorded and versioned. This approach promotes auditability, traceability and also security. We emphasize on these three traits especially when the changes are brought in by collaboration of a team with multiple authors.


---

### GOGS

![Gogs](images/gogs.png)

---

### Managing changes of flyway scripts

Managing changes in the scripts are done by following maker-checker approach. In git, this is known as Pull Requests (PR).

ref: http://172.30.81.68:3000/DCP/dcp-be-flyway/pulls

Each changes are not written directly on the main branch but it is written on separate branch and then will be reviewed by another person before it can be merged into the main branch. This fulfills the maker-checker requirement. This approach also prevent any malicious/erroneous changes to be part of the main branch.

---

### Managing secrets

Flyway enables you to prevent the need to share credentials in order to operate databases.

ref: https://flywaydb.org/documentation/configuration/secretsManagement

You can store the credentials over in AWS, Hashicord Vault. This way you only need to specify the token or secret id without the need to specify the username and password pair.

Currently we do it in a different method when we use Jenkins's credential manager.

ref: http://172.30.81.70:8080/credentials/

---

### Jenkins job

![jenkins-dcp-be-flyway-deployment](images/jenkins-dcp-be-flyway-deployment.png)

---

### Jenkins pipeline code

We code in Jenkins to allow automation on using flyway. With jenkins we use jenkins credential manager to inject the secret without revealing what are the credentials:

ref: http://172.30.81.70:8080/job/dcp-be-flyway-deployment/

``` groovy
stage('Deploy to Environment') {
           steps{
                script {
                    if("${env.DEPLOYTO}" == "SIT_2") {
                        withCredentials([usernamePassword(credentialsId: 'sit-db', passwordVariable: 'password', usernameVariable: 'username')]) {
                            sh '''
                            mvn flyway:${COMMAND} -Dflyway.configFiles=flyway-sit.properties -Dflyway.user=${username} -Dflyway.password=${password}
                            '''
                        }

```
The resulting output from the script can be seen in Jenkins when executing the job:
```
[Pipeline] withCredentials
Masking supported pattern matches of $username or $password
[Pipeline] {
[Pipeline] sh
+ mvn flyway:migrate -Dflyway.configFiles=flyway-sit-2.properties -Dflyway.user=**** '-Dflyway.password=****'
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building flywaypro 4.8.0.0
[INFO] ------------------------------------------------------------------------
[INFO]
```
---

# Chapter 1

## Set up database
For this exercise we are going to work with a database called `tutorial`.

```sql
use tutorial;

SELECT *
FROM INFORMATION_SCHEMA.TABLES;  
```
will show:
``` sql
TABLE_CATALOG|TABLE_SCHEMA|TABLE_NAME|TABLE_TYPE|
-------------+------------+----------+----------+
```


---

## Setting up flyway connection

Create a file called flyway.conf with this content
``` java-properties
flyway.url=jdbc:sqlserver://172.17.0.2:1433;databaseName=tutorial
flyway.user=sa
flyway.password=RGBr0cks!
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


---

## First Change

Then we create our first change in the database. We are going to store all of the changes in a directory called `sql/`

We create a file in the said folder and we call it `sql/V1__Create_person_table.sql`
``` sql
create table PERSON (
    ID int not null,
    NAME varchar(100) not null
);
```

---

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


---

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


---

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


---

Let's check out what happen to the content of the tutorial database:

``` sql
use tutorial;

SELECT *
FROM INFORMATION_SCHEMA.TABLES;  
```

``` sql
TABLE_CATALOG|TABLE_SCHEMA|TABLE_NAME           |TABLE_TYPE|
-------------+------------+---------------------+----------+
tutorial     |dbo         |flyway_schema_history|BASE TABLE|
tutorial     |dbo         |PERSON               |BASE TABLE|
```


---

## flyway_schema_history
The database shows two new table, PERSON and flyway_schema_history. The PERSON table is intended to be created. flyway_schema_history in the other hand stores the history of flyway execution. The content of the flyway_schema_history shows as below:

```sql
use tutorial;

SELECT *
FROM dbo.flyway_schema_history;  
```
``` sql
installed_rank|version|description        |type|script                     |checksum  |installed_by|installed_on       |execution_time|success|
--------------+-------+-------------------+----+---------------------------+----------+------------+-------------------+--------------+-------+
             1|1      |Create person table|SQL |V1__Create_person_table.sql|1715188512|sa          |2021-06-28 17:24:40|             7|      1|
```


---

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

> **Repeatable migrations** have a description and a checksum, but no version. Instead of being run just once, they are **(re-)applied every time their checksum changes**.


---

## Clearing off to square one
There is a way to purge all changes we had done so far by calling the `flyway clean` command. However I don't think we are going to use this in production database.

>I am showing this command just to show that it is possible to start fresh. Useful especially when doing test and development.

```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf clean                                                                                                                                             
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully dropped pre-schema database level objects (execution time 00:00.002s)
Successfully cleaned schema [dbo] (execution time 00:00.259s)
Successfully dropped post-schema database level objects (execution time 00:00.037s)
```


---

# Chapter 2

## Let's add more content to the Person table
Let's prepare another sql script to add people into Person table. The name of the file is `sql/V2_Add_people.sql` :
``` sql
insert into PERSON (ID, NAME) values (1, 'Axel');
insert into PERSON (ID, NAME) values (2, 'Mr. Foo');
insert into PERSON (ID, NAME) values (3, 'Ms. Bar');
```


---

Creating a new versioned migration script will let flyway to detect a new script is pending for migration when running `flyway info`
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf info                                                                                                                                              
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: 1

+-----------+---------+---------------------+------+---------------------+---------+
| Category  | Version | Description         | Type | Installed On        | State   |
+-----------+---------+---------------------+------+---------------------+---------+
| Versioned | 1       | Create person table | SQL  | 2021-06-28 17:24:40 | Success |
| Versioned | 2       | Add people          | SQL  |                     | Pending |
+-----------+---------+---------------------+------+---------------------+---------+
```

---

Running `flyway validate` will result in error:
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf validate                                                                                                                                          
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
ERROR: Validate failed: Migrations have failed validation
Detected resolved migration not applied to database: 2. To fix this error, either run migrate, or set -ignorePendingMigrations=true.
Need more flexibility with validation rules? Learn more: https://flywaydb.org/custom-validate-rules/?ref=v7.10.0
```
Validate command shows error because of three reasons:
>    - differences in migration names, types or checksums are found
>    - versions have been applied that aren???t resolved locally anymore
>    - versions have been resolved that haven???t been applied yet



---

We go ahead and migrate version 2 of the script into our database
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf migrate                                                                                                                                           
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully validated 2 migrations (execution time 00:00.033s)
Current version of schema [dbo]: 1
Migrating schema [dbo] to version "2 - Add people"
```


---

Running `flyway info` will show that both script has been migrated successfully into out database.
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf info                                                                                                                                              
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: 2

+-----------+---------+---------------------+------+---------------------+---------+
| Category  | Version | Description         | Type | Installed On        | State   |
+-----------+---------+---------------------+------+---------------------+---------+
| Versioned | 1       | Create person table | SQL  | 2021-06-28 17:24:40 | Success |
| Versioned | 2       | Add people          | SQL  | 2021-06-29 17:21:47 | Success |
+-----------+---------+---------------------+------+---------------------+------
```

---

This time `flyway validate` will show no error:
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf validate                                                                                                                                     [130]
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully validated 2 migrations (execution time 00:00.038s)
```

---

Check out the content we just migrated over in table Person:
``` sql
use tutorial;

SELECT  * FROM  tutorial.dbo.PERSON p;
```

``` sql
ID|NAME   |
--+-------+
 1|Axel   |
 2|Mr. Foo|
 3|Ms. Bar|
```

---

## Migrate to certain version

There is a way to perform migration not applying all versions available. The following scenario will show that you are able to migrate up to targeted version.

To show this scenario, we are going to start fresh by executing `flyway clean`
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf clean                                                                                                                                             
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully dropped pre-schema database level objects (execution time 00:00.002s)
Successfully cleaned schema [dbo] (execution time 00:00.257s)
Successfully dropped post-schema database level objects (execution time 00:00.026s)
```

---

Now we are going to perform `flyway migrate` but with mentioning the target version we want to migrate

Pay attention to the option `-target=1` in the command, this will instruct flyway to perform migration up until version 1:
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf -target=1 migrate                                                                                                                                 
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully validated 2 migrations (execution time 00:00.032s)
Creating Schema History table [tutorial].[dbo].[flyway_schema_history] ...
Current version of schema [dbo]: << Empty Schema >>
Migrating schema [dbo] to version "1 - Create person table"
Successfully applied 1 migration to schema [dbo], now at version v1 (execution time 00:00.051s)
```

---

Running `flyway info` will show that version 2 have not taken place, it is pending for migration:
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf info                                                                                                                                              
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: 1

+-----------+---------+---------------------+------+---------------------+---------+
| Category  | Version | Description         | Type | Installed On        | State   |
+-----------+---------+---------------------+------+---------------------+---------+
| Versioned | 1       | Create person table | SQL  | 2021-06-29 18:12:59 | Success |
| Versioned | 2       | Add people          | SQL  |                     | Pending |
+-----------+---------+---------------------+------+---------------------+---------+
```

---

Querying from Person table will show empty table because version 2 have not taken place:
```sql
use tutorial;

SELECT  * FROM  tutorial.dbo.PERSON p;
```
```sql
ID|NAME|
--+----+
```

---

## What if there is a manual change in the table?

We are going to migrate fully our changes before we deep dive into this scenario

```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf migrate                                                                                                                                           
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully validated 2 migrations (execution time 00:00.041s)
Current version of schema [dbo]: 1
Migrating schema [dbo] to version "2 - Add people"
Successfully applied 1 migration to schema [dbo], now at version v2 (execution time 00:00.058s)
```

---

So I change the Person table by adding another column:
``` sql
ALTER TABLE tutorial.dbo.PERSON ADD ic varchar(100) NULL;
```
Changing the content like this will not break flyway:
```
~/checkouts/flyway-tutorial >>> flyway -configFiles=flyway.conf validate                                                                                                                                          
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=tutorial;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully validated 2 migrations (execution time 00:00.038s)
```

---

# Chapter 3

## Dealing with an existing database

Usually we deal with an existing database where it has been filled with structures and data. We do not have the luxury of redesigning the data nor start new and ditching the current database.

For this scenario, we are going to take the current database and `flyway baseline` it.

We are going to use another database called Northwind. Northwind database is having a complete set of table meant for its application.

---

To switch to Northwind, we need to have a new config file that will instruct Flyway to connect to Northwind database:

``` java-properties
flyway.url=jdbc:sqlserver://172.17.0.2:1433;databaseName=Northwind
flyway.user=sa
flyway.password=RGBr0cks!
flyway.locations=northwind
```

The new configuration will instruct flyway to connect to a new connection url and  the flyway locations will be pointing to new directory called northwind. Flyway will scan northwind for migration scripts.

---

Running `flyway info` will show no information

```
~/checkouts/flyway-tutorial >>> flyway -configFiles=northwind.conf info                                                                                                                                           
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=Northwind;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: << Empty Schema >>

+----------+---------+-------------+------+--------------+-------+
| Category | Version | Description | Type | Installed On | State |
+----------+---------+-------------+------+--------------+-------+
| No migrations found                                            |
+----------+---------+-------------+------+--------------+-------+
```

---

## Make changes to Northwind existing table

We are going to add two new migration script to Northwind database

- V1_Add_Ivan_Drago.sql
- V2_Add_fruit_category.sql

---

Below is the content of V1_Add_Ivan_Drago.sql

```sql
INSERT INTO Employees (LastName,FirstName,Title,TitleOfCourtesy,BirthDate,HireDate,Address,City,Region,PostalCode,Country,HomePhone,Extension)
  	VALUES ('Drago','Ivan','Sales Representative','Mr.','1984-07-01 00:00:00.000','2021-07-01 00:00:00.000','12, Notting Hill','Los Angeles','LA','91842','USA','(206) 555-0987','3241');
```

---

Below is the content of V2_Add_fruit_category.sql

```sql
INSERT INTO Northwind.dbo.Categories (CategoryName,Description)
	VALUES ('Fruit','Fresh uncut fruits');
```

---

Adding these two in northwind directory will make flyway to recognize the two migration scripts:

```
~/checkouts/flyway-tutorial >>> flyway -configFiles=northwind.conf info                                                                                                                                           
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=Northwind;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: << Empty Schema >>

+-----------+---------+--------------------+------+--------------+---------+
| Category  | Version | Description        | Type | Installed On | State   |
+-----------+---------+--------------------+------+--------------+---------+
| Versioned | 1       | Add Ivan Drago     | SQL  |              | Pending |
| Versioned | 2       | Add fruit category | SQL  |              | Pending |
+-----------+---------+--------------------+------+--------------+---------+
```

---

We are going to migrate these two scripts using flyway for the first time on the existing Northwind database:

> Note that the flyway_schema_history table is not there on the existing database


```
~/checkouts/flyway-tutorial >>> flyway -configFiles=northwind.conf migrate                                                                                                                                        
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=Northwind;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully validated 2 migrations (execution time 00:00.031s)
ERROR: Found non-empty schema(s) [dbo] but no schema history table. Use baseline() or set baselineOnMigrate to true to initialize the schema history table.
```

The message says that the migration scripts were successfully **validated** but there was an error because there are existing schema without the flyway_schema_history. Migration did not happen.

---

Checking `flyway info` will show the state is still in **Pending** state

```
~/checkouts/flyway-tutorial >>> flyway -configFiles=northwind.conf info                                                                                                                                        [1]
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=Northwind;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: << Empty Schema >>

+-----------+---------+--------------------+------+--------------+---------+
| Category  | Version | Description        | Type | Installed On | State   |
+-----------+---------+--------------------+------+--------------+---------+
| Versioned | 1       | Add Ivan Drago     | SQL  |              | Pending |
| Versioned | 2       | Add fruit category | SQL  |              | Pending |
+-----------+---------+--------------------+------+--------------+---------+
```

Baselining a version on existing database is required for flyway to work

---

Let's proceed with the baselining exercise:

```
~/checkouts/flyway-tutorial >>> flyway -configFiles=northwind.conf baseline                                                                                                                                       
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=Northwind;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Creating Schema History table [Northwind].[dbo].[flyway_schema_history] with baseline ...
Successfully baselined schema with version: 1
```

---

Take note that baselining will consume a version, therefore version 1 of the migration will be baseline. Due to this nature I am bumping my version 1 script to 1.1

``` bash
~/checkouts/flyway-tutorial >>> tree northwind                                                                                                                                                                    
northwind
????????? V1.1__Add_Ivan_Drago.sql
????????? V2__Add_fruit_category.sql

0 directories, 2 files
```

---

With that baseline, flyway info will tell us that version 1 was recorded as the baselining activity:

```
/checkouts/flyway-tutorial >>> flyway -configFiles=northwind.conf info                                                                                                                                           
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=Northwind;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: 1

+-----------+---------+-----------------------+----------+---------------------+----------+
| Category  | Version | Description           | Type     | Installed On        | State    |
+-----------+---------+-----------------------+----------+---------------------+----------+
|           | 1       | << Flyway Baseline >> | BASELINE | 2021-07-01 04:14:44 | Baseline |
| Versioned | 1.1     | Add Ivan Drago        | SQL      |                     | Pending  |
| Versioned | 2       | Add fruit category    | SQL      |                     | Pending  |
+-----------+---------+-----------------------+----------+---------------------+----------+
```

---

The table looks good for us to work with flyway now. We can proceed migrating the pending scripts to the database:

```
~/checkouts/flyway-tutorial >>> flyway -configFiles=northwind.conf migrate                                                                                                                                     [1]
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=Northwind;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Successfully validated 3 migrations (execution time 00:00.046s)
Current version of schema [dbo]: 1
Migrating schema [dbo] to version "1.1 - Add Ivan Drago"
Migrating schema [dbo] to version "2 - Add fruit category"
Successfully applied 2 migrations to schema [dbo], now at version v2 (execution time 00:00.093s)
```

---

A successful migration will yield the following output when we run `flyway info`

```
~/checkouts/flyway-tutorial >>> flyway -configFiles=northwind.conf info                                                                                                                                           
Flyway Community Edition 7.10.0 by Redgate
Database: jdbc:sqlserver://172.17.0.2:1433;maxResultBuffer=-1;sendTemporalDataTypesAsStringForBulkCopy=true;delayLoadingLobs=true;useFmtOnly=false;useBulkCopyForBatchInsert=false;cancelQueryTimeout=-1;sslProtocol=TLS;jaasConfigurationName=SQLJDBCDriver;statementPoolingCacheSize=0;serverPreparedStatementDiscardThreshold=10;enablePrepareOnFirstPreparedStatementCall=false;fips=false;socketTimeout=0;authentication=NotSpecified;authenticationScheme=nativeAuthentication;xopenStates=false;sendTimeAsDatetime=true;trustStoreType=JKS;trustServerCertificate=false;TransparentNetworkIPResolution=true;serverNameAsACE=false;sendStringParametersAsUnicode=true;selectMethod=direct;responseBuffering=adaptive;queryTimeout=-1;packetSize=8000;multiSubnetFailover=false;loginTimeout=15;lockTimeout=-1;lastUpdateCount=true;encrypt=false;disableStatementPooling=true;databaseName=Northwind;columnEncryptionSetting=Disabled;applicationName=Microsoft JDBC Driver for SQL Server;applicationIntent=readwrite; (Microsoft SQL Server 14.0)
Schema version: 2

+-----------+---------+-----------------------+----------+---------------------+----------+
| Category  | Version | Description           | Type     | Installed On        | State    |
+-----------+---------+-----------------------+----------+---------------------+----------+
|           | 1       | << Flyway Baseline >> | BASELINE | 2021-07-01 13:55:00 | Baseline |
| Versioned | 1.1     | Add Ivan Drago        | SQL      | 2021-07-01 14:33:11 | Success  |
| Versioned | 2       | Add fruit category    | SQL      | 2021-07-01 14:33:11 | Success  |
+-----------+---------+-----------------------+----------+---------------------+----------+
```
