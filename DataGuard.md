# 1. Why We Need to Enable FORCE LOGGING Option in the Data Guard Environment

``` sql
ALTER DATABASE FORCE LOGGING;
```

The **FORCE LOGGING** option is the safest method to ensure that all
changes made in the database will be captured and available for recovery
in the redo logs.

FORCE LOGGING is a feature added to the family of logging attributes.

## Tablespace-Level FORCE LOGGING

``` sql
ALTER TABLESPACE <tablespace_name> FORCE LOGGING;

ALTER TABLESPACE <tablespace_name> NO FORCE LOGGING;
```

Oracle 9i Release 2 introduced the **FORCE LOGGING** option.

The FORCE LOGGING option can be set at the **database level** or
**tablespace level**.

The precedence is from **database to tablespace**.

If a tablespace is created or altered to have FORCE LOGGING enabled, any
change in that tablespace will go into the redo log and be available for
recovery.

Similarly, if a database is created or altered to have FORCE LOGGING
enabled, any change across the database, with the exception of
**temporary segments and temporary tablespaces**, will be available in
the redo logs for recovery.

The FORCE LOGGING option can be set at database creation time or later
using the `ALTER DATABASE` command.

## Check FORCE LOGGING Status

``` sql
SELECT force_logging
FROM v$database;
```

To check the FORCE LOGGING status of a tablespace:

``` sql
SELECT force_logging
FROM dba_tablespaces;
```
# 2. What is the difference between LOGGING and NOLOGGING in Oracle?

If you specify **LOGGING**, then the creation of a database object, as
well as subsequent inserts into the object, will be logged in the redo
log file.

If you specify **NOLOGGING**, then the creation of a database object, as
well as subsequent conventional inserts, will be logged in the redo log
file.


# **Oracle Data Guard: Architecture:**

**Primary database, Data Guard uses the following processes:**

**Log writer (LGWR): LGWR collects transaction redo information and updates the online redo logs.**

**Synchronous (SYNC) standby database, LGWR passes the redo to an LNS (Log Writer Network Server) process, which ships the redo directly to the remote file server (RFS) process on the standby database. LGWR waits for confirmation from the LNS process before acknowledging the commit.**

**Asynchronous (ASYNC)standby databases, independent LNS processes read the redo from either the redo log buffer in memory or the online redo log file, and then ship the redo to its standby database.**

**Archiver (ARCn): The ARCn process creates a copy of the online redo log files locally for use in a primary database recovery operation. ARCn is also responsible for shipping redo data to an RFS process at a standby database and for proactively detecting and resolving gaps on all standby databases.**

**The Standby database, Data Guard uses the following processes:**

**Remote file server (RFS):**

**RFS receives redo information from the primary database and write into standby redo logs.**

**Archiver (ARCn): The ARCn process archives the standby redo logs.**



**Managed recovery (MRP):**

**   For physical standby databases only, MRP applies archived redo log information to the physical standby database.**

**Logical standby (LSP):**

**   For logical standby databases only, LSP controls the application of archived redo log information to the logical standby database.**



**How Many Type of Standby Databases In Oracle :**

## **Physical Standby Database:**

**    It’s identical to the primary database on a block-for-block basis.**

**It’s synchronized with the primary database through redo data received from the primary database.**

**Can be used concurrently for data protection and reporting.**

## **Logical Standby Database:**

**  It’s Shares the same schema definition**

**It’s kept synchronized with the primary database by transforming the data in the redo received from the primary database into SQL statements and then executing the SQL statements.**

**Can be used concurrently for data protection, reporting, and database upgrades**

## **Snapshot standby database:**

**     It’s a fully updatable standby database.**

**It’s created by converting a physical standby database**

**Can be used for updates, but those updates are discarded before the snapshot standby database is converted back into a physical standby database.**

**The snapshot standby database is appropriate when you require a temporary, updatable version of a physical standby database.**

**Data Protection Modes In Oracle DG :**

## **Maximum Protection:**

**This protection mode guarantees that no data loss occurs if the primary database fails.the redo data that is needed to recover  must be written to both the local online redo log and the standby redo log  before the transaction commits. In this mode To ensure that data loss does not occur, the primary database shuts down if redo is not shipped to standby database.**



## **Maximum Availability:**

**It works similar Maximum protection but the primary database does not shut down if redo is not shipped**

**to a remote standby redo log. Instead, the primary database operates in an unsynchronized mode until the fault is corrected and all the gaps in the redo log files are resolved.**

## **Maximum Performance ( Default)**

**     In case of maximum performance this allow a transaction to commit as soon as the redo data needed to recover that transaction is written to the local online redo log.**

