<!-- Copyright 2024 Google LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License. -->

Cloud SQL PostgreSQL Upgrade Tool User Guide

Deployment Guide


Date: March , 2024


Authors: Rani Singh Siddhant Verma Piyush Sarraf
Reviewer: Swati Aggarwal
Prepared for: Google
Document Type: Deployment Guide


Contents 

                  
1. Overview of Deployment Guide and Tool	4
1.1 Objective of Deployment Guide	4
1.2 About the PostgreSQL Upgrade Tool	4
2. Tool Functionality	6
3. Prerequisites	7
3.1 Important highlights of the Tool	7
3.2 Complete information to run the tool	7
3.3 User Input Requirements	8
4. Installation Steps	8





1. Overview of Deployment Guide and Tool
1.1 Objective of Deployment Guide

This document is created to help users run the Cloud SQL Upgrade Tool for PostgreSQL MVU with E+ . It is a step-by-step guide to run the tool. 

1.2 About the PostgreSQL Upgrade Tool

This tool is developed to enable automated assessment and upgrades of PostgreSQL databases for PostgreSQL 9.6, 10, 11  to 14, 15  or above with Enterprise edition to Enterprise Plus.
The approach here is to do In-place major version upgrade with automated assessment that can fix known issues and guide during the overall upgrade process. 
It is basically a 2 step process first being Major Version Upgrade followed by Enterprise + upgrade.

This tool can be run by a user that has privileges to run postgresql checks and to be able to upgrade PostgreSQL databases.
It generates a PDF report with information related to the extensive sanity checks performed to evaluate the compatibility, data integrity for upgrading PostgreSQL 9.6 instance to 14 or 15 and to capture a baseline performance.

The  is a combination of Python Scripts and gcloud commands that runs sanity checks on PostgreSQL database and upgrades PostgreSQL 9.6 instance to 14 or 15 version.

Following are the sanity checks that are being performed:

Extension Compatibility Check - Most extensions work on the upgraded database major version, but some of them can hinder MVU upgrade.  Check installation of extensions in each and every Cloud SQL Databases, check  the extension feasibility with installed version to that of default version. Drop any extensions that are no longer supported in the target version.
LC_COLLATE compatibility Check- The character set for each database must be en_US.UTF8. If the LC_COLLATE value for the template and postgresql databases isn't en_US.UTF8, then the major version upgrade fails. If any database has a character set other than en_US.UTF8, then change the LC_COLLATE value to en_US.UTF8 before performing the upgrade.
Cloud SQL with active replication slot publisher Check - Cloud SQL for PostgreSQL does not support cross-version replication, which means that upgrading the primary instance while the instance is replicating to the read replicas is not possible. Before upgrading, either disable replication for each read replica or delete the read replicas. If Cloud SQL is the logical replication source, disable pg_logical extension replication and enable it again after the upgrade.
Unsupported/unknown Data Types - PostgreSQL Check - The PostgreSQL upgrade utility, pg_upgrade, does not support upgrading databases that contain table columns using the reg* OID-referencing system data types. Therefore, it is suggested to remove all uses of reg* data types, except for regclass, regrole, and regtype, before attempting an upgrade.
Max Locks per Transaction Check - The recommended max_connections value for the upgrade is (number of tables) < (max_connections)*(max_locks_per_transaction). However, if there are problems because of a high number of tables, the recommendation is to change this value to a suitable value based on the number of tables.
SQL Identifier Detection Check - Usage of 'sql_identifier' in all databases within PostgreSQL instance will give following exception during upgrade - “Please remove the following usages of 'sql_identifier' data types before attempting an upgrade: (database: db_name, relation: rel_name, attribute: attr_name).“  Remove these identifiers before proceeding with the upgrade. 
Tables with Object Identifiers Check - Usage of 'oid' in all databases within PostgreSQL instance will give following exception during upgrade - “Please remove the following usages of tables with OIDs before attempting an upgrade: (database: db_name, relation: rel_name).” Make sure to remove/alter tables with OIDs before proceeding with the upgrade.                                                                                                                                                


