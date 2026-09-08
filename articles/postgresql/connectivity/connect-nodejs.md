---
title: "Connect Node.js to Azure Database for PostgreSQL"
description: Connect a Node.js application to Azure Database for PostgreSQL flexible server with node-postgres, TLS, and a verifiable SQL query.
#customer intent: As a Node.js developer new to Azure, I want to connect to an Azure Database for PostgreSQL flexible server, so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 09/07/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.custom:
  - mvc
  - mode-api
  - devx-track-js
ms.devlang: javascript
---

# Quickstart: Use Node.js to connect and query data in Azure Database for PostgreSQL flexible server

Azure Database for PostgreSQL flexible server is a managed service for running, managing, and scaling PostgreSQL databases in the cloud. A Node.js application can connect to the service by using the `pg` package, also known as node-postgres.

In this quickstart, you create a local Node.js project, configure a Transport Layer Security (TLS) connection to an existing flexible server, run a non-mutating `SELECT` statement, and confirm success from the printed result. The steps create only local project and dependency files; they don't create or change Azure resources or database objects.

## Prerequisites

- An Azure account with an active subscription. [Create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing Azure Database for PostgreSQL flexible server that uses PostgreSQL password authentication. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your development computer. For a server with public access, [add a firewall rule for your client IP address](../network/how-to-networking-servers-deployed-public-access-add-firewall-rules.md). For a server with private access, run the sample from a resource that can reach the server in its virtual network.
- A current supported [Node.js LTS release](https://nodejs.org/en/download), which includes npm.
- A PEM file that contains the complete current set of trusted Azure root certificate authorities (CAs). Follow [Configure TLS connections](../security/security-tls-how-to-connect.md) to prepare and maintain the trusted root certificates.

## Create the Node.js project

Create a local project and install the unpinned `pg` package from npm.

1. Open a terminal and create a project directory:

   ```bash
   mkdir postgresql-nodejs
   cd postgresql-nodejs
   ```

1. Initialize the npm project:

   ```bash
   npm init --yes
   ```

1. Install node-postgres:

   ```bash
   npm install pg
   ```

   The command adds `pg` to the project dependencies. Continue after npm finishes without an error.

## Get the connection information

Get the endpoint and PostgreSQL credentials for the existing flexible server.

1. In the [Azure portal](https://portal.azure.com), open your Azure Database for PostgreSQL flexible server.
1. On the server **Overview** page, copy the **Endpoint** and **Administrator login** values.
1. Choose the database to query. You can use the default `postgres` database or another existing database.
1. Locate the administrator password that you set when you created the server. If you no longer have it, reset the password from the server in the Azure portal.

## Set the connection environment variables

Store the connection values outside the source file. Set `DBHOST` to the server endpoint, `DBUSER` to the administrator login, `DBPASSWORD` to the administrator password, `DBNAME` to the database name, and `DBROOTCERT` to the full path of your trusted Azure root CA PEM file.

### Bash

Run these commands in the terminal where you'll run the application:

```bash
export DBHOST="myserver.postgres.database.azure.com"
export DBNAME="postgres"
export DBUSER="myadmin"
export DBPASSWORD="<administrator-password>"
export DBROOTCERT="/path/to/azure-root-ca-bundle.pem"
```

### PowerShell

Run these commands in the PowerShell session where you'll run the application:

```powershell
$env:DBHOST = "myserver.postgres.database.azure.com"
$env:DBNAME = "postgres"
$env:DBUSER = "myadmin"
$env:DBPASSWORD = "<administrator-password>"
$env:DBROOTCERT = "C:\path\to\azure-root-ca-bundle.pem"
```

Keep passwords and certificate files out of source control. These environment variables apply only to the current terminal session.

## Create and run the Node.js application

The application creates one node-postgres `Client`, explicitly enables TLS certificate verification with the trusted root CA file, runs a deterministic `SELECT`, prints the returned value, and closes the client.

1. Create a file named `index.mjs` in the `postgresql-nodejs` directory, and add the following code:

   ```javascript
   import fs from 'node:fs';
   import pg from 'pg';

   const { Client } = pg;

   let client;
   let connected = false;

   try {
     client = new Client({
       host: process.env.DBHOST,
       port: 5432,
       database: process.env.DBNAME,
       user: process.env.DBUSER,
       password: process.env.DBPASSWORD,
       ssl: {
         ca: fs.readFileSync(process.env.DBROOTCERT, 'utf8'),
         rejectUnauthorized: true,
       },
     });

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
   ```

1. Run the application from the same terminal session where you set the environment variables:

   ```bash
   node index.mjs
   ```

1. Confirm that the application prints this exact result:

   ```output
   Connected to Azure Database for PostgreSQL
   ```

   If the application instead prints `Connection or query failed`, use the error details to check the endpoint, credentials, network access, environment variables, and trusted root CA file before you run it again.

## Use connection pooling for frequent queries

This quickstart uses a single `Client` for one query. For a web application or other software that makes frequent queries, use the `Pool` API included with `pg` to reuse connections; you don't need a different database library.

## Clean up resources

This quickstart creates only the local `postgresql-nodejs` directory and its files. Delete that directory when you no longer need the sample; deleting it also removes `index.mjs`, `node_modules`, `package.json`, and `package-lock.json`. Don't delete an existing or shared flexible server.

## Next step

> [!div class="nextstepaction"]
> [Configure TLS connections and certificate validation](../security/security-tls-how-to-connect.md)
