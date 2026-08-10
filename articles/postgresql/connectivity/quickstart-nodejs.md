---
title: "Quickstart: Connect Node.js Apps to PostgreSQL with pg"
description: Connect a Node.js application to Azure Database for PostgreSQL flexible server with pg, configure TLS, run a SELECT query, and print the result.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/10/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ms.devlang: javascript
ai-usage: ai-generated
---

# Quickstart: Connect to Azure Database for PostgreSQL flexible server with Node.js

In this quickstart, you use the pg (node-postgres) client to connect a Node.js application to an existing Azure Database for PostgreSQL flexible server. You configure Transport Layer Security (TLS) explicitly, run a `SELECT` statement, and print the returned value to confirm the connection.

## Prerequisites

Before you begin, make sure you have:

- An existing Azure Database for PostgreSQL flexible server. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- [Node.js with npm](https://nodejs.org/en/download) installed.
- Network access from the computer that runs the Node.js application:
  - For **Public access (allowed IP addresses)**, the server firewall must allow the computer's public IP address.
  - For **Private access (VNet Integration)**, run the application from a resource that can reach the server's virtual network.

For network configuration steps, see [Networking in Azure Database for PostgreSQL flexible server](../network/how-to-networking.md).

## Create the Node.js project

Initialize a minimal Node.js project and install pg from the project directory.

1. Create a directory named *postgresql-nodejs* and change to it.

   ```bash
   mkdir postgresql-nodejs
   cd postgresql-nodejs
   ```

1. Create the project's *package.json* file.

   ```bash
   npm init -y
   ```

1. Install the pg package.

   ```bash
   npm install pg
   ```

   After the command finishes, *package.json* lists pg as a dependency.

## Configure the connection values

Keep connection values and credentials outside source code by setting environment variables in the terminal where you'll run the application.

The sample uses these connection inputs:

- `DBHOST`: The fully qualified server endpoint, such as `myserver.postgres.database.azure.com`.
- `DBPORT`: The PostgreSQL port for the endpoint, typically `5432`.
- `DBNAME`: The database name, such as `postgres`.
- `DBUSER`: The PostgreSQL user name.
- `DBPASSWORD`: The PostgreSQL password. If your server and database user are configured for token authentication, use a current supported authentication token instead.

Choose the instructions for your terminal.

### Windows PowerShell

Set each environment variable after replacing the example values with the connection information for your server.

1. Run the following commands in PowerShell:

   ```powershell
   $env:DBHOST = "myserver.postgres.database.azure.com"
   $env:DBPORT = "5432"
   $env:DBNAME = "postgres"
   $env:DBUSER = "myadmin"
   $env:DBPASSWORD = "<password-or-token>"
   ```

1. Confirm that all five environment variables are set before continuing. The command reports whether each variable is set without printing its value.

   ```powershell
   "DBHOST", "DBPORT", "DBNAME", "DBUSER", "DBPASSWORD" |
     ForEach-Object { "{0}: {1}" -f $_, (Test-Path "Env:$_") }
   ```

   Confirm that the command prints `True` for every variable. If any variable is `False`, set it again in the current PowerShell session.

### macOS and Linux

Set each environment variable after replacing the example values with the connection information for your server.

1. Run the following commands in your shell:

   ```bash
   export DBHOST="myserver.postgres.database.azure.com"
   export DBPORT="5432"
   export DBNAME="postgres"
   export DBUSER="myadmin"
   export DBPASSWORD="<password-or-token>"
   ```

1. Confirm that all five environment variables are set before continuing. The command reports whether each variable is set without printing its value.

   ```bash
   for variable in DBHOST DBPORT DBNAME DBUSER DBPASSWORD; do
     if [ -n "$(printenv "$variable")" ]; then
       echo "$variable: set"
     else
       echo "$variable: not set"
     fi
   done
   ```

   Confirm that the command reports `set` for every variable. If any variable is `not set`, set it again in the current shell session.

## Add the connection code

Create one JavaScript file that opens a TLS-protected connection, runs a query, prints its result, reports failures, and closes the client.

1. Create a file named *index.js* in the *postgresql-nodejs* directory.

1. Add the following code to *index.js*:

   ```javascript
   const { Client } = require('pg');

   const client = new Client({
     host: process.env.DBHOST,
     port: Number(process.env.DBPORT),
     database: process.env.DBNAME,
     user: process.env.DBUSER,
     password: process.env.DBPASSWORD,
     ssl: { rejectUnauthorized: true },
   });

   async function main() {
     let connected = false;

     try {
       await client.connect();
       connected = true;

       const result = await client.query(
         "SELECT 'Connected to Azure Database for PostgreSQL' AS message"
       );
       console.log(result.rows[0].message);
     } catch (error) {
       console.error('Connection or query failed:', error.message);
       process.exitCode = 1;
     } finally {
       if (connected) {
         await client.end();
       }
     }
   }

   main();
   ```

   The `ssl` option explicitly enables TLS and validates the server certificate against the trusted certificates available to Node.js. For certificate-validation options and trusted root certificate setup, see [Configure TLS connections in Azure Database for PostgreSQL flexible server](../security/security-tls-how-to-connect.md).

## Run the query

Run the application from the same terminal session where you set the connection environment variables.

1. From the *postgresql-nodejs* directory, run the sample:

   ```bash
   node index.js
   ```

1. Confirm that the query returns this output:

   ```output
   Connected to Azure Database for PostgreSQL
   ```

   If the command reports a connection error, confirm that the environment variables contain the correct endpoint and credentials, then check the server's public firewall or private virtual-network reachability.

For applications that serve concurrent requests, pg also provides `Pool` for connection pooling. This quickstart uses one `Client` to keep the first connection focused.

## Related content

- [Connection libraries in Azure Database for PostgreSQL flexible server](concepts-connection-libraries.md)
- [Configure TLS connections in Azure Database for PostgreSQL flexible server](../security/security-tls-how-to-connect.md)
- [Create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md)
- [Networking in Azure Database for PostgreSQL flexible server](../network/how-to-networking.md)
