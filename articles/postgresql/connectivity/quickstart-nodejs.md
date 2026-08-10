---
title: "Connect to Azure PostgreSQL with Node.js and pg"
description: Use Node.js and pg to make a TLS-secured connection to Azure Database for PostgreSQL, run a SELECT query, and verify the result.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/10/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: javascript
---

# Quickstart: Connect to Azure Database for PostgreSQL with Node.js

Azure Database for PostgreSQL flexible server provides managed PostgreSQL databases for applications that need a PostgreSQL-compatible data store.

In this quickstart, you use the `pg` (node-postgres) client to make a TLS-secured connection from Node.js to an existing flexible server. You run a `SELECT` statement and print its returned result to confirm the connection.

## Prerequisites

- [Node.js](https://nodejs.org/en/download) and npm installed on your computer.
- An existing Azure Database for PostgreSQL flexible server and database. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- The server host name, database name, administrator user name, and password.
- Network access from your computer to the server. This quickstart assumes a server that uses **Public access (allowed IP addresses)** and has a firewall rule for your current client IP address. For other configurations, see [Azure Database for PostgreSQL networking](../network/how-to-networking.md).

## Create a Node.js project

Create a minimal Node.js project and install the `pg` package from npm.

1. Open a terminal and create a directory named *postgresql-node-quickstart*:

   ```bash
   mkdir postgresql-node-quickstart
   cd postgresql-node-quickstart
   ```

1. Create a default *package.json* file:

   ```bash
   npm init -y
   ```

1. Install `pg`:

   ```bash
   npm install pg
   ```

   When installation succeeds, npm adds `pg` to the `dependencies` section of *package.json*.

## Configure TLS and connection values

Store connection settings outside the application source and configure `pg` to validate the server certificate.

1. In the [Azure portal](https://portal.azure.com), open your flexible server. On the **Overview** page, copy the **Server name** and **Server admin login name**. Use the database you created, or use the default `postgres` database. Connections typically use port `5432`.

1. Follow [Install trusted root Certificate Authorities](../security/security-tls-how-to-connect.md#install-trusted-root-certificate-authorities-cas) to download and combine the Microsoft RSA Root CA 2017 and DigiCert Global Root G2 certificates into one PEM file. Save the combined file as *postgres-ca.pem*.

1. Set environment variables in the same terminal where you'll run the application. Replace the sample server name, user name, password, and CA certificate path with your values.

   **Bash**

   ```bash
   export AZURE_POSTGRESQL_HOST="myserver.postgres.database.azure.com"
   export AZURE_POSTGRESQL_DATABASE="postgres"
   export AZURE_POSTGRESQL_USER="myadmin"
   export AZURE_POSTGRESQL_PASSWORD="your-password"
   export AZURE_POSTGRESQL_PORT="5432"
   export AZURE_POSTGRESQL_CA_CERT_PATH="/absolute/path/postgres-ca.pem"
   ```

   **PowerShell**

   ```powershell
   $env:AZURE_POSTGRESQL_HOST="myserver.postgres.database.azure.com"
   $env:AZURE_POSTGRESQL_DATABASE="postgres"
   $env:AZURE_POSTGRESQL_USER="myadmin"
   $env:AZURE_POSTGRESQL_PASSWORD="your-password"
   $env:AZURE_POSTGRESQL_PORT="5432"
   $env:AZURE_POSTGRESQL_CA_CERT_PATH="C:\certificates\postgres-ca.pem"
   ```

   These environment variables apply to the current terminal session. Don't commit passwords or certificate files to source control.

## Connect and run a SELECT query

Use `pg.Client` for the connection, pass the CA bundle to the `ssl` option, and require certificate validation.

1. Create a file named *index.js* in the *postgresql-node-quickstart* directory.

1. Add the following code to *index.js*:

   ```javascript
   const fs = require('node:fs');
   const { Client } = require('pg');

   const requiredVariables = [
     'AZURE_POSTGRESQL_HOST',
     'AZURE_POSTGRESQL_DATABASE',
     'AZURE_POSTGRESQL_USER',
     'AZURE_POSTGRESQL_PASSWORD',
     'AZURE_POSTGRESQL_PORT',
     'AZURE_POSTGRESQL_CA_CERT_PATH',
   ];

   for (const variable of requiredVariables) {
     if (!process.env[variable]) {
       throw new Error(`Missing required environment variable: ${variable}`);
     }
   }

   const client = new Client({
     host: process.env.AZURE_POSTGRESQL_HOST,
     database: process.env.AZURE_POSTGRESQL_DATABASE,
     user: process.env.AZURE_POSTGRESQL_USER,
     password: process.env.AZURE_POSTGRESQL_PASSWORD,
     port: Number(process.env.AZURE_POSTGRESQL_PORT),
     ssl: {
       ca: fs.readFileSync(process.env.AZURE_POSTGRESQL_CA_CERT_PATH, 'utf8'),
       rejectUnauthorized: true,
     },
   });

   async function main() {
     let connected = false;

     try {
       await client.connect();
       connected = true;
       const result = await client.query(
         "SELECT 'Connection successful' AS status;"
       );
       console.log(result.rows[0]);
     } finally {
       if (connected) {
         await client.end();
       }
     }
   }

   main().catch((error) => {
     console.error(error);
     process.exitCode = 1;
   });
   ```

1. Run the application from the same terminal where you set the environment variables:

   ```bash
   node index.js
   ```

1. Confirm that the application prints the row returned by the `SELECT` statement:

   ```output
   { status: 'Connection successful' }
   ```

   The application closes the database connection after the query completes.

## Resolve connection errors

If the expected result doesn't appear, use the error message to check the corresponding connection setting.

1. For a timeout or connection refusal, confirm that `AZURE_POSTGRESQL_HOST` and `AZURE_POSTGRESQL_PORT` match the server values and that the firewall allows your current client IP address. Review [Azure Database for PostgreSQL networking](../network/how-to-networking.md), and then rerun `node index.js`.
1. For an authentication error, set `AZURE_POSTGRESQL_USER` and `AZURE_POSTGRESQL_PASSWORD` again with the server administrator credentials, and rerun `node index.js`.
1. For a host name or certificate validation error, confirm that `AZURE_POSTGRESQL_HOST` is the complete server name from the Azure portal and that `AZURE_POSTGRESQL_CA_CERT_PATH` points to the combined PEM file. Review [TLS certificate configuration](../security/security-tls-how-to-connect.md), and then rerun `node index.js`.

## Related content

- [Create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md)
- [Configure TLS connections to Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md)
- [PostgreSQL connectivity client libraries](concepts-connection-libraries.md)
