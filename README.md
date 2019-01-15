# SQL-Server-Maintenance
Standardized database maintenance for SQL Server, built over popular Ola Halengreen's procedures.

Official documentation of one-script maintenance solution. More detailed info within [documentation file](docs/SQL%20Server%20Maintenance%20-%20documentation.pdf).

Table of contents:
  * [Introduction](#1-introduction)
  * [Technical preview](#2-technical-preview)
  * [Deployment](#3-deployment)

## 1 Introduction

This document describes maintenance solution used for keeping SQL Server in good condition purposes as agreed maintenance standard used on all SQL Server environments which is deployed after SQL Server instance installation. By default it is configured on agreed schedules but can be manually customized after deployment to the instance directly in SQL Agent jobs.
All environment specific need can be meet by modifications to standard implementation, for more info regarding to customization of Ola Halengreen’s scripts can be found at official sites.
Maintenance can be deployed to SQL Server versions >= 2005 (SQL Server 2000 not supported but some solution is in long term plans to have one script maintenance solution as this one)

## 2 Technical preview

Whole solution is mostly based on popular Ola Hallengren’s maintenance procedures (visit here for more details https://ola.hallengren.com/). Olla’s procedures have been modified so as to fit needs described in standard specified in our team. Script is creating maintenance jobs as its main objection. All tasks performed on the target SQL Server instance are listed below.

Script perform following stuff:
 * create OH procedures in master database
 * create and drop helper procedures in master database
 * create SQL Agent jobs (full backup, differential backup, log backup, integrity check and index optimize, recycle error log)
 * schedule SQL Agent jobs based on agreed standard times
 * rename maintenance job syspolicy_purge_history
 * optimize tempdb according to best practices (this is optional)
 * change initial size and Autogrowth on model and tempdb database based on our agreed standard

Some of above described actions are not executed automatically but can be fired by changing input parameters, for more info see section [Deployment](#deployment)

All important info also included in script header, containing also some short update history.

## 3 Deployment 

Only thing you have to do is to copy script from its official storage on GitHub. Copy script to SQL Server Management Studio and set parameters if needed, by default parameters are set as following (all of them are pretty self-commented). You can change them based on environment you are deploying to.

```
SET @CreateJobs = 'Y' -- Create maintenance jobs Y/N
SET @LogToTable = 'Y' -- Log commands to a table Y/N
SET @NotifyApplog = 'Y' -- write notification to APPLOG when job fails Y/N
SET @CreateSubfolders = 'N' -- Create subfolders for backups ex.: {BackupPath}\User\Translog...if not all backups are in root backup folder
SET @BackupTimeStampSuffix = 'Y' -- Affect backup file name, when set to Y then file name includes actual timestamp ex.: ..._20181126_133840.trn
SET @RunJobs = 'N' -- Run some jobs after deployment Y/N
SET @RemoveJobs = 'N' -- Remove maintenance jobs Y/N
SET @OptimizeTempdb = 'N' -- Optimize tempdb database Y/N
SET @AddStartupParameters = 'N' -- Add optimizing startup parameters Y/N (traceflags 1117 and 1118)
SET @SharePointInstance = 'N' -- Is this instance hosting SharePoint databases (chnages in backups) Y/N
SET @BizTalkInstance = 'N' -- Is this instance hosting BizTalk databases (chnages in backups) Y/N
SET @DelayMinutes = 15 -- count of minutes which will be used in delays between instances installed on the server
```

**You can also change script body if there is something that you do not like there, but only by your own responsibility!**

You can deploy to single server or to more servers using registered servers, where script will be executed in parallel on all registered instances (never mind if running on one or more servers).

One important thing is that script is creating delayed schedules for more instance running on same machine. Because of reducing system resources load during run at same time on all instances. You can set delay by parameter **@DelayMinutes**. Then every installed instance is given index based on value in registry key **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server** under value of key name **IntalledInstances**.

### 3.1 Exceptions in environments	
Parameters: **@SharePointInstance, @BizTalkInstance**

Some application specific SQL environments requires some exception in SQL maintenance, for example BizTalk related SQL databases are handled by backups from BizTalk application. Or SharePoint databases can be handled by 3rd party tool like DocAve. There are prepared switch parameters at the script beginning to modify maintenance accordingly. 
Now there are only two exceptions for mentioned BizTalk and SharePoint environments as described below.

If instance is marked as **@SharePointInstance** then only system databases are handled by this maintenance solution. User databases are handled by DocAve in this situation.

If instance is marked as **@BizTalkInstance** then exceptions for standard BizTalk databases are included in backup step of user databases. Then parameter in script is set as following 
```
@Databases = 'USER_DATABASES, -BAMPrimaryImport%, -BAMAlertsApplication%, -BAMAlertsNSMain%, -BizTalkDTADb%, -BizTalkMgmtDb%, -BizTalkMsgBoxDb%, -BizTalkRuleEngineDb%, -SSODB%, -BAMStarSchema%, -BAMAnalysis%, -BAMArchive%, -BizTalkAnalysisDb%'
```
This exclusion is done based on official BizTalk documentation https://msdn.microsoft.com/en-us/library/aa578342.aspx

### 3.2 Optimize TempDB
Parameters: **@OptimizeTempdb**
There are some optimization recommendations regarding TempDB performance. Based on MS best practices following TempDB configurations should be followed for optimal performance of TempDB database:
 * database files should be stored on separate disk drives to provide exclusive IO operations just for communication with this database on disk drives (this is not object of maintenance solution and should be done by design of environment)
 * database files should be divided into number of files equally to count of logical processors but maximally up to 8, then advantages of parallel accessing become disadvantage because of internal management of SQL engine
 * database auto growth and initial size should be the same on all files mentioned in previous step. We have chosen following values for our standard. Initial size is set to 512 MB  and file growth is set to 256 MB which are agreed values not causing any overload when performing auto growth of files and similar issues.
 
More info about TempDB optimization can be found on official MS web pages https://msdn.microsoft.com/en-us/library/ms190768.aspx.

### 3.3 Optimize model settings
The model database is used as the template for all databases created on an instance of SQL Server. Because tempdb is created every time SQL Server is started, the model database must always exist on a SQL Server system. The entire contents of the model database, including database options, are copied to the new database. Some of the settings of model are also used for creating a new tempdb during start up, so the model database must always exist on a SQL Server system.

We are changing model database configuration to more efficient and optimal values in our standardized solution. We are changing initial file size and file growth attributes of this database from default values to agreed values providing some performance improvements on newly created databases with default configuration in comparison to default values set after installation of SQL Server instance. Values change described in table below.	

| Attribute |	Default value	| Changed value |
|:---|:---|:---|
| Autogrowth for data file |	By 1 MB,  Unlimited	| By 128 MB,  Unlimited |
| Autogrowth for log file |	By 10 percent, Unlimited	| By 64 MB,  Unlimited |
| Initial size for data file |	5 MB	| 128 MB |
| Initial size for log file |	2 MB	| 64 MB |


For more info about model database see official MSDN article https://msdn.microsoft.com/en-us/library/ms186388.aspx.	



