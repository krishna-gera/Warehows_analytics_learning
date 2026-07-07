# Technical Report: Steps Performed During PostgreSQL to BigQuery Migration

## Overview

This document summarizes the complete workflow followed to migrate the
**Northwind PostgreSQL database** to **Google Cloud BigQuery** using the
**BigQuery Data Transfer Service (DTS)**. It also includes the issues
encountered during implementation and the corresponding solutions.

------------------------------------------------------------------------

# Step 1: Install PostgreSQL

-   Installed PostgreSQL 17 using Homebrew.
-   Started the PostgreSQL service.
-   Verified that the PostgreSQL server was running successfully.

------------------------------------------------------------------------

# Step 2: Create Database and User

-   Connected to PostgreSQL.
-   Created a dedicated database user (`dts_user`).
-   Created the **Northwind** database.
-   Assigned ownership of the database to the newly created user.

------------------------------------------------------------------------

# Step 3: Import the Northwind Dataset

-   Imported the provided `northwind.sql` file.
-   Created all required tables.
-   Loaded the sample Northwind data into PostgreSQL.

------------------------------------------------------------------------

# Step 4: Verify Local Database

Verified that the import was successful by:

-   Listing all database tables (`\dt`)
-   Checking row counts for major tables

### Verified Data

| Table      | Row Count |
|------------|----------:|
| Customers  |        91 |
| Orders     |       830 |
| Products   |        77 |

------------------------------------------------------------------------

# Step 5: Expose the Local Database

Since Google Cloud cannot directly access a database running on
`localhost`, the local PostgreSQL instance was exposed using **Ngrok**.

### Actions Performed

-   Installed Ngrok.
-   Started a TCP tunnel on PostgreSQL port **5432**.
-   Obtained a public host and port for secure external access.

------------------------------------------------------------------------

# Step 6: Create BigQuery Dataset

Inside Google Cloud:

-   Opened BigQuery Studio.
-   Created a new dataset named **northwind_sync**.
-   Selected the appropriate region for deployment.

------------------------------------------------------------------------

# Step 7: Configure BigQuery Data Transfer Service (DTS)

Created a PostgreSQL Data Transfer job with the following configuration:

-   Ngrok Host
-   Ngrok Port
-   Database Name
-   Username
-   Password
-   Tables to Transfer
-   Daily Schedule

Configured the ingestion mode as **Full Refresh**.

------------------------------------------------------------------------

# Issues Encountered and Solutions

## Issue 1: Google Cloud Could Not Reach Local PostgreSQL

**Cause**

The PostgreSQL server was only accessible through `localhost`.

**Solution**

Created an Ngrok TCP tunnel to expose the local PostgreSQL server over
the internet.

------------------------------------------------------------------------

## Issue 2: "The server does not support SSL"

**Cause**

BigQuery attempted to establish an SSL connection while the local
PostgreSQL server was configured without SSL support.

**Solution**

-   Disabled SSL in PostgreSQL.
-   Restarted PostgreSQL.
-   Configured BigQuery DTS to use **No Encryption / No Verification**.

------------------------------------------------------------------------

## Issue 3: PostgreSQL User Authentication Error

**Cause**

The default `postgres` role was unavailable because PostgreSQL had been
installed using Homebrew.

**Solution**

Created a dedicated database user (`dts_user`) and used those
credentials in the transfer configuration.

------------------------------------------------------------------------

## Issue 4: Invalid Table Object Format

**Cause**

BigQuery could not parse the supplied PostgreSQL object list.

**Solution**

Specified each table using the fully qualified format:

``` text
northwind/public/customers
northwind/public/orders
northwind/public/products
```

------------------------------------------------------------------------

## Issue 5: Empty Tables After Transfer

**Cause**

The source database initially contained no records.

**Solution**

Verified the SQL import and confirmed that all required data had been
loaded before executing the transfer again.

------------------------------------------------------------------------

## Issue 6: DTS Configuration Validation Errors

**Cause**

Some fields became temporarily locked or invalid during transfer
configuration.

**Solution**

Reconfigured the transfer settings and verified that all required fields
were correctly populated.

------------------------------------------------------------------------

# Step 8: Execute the Transfer

-   Started the transfer manually.
-   Waited for successful execution.
-   Confirmed that all selected tables were created in BigQuery.

------------------------------------------------------------------------

# Step 9: Validate Data Integrity

Compared row counts between PostgreSQL and BigQuery.

 | Table     | PostgreSQL | BigQuery | Status   |
|-----------|-----------:|---------:|----------|
| Customers |         91 |       91 | ✅ Match |
| Orders    |        830 |      830 | ✅ Match |
| Products  |         77 |       77 | ✅ Match |

------------------------------------------------------------------------

# Step 10: Verify Analytical Queries

Executed SQL queries in BigQuery to:

-   Join Customers and Orders.
-   Aggregate freight values.
-   Verify relational integrity.
-   Confirm that migrated data matched the source database.

The results matched the PostgreSQL database.

------------------------------------------------------------------------

# Final Outcome

The assignment was successfully completed by:

-   Installing and configuring PostgreSQL locally.
-   Importing the Northwind sample database.
-   Exposing the local database securely using Ngrok.
-   Creating a BigQuery dataset.
-   Configuring the BigQuery Data Transfer Service.
-   Resolving networking, SSL, authentication, and schema configuration
    issues.
-   Successfully migrating PostgreSQL data to BigQuery.
-   Validating data accuracy using row counts and SQL queries.

The final pipeline provides an automated PostgreSQL-to-BigQuery ETL
workflow capable of performing scheduled data synchronization.