**Why we need to enable Force Logging Mode:**

## ** FORCE LOGGING mode is recommended to ensure data consistency and forces redo to be generated even when NOLOGGING operations are executed.**

**Standby Redo Logs:**

**  Are used to store redo data received from the primary database.**

**Synchronous transport mode,Real-time apply**

# **What is real-time apply in DG ?**

**Log buffer on primary is read by LGWR and sent to redo shipping process LNS. LNS now transfers this to RFS and is written to SRL's. MRP will apply from SRL.**

**Setting Initialization Parameters on the Primary Database to Control Redo Transport**

**LOG\_ARCHIVE\_CONFIG,LOG\_ARCHIVE\_DEST\_n,DB\_FILE\_NAME\_CONVERT,LOG\_FILE\_NAME\_CONVERT,STANDBY\_FILE\_MANAGEMENT,DB\_UNIQUE\_NAME**

**SYNC: Specifies that redo data generated by a transaction must have been received at a destination before the transaction can commit; otherwise, the destination is deemed to have failed.**

**ASYNC (default): Specifies that redo data generated by a transaction need not have been received at a destination that has this attribute before the transaction can commit**

**AFFIRM: Specifies that a redo transport destination acknowledges redo data received after writing it to the standby redo log.**

**NOAFFIRM: Specifies that a redo transport destination acknowledges received redo data before writing it to the standby redo log.**

**the default is AFFIRM when the SYNC attribute is specified and NOAFFIRM when the ASYNC attribute is specified.**

# **Logical Standby Database or SQL Apply Architecture:**

**A logical standby database provides benefits in disaster recovery, high availability, and data protection that are similar to those of a physical standby database.**

**Benefits of Implementing a Logical Standby Database**

**Provides data protection:**

**Primary database corruptions not propagated**

**Provides disaster recovery capabilities:**

**-Switchover and failover**

**– Minimizes down time for planned and unplanned outages**

**Can be used to upgrade Oracle Database software and apply patch sets**

**How to Create and Managing a Snapshot Standby Database?**

**A snapshot standby database is a fully updatable standby database created by converting a physical standby database.**

**Snapshot standby databases receive and archive—but do not apply—redo data from a primary database.**

**When the physical standby database is converted, an implicit guaranteed restore point is created and Flashback Database is enabled.**

# **What is Oracle Active Data Guard:**

**Its a new feature of 11g database.**

**This feature includes**

** 1. Real-time query**

** 2. RMAN block change tracking on a physical standby database.**

# **What is fast sync?**

**Data Guard maximum availability supports the use of the NOAFFIRM redo transport attribute. A standby database returns receipt acknowledgment to its primary database as soon as redo is received in memory. The standby database does not wait for the Remote File Server (RFS) to write to a standby redo log file.**

**This feature provides increased primary database performance in Data Guard configurations using maximum availability and SYNC redo transport. Fast Sync isolates the primary database in a maximum availability configuration from any performance impact due to slow I/O at a standby database.**

# **What is far sync?**

**Oracle Far Sync is an Oracle 12c new feature for Oracle Data Guard. This feature is meant to resolve the performance problems induced by network latency when we maintain a standby database geographically distant of the primary database. In this type of situation we sometimes have to make a compromise between performance and data loss. The Far Sync feature offer you both.**

**Far Sync instance receive data synchronously from the primary database and then forward it asynchronously to up de 29 remote destinations.**

**The far sync database is not a standard database, it only contains a specific controlfile, a spfile and standby redologs. This database must be placed near the primary database to guarantee an optimal network latency during synchronous replication. But be careful, don’t place this database on the same geographical place than the primary, because if your primary database experiences a geographical disaster, your Far Sync will be impacted too, and some data could be lost.**

**In case of an outage on the primary database, the standard failover procedure applies and the far sync instance guarantee that no data is lost during the failover.**

## **Step by step to Creating Standby database**

**1. Enable Forced Logging  (Primary database)**

**2. Create a Password File**

**3. Configure a Standby Redo Log**

**   a. Ensure log file sizes are identical on the primary and standby databases.**

**The size of the current standby redo log files must exactly match the size of the current primary database online redo log files.**

**   b. Determine the appropriate number of standby redo log file groups.**

**Minimally, the configuration should have one more standby redo log file group than the number of online redo log file groups on the primary database. However, the recommended number of standby redo log file groups is dependent on the number of threads on the primary database. Use the following equation to determine an appropriate number of standby redo log file groups:**

**(maximum number of logfiles for each thread + 1) \* maximum number of threads**

