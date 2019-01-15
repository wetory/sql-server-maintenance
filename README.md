# SQL-Server-Maintenance
Standardized database maintenance for SQL Server, built over popular Ola Halengreen's procedures.

Official documentation of one-script maintenance solution. More detailed info within [documentation file](docs/SQL%20Server%20Maintenance%20-%20documentation.pdf).

Table of contents:
  * [Introduction](#introduction)

## Introduction

This document describes maintenance solution used for keeping SQL Server in good condition purposes as agreed maintenance standard used on all SQL Server environments which is deployed after SQL Server instance installation. By default it is configured on agreed schedules but can be manually customized after deployment to the instance directly in SQL Agent jobs.
All environment specific need can be meet by modifications to standard implementation, for more info regarding to customization of Ola Halengreen’s scripts can be found at official sites.
Maintenance can be deployed to SQL Server versions >= 2005 (SQL Server 2000 not supported but some solution is in long term plans to have one script maintenance solution as this one)

## Technical preview

Whole solution is mostly based on popular Ola Hallengren’s maintenance procedures (visit here for more details https://ola.hallengren.com/). Olla’s procedures have been modified so as to fit needs described in standard specified in our team. Script is creating maintenance jobs as its main objection. All tasks performed on the target SQL Server instance are listed below.

Script perform following stuff:
•	create OH procedures in master database
•	create and drop helper procedures in master database
•	create SQL Agent jobs (full backup, differential backup, log backup, integrity check and index optimize, recycle error log)
•	schedule SQL Agent jobs based on agreed standard times
•	rename maintenance job syspolicy_purge_history
•	optimize tempdb according to best practices (this is optional)
•	change initial size and Autogrowth on model and tempdb database based on our agreed standard

Some of above described actions are not executed automatically but can be fired by changing input parameters, for more info see section Deployment

All important info also included in script header, containing also some short update history.