2. Tool Functionality

The CloudSQL upgrade tool is built to perform sanity checks on the PostgreSQL databases and generate reports with detected problems along with automated scripts to fix some of the issues. Once it passes all the sanity checks then Major Version Upgrade from PostgreSQL 9.6 to 14 and 15 version will be carried out followed by Enterprise plus upgrade.


This tool will need below packages to be installed.
Python Packages
wkhtmltopdf 
psycopg2-binary
Pdfkit
Jinja2
google-cloud-secret-manager
matplotlib
Input to the Framework are,
PostgreSQL DB Host
Instance Name
User id and password (Should have privileges to run postgresql sanity checks)	
Database
Database version to which you want to upgrade ( 14 or 15 )
Output Generate by the tool are,
#1 PDF file - assessment report
3 scripts to fix the identified issues for Extension compatibility before and after upgrade, Alter OID scripts. 


3. Prerequisites
3.1 Important highlights of the Tool

The tool runs on the following OS versions Debian 11, MacOS.
Reach out to your Google account teams for access to the binary file
To ensure that the tool can run sanity checks for you, verify that the server where it is being deployed has access to your PostgreSQL instance.

3.2 Complete information to run the tool

The tool can run on any supported  VM that conforms to prerequisites listed in Section 3.1,  and fulfills following prerequisites:
Connectivity to PostgreSQL instance
DB user with privileges to perform sanity checks and upgrade.
The tool initially requires <  200mb of space
This tool has been tested on the PostgreSQL 9.6 version.
This tool runs on the following Operating systems:
Debian 11 and above
MacOS

After a successful tool execution, a PDF report will be generated at the location:
 ./storage/reports/postgresql-report.pdf.
After successful tool execution, a script will be generated that consists of alter queries for the tables with OID:
./storage/scripts/beforeupgrade_alter_table_with_oid


3.3 User Input Requirements
The tool needs the below permissions to run the code and generate the PDF report. 
PostgreSQL admin user to run sanity checks on the database and to execute MVU and E+ Upgrade.
Or the user in the Postgre database should have below mentioned privileges to be able to proceed with an upgrade 
 		SELECT, INSERT, UPDATE, DELETE, TRUNCATE, REFERENCES, TRIGGER
 
4. Installation Steps
Debian - 

Request access - 

Request the account team to submit an access request for the binary file.



Install Wkhtmltopdf package: Install the wkhtmltopdf package using the following command -

sudo apt-get install wkhtmltopdf


Download files: Download the binary and config file, using the gsutil command below.

gsutil -m cp -r gs://cloudsqlupgrade/binaries/postgresql . && cd postgresql


Setup gcloud auth login: To execute gcloud commands, setup the auth login and project_id -

gcloud auth login
gcloud config set project <PROJECT_ID>
gcloud auth application-default login	


Follow the steps to install proxy server to be able to connect Cloud SQL PostgreSQL instance:
            
	https://cloud.google.com/sql/docs/postgres/connect-instance-auth-proxy#install-proxy 

Populate the config file: Add the respective values to the config.json file. 

{
    "projectId": "",
    "instanceId": "",
    "user": "",
    "database": "",
    "secretId": "",
    "password": "",
    "sourceVersion": "",
    "upgradeVersion": "",
    "machineType": "",
    "runAssessment": "",
    "cloneSqlInstance": "",
    "executeAlterOidScript": "",
    "enterpriseUpgrade": "",
    "enterprisePlusUpgrade": ""
}