**   c. Create standby redo log file groups. >>>ALTER DATABASE ADD STANDBY LOGFILE GROUP 10**

# **4. Set Primary Database Initialization Parameters**

**Primary : PROD**

**DB\_NAME=PROD**

**DB\_UNIQUE\_NAME=PROD**

**LOG\_ARCHIVE\_CONFIG='DG\_CONFIG=(PROD,STBY)'**

**CONTROL\_FILES='/arch1/PROD/control1.ctl', '/arch2/PROD/control2.ctl'**

**LOG\_ARCHIVE\_DEST\_1=**

** 'LOCATION=/arch1/PROD/**

**  VALID\_FOR=(ALL\_LOGFILES,ALL\_ROLES)**

**  DB\_UNIQUE\_NAME=PROD'**

**LOG\_ARCHIVE\_DEST\_2=**

** 'SERVICE=STBY LGWR ASYNC**

**  VALID\_FOR=(ONLINE\_LOGFILES,PRIMARY\_ROLE)**

**  DB\_UNIQUE\_NAME=STBY'**

**LOG\_ARCHIVE\_DEST\_STATE\_1=ENABLE**

**LOG\_ARCHIVE\_DEST\_STATE\_2=ENABLE**

**REMOTE\_LOGIN\_PASSWORDFILE=EXCLUSIVE**

**LOG\_ARCHIVE\_FORMAT=%t\_%s\_%r.arc**

**LOG\_ARCHIVE\_MAX\_PROCESSES=30**

**Primary Database: Standby Role Initialization Parameters**

**FAL\_SERVER=STBY**

**FAL\_CLIENT=PROD**

**DB\_FILE\_NAME\_CONVERT='STBY','PROD'**

**LOG\_FILE\_NAME\_CONVERT=**

** '/arch1/STBY/','/arch1/PROD/','/arch2/STBY/','/arch2/PROD/'**

**STANDBY\_FILE\_MANAGEMENT=AUTO**



**Verifying Flashback and converting to snapshot standby database**

**1. Flash back should be on**

**DGMGRL> connect sys/Welcome12**

**Connected.**

**DGMGRL> edit database 'PROD' set state='apply-on';**

**DGMGRL> convert database PROD to snapshot standby;**

**Convert the snapshot standby database back to a physical standby database.**

**DGMGRL> convert database PROD to physical standby;**

**Active Data guard >>>>**

**DGMGRL> connect sys/Welcome12**

**Disable Redo Apply on your physical standby database.**

**DGMGRL> edit database 'PROD' set state='apply-off';**

**sqlplus / as sysdba >>>>>>> On standby**

**SQL> alter database open read only;**

**Restart Redo Apply and confirm your change. Exit DGMGRL**

**DGMGRL> edit database 'PROD' set state='apply-on';**

**We can enable block change tracking on Active dataguard**

**SQL> alter database enable block change tracking;**

# **How to Change the protection mode in DG?**
```
**show database PROD LogXptMode;**

**LogXptMode = 'ASYNC'**

**show parameter log\_archive\_dest**

**<< Output formatted below for display >>**

**NAME TYPE VALUE**

**--------------------- ------- ----------------------------**

**log\_archive\_dest\_1 string LOCATION=**

**USE\_DB\_RECOVERY\_FILE\_DEST**

**log\_archive\_dest\_2 string service=PROD async valid\_for**

**(online\_logfile,primary\_role)**

**db\_unique\_name=PROD**

**DGMGRL> edit database 'PROD' set property**

**'LogXptMode'='SYNC';**

**Property "LogXptMode" updated**

**DGMGRL> edit configuration set protection mode as**

**maxavailability;**

**Succeeded.**

**set the protection mode back to MAXPERFORMANCE.**

**\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~**

**DGMGRL> edit configuration set protection mode as**

**maxperformance;**

**Succeeded.**

**DGMGRL> edit database PROD set property**

**'LogXptMode'='ASYNC';**

**Property "LogXptMode" updated**

**Enabling Fast-Start Failover**

**DGMGRL> edit database pc01prmy**

**> set property FastStartFailoverTarget = PROD;**

**Property "faststartfailovertarget" updated**

**DGMGRL> edit database PROD**

**> set property FastStartFailoverTarget = pc01prmy;**

**Property "faststartfailovertarget" updated**

**Set the fast-start failover threshold to 90 seconds.**

**DGMGRL> edit configuration**

**> set property FastStartFailoverThreshold=90;**

**Property "faststartfailoverthreshold" updated**

**DGMGRL> enable fast\_start failover;**

**Enabled.**
```
Putting a database in **FORCE LOGGING** mode will have some
**performance impact**.
