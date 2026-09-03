---
title: "Connect With Node.js to Azure Database for PostgreSQL"
description: Connect a Node.js application to Azure Database for PostgreSQL over TLS, run a SELECT query, and print the result by using node-postgres.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 09/03/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: "javascript"
---

# Quickstart: Connect and query with Node.js in Azure Database for PostgreSQL flexible server

In this quickstart, you use Node.js and the `pg` package to connect to an existing Azure Database for PostgreSQL flexible server over Transport Layer Security (TLS), run a `SELECT` query, and print `Connection test: 1` to confirm success.

Azure Database for PostgreSQL is a managed PostgreSQL service. This quickstart creates only a local Node.js project and doesn't create or change Azure resources.

## Prerequisites

- An Azure account with an active subscription. If you don't have one, [create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing Azure Database for PostgreSQL flexible server and database. If you need them, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- The server's fully qualified domain name (FQDN), database name, administrator user name, and password. Find these values on the server's **Overview** page in the Azure portal.
- Network access from your development environment to the server. For public access, add your client IP address to the server firewall rules. For private access, run the application from a client that can reach the server through its virtual network.
- A current Long Term Support (LTS) release of [Node.js](https://nodejs.org/en/download), which includes npm.
- A PEM file that contains the recommended trusted root certificate authorities for Azure Database for PostgreSQL. Follow the [Azure Database for PostgreSQL TLS guidance](../security/security-tls.md) to prepare the file. Don't use an intermediate certificate or an individual server certificate as a trusted root.

## Create the Node.js project

Create a local project and install the `pg` package as an application dependency.

1. Open a terminal and create a project directory:

   ```bash
   mkdir pg-quickstart
   cd pg-quickstart
   ```

1. Initialize the project:

   ```bash
   npm init -y
   ```

1. Install `pg`:

   ```bash
   npm install pg
   ```

   The project directory now contains the package manifest, package lock, and installed dependency.

## Configure the connection

Store the connection values in environment variables so that the application doesn't contain your password or environment-specific settings.

1. Set the variables for your platform. Replace each placeholder with the corresponding server value. Set `PGSSLROOTCERT` to the full path of the trusted root CA PEM file from the prerequisites.

   ### [Windows (PowerShell)](#tab/powershell)

   ```powershell
   $env:PGHOST="<server-name>.postgres.database.azure.com"
   $env:PGPORT="5432"
   $env:PGDATABASE="<database-name>"
   $env:PGUSER="<administrator-user-name>"
   $env:PGPASSWORD="<password>"
   $env:PGSSLROOTCERT="<full-path-to-root-ca-pem-file>"
   ```

   ### [macOS or Linux](#tab/bash)

   ```bash
   export PGHOST="<server-name>.postgres.database.azure.com"
   export PGPORT="5432"
   export PGDATABASE="<database-name>"
   export PGUSER="<administrator-user-name>"
   export PGPASSWORD="<password>"
   export PGSSLROOTCERT="<full-path-to-root-ca-pem-file>"
   ```

   ---

1. Keep the terminal open. The application uses these variables in the next section.

## Connect and run a query

Use a single `pg.Client` to open a TLS connection, run a parameter-free `SELECT`, read the returned row, and close the connection.

1. Create a file named `index.mjs` in the `pg-quickstart` directory, and add the following code:

   ```javascript
   import fs from 'node:fs';
   import { Client } from 'pg';

   const client = new Client({
     host: process.env.PGHOST,
     port: Number(process.env.PGPORT),
     database: process.env.PGDATABASE,
     user: process.env.PGUSER,
     password: process.env.PGPASSWORD,
     ssl: {
       ca: fs.readFileSync(process.env.PGSSLROOTCERT, 'utf8'),
       rejectUnauthorized: true,
     },
   });

   let connected = false;

   try {
     await client.connect();
     connected = true;

     const result = await client.query('SELECT 1 AS connection_test');
     console.log(`Connection test: ${result.rows[0].connection_test}`);
   } catch (error) {
     console.error('Connection or query failed:', error);
     process.exitCode = 1;
   } finally {
     if (connected) {
       await client.end();
     }
   }
   ```

   The `ssl` object supplies the trusted root CA material to Node.js TLS and keeps server-certificate verification enabled.

1. From the same terminal where you set the environment variables, run the application:

   ```bash
   node index.mjs
   ```

1. Confirm that the application prints the following result:

   ```output
   Connection test: 1
   ```

   If the connection fails, verify the FQDN, database name, credentials, root CA file path, and network access. Then run the application again.

## Clean up resources

This quickstart creates only the local `pg-quickstart` directory and its contents.

1. When you no longer need the sample, close the terminal and delete the `pg-quickstart` directory. This action removes `index.mjs`, `package.json`, `package-lock.json`, and the local `node_modules` directory. Keep the trusted root CA PEM file if other applications use it.

## Related content

- [Connection libraries for Azure Database for PostgreSQL](concepts-connection-libraries.md)
- [TLS in Azure Database for PostgreSQL](../security/security-tls.md)
- [Networking for Azure Database for PostgreSQL](../network/how-to-networking.md)