Note: Some of parameter definition are as follows: 
projectId: Project ID where CloudSQL instance is hosted.
instanceId: Instance ID of CloudSQL instance.
user: database user with required permissions
database : CloudSQL instance database for connecting to
secretId: ID of file in Secret Manager where database user password is stored. 
Either provide the user password in secret manager(enter the secretID) or as a plain text in Password variable. Tool will pick either of these.
Password: database user password(in plain text). If you have provided secretId above then leave this blank.
sourceVersion : Version of current PostgresSQL instance(9.6 or 10 or 11). 
upgradeVersion : PostgreSQL version to upgrade CloudSQL instance to (14 or 15)
machineType: Machine type of CloudSQL Enterprise Plus Edition. Select required machine type from below image.
Ex: db-perf-optimized-N-2
runAssessment: [Yes/No], if choosen "Yes", will run assessment on the instance and also generate Grant Script
cloneSqlInstance: [Yes/No], if choosen "Yes", will make a clone of current instance and perform upgrade on that instance, else perform upgrade on current instance. 
executeAlterOidScript: [Yes/No], if choosen "Yes", will run the script that will alter the tables with OID. 
enterpriseUpgrade: [Yes/No], if choosen "Yes", will perform a Major Version Upgrade.
enterprisePlusUpgrade: [Yes/No], if choosen "Yes", will perform an upgrade to Enterprise Plus. 

Choose any machine type from below list : 




To Execute the upgrade script follow steps below - 

Add the execution permission to binary file:


chmod +x cloudsql_upgrade


Run the binary file:


./cloudsql_upgrade

[INFO]
Incase you get this warning while trying to run the binary - 
“MAC_binaries_postgresql_cloudsql_upgrade” cannot be opened because it is from an unidentified developer.



Verify logs generated at :

./storage/logs/app.log

Run the following command to start the CloudSQL Proxy Server


"Open a new session in the terminal and run the following command - "
./cloud-sql-proxy {connectionName}

Note : Open a new terminal and execute the command given above and let the session run.
Example command : "./cloud-sql-proxy cloudsql-testing:us-central1:postgres-test"

Go back to the first terminal and provide your Cloud SQL Auth Proxy Host:

Please provide your Cloud SQL Auth Proxy Host: 127.0.0.1

Note: By default the host is 127.0.0.1. Incase if it is different provide appropriately. 

Provide your Cloud SQL Auth Proxy Port:

Please provide your Cloud SQL Auth Proxy Port: 5432

Note: By default the port is 5432. Incase if it is different provide appropriately. 


Please make sure all the problems are resolved, which are detected during sanity checks:

Prompt: "Please check and resolve if any problems detected during assessment"
Confirm if all the issues are resolved and good to proceed with clone and upgrade?(yes/no) - 


	Now the script will create a clone or not based on the input provided in the config file.
 	Note : Creating a clone of the instance might take a few minutes.


As we will now use the cloned instance for further steps , go ahead and terminate the 2nd terminal which we used at step 11.2 for proxy connection to the main instance.
Now create a new proxy session for the cloned instance. 



Clone Done for postgres-test
************************************************************
Open a new session in the terminal and run the following command -
./cloud-sql-proxy cloudsql-testing:us-central1:postgres-test-clone

Note : Open a new terminal and execute the command given above and let the session run. Pick the cloned instance name from the output above.
Example command : "./cloud-sql-proxy cloudsql-testing:us-central1:postgres-test-clone"




Go back to terminal and provide your Cloud SQL Auth Proxy Host and Port:

Please provide your Cloud SQL Auth Proxy Host: 127.0.0.1

Note: By default the host is 127.0.0.1 . Incase if it is different provide appropriately. 

Please provide your Cloud SQL Auth Proxy Port: 5432

Note: By default the host is 127.0.0.1 and port is 5432. Incase if it is different provide appropriately. 





Please verify if the clone ran successfully or not.
“executeAlterOid” script will run or not, based on the value in the config file. 
Please verify that the upgrade ran successfully or not. 
After the successful upgrade please make sure the instance is ready for Enterprise Plus upgrade 

Confirm if the instance is ready for Enterprise Plus Upgrade?(yes/no) - 





