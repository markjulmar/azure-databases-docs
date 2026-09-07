---
title: "Quickstart: Connect to PostgreSQL with Node.js"
description: Connect a Node.js application to Azure Database for PostgreSQL with pg, explicit TLS certificate verification, and a test SELECT query.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 09/07/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: javascript
---

# Quickstart: Connect to Azure Database for PostgreSQL with Node.js

Azure Database for PostgreSQL flexible server is a managed PostgreSQL service. The `pg` package provides a Node.js client that you can use to connect an application to the service.

In this quickstart, you create a local Node.js project, connect it to an existing flexible server over Transport Layer Security (TLS), run a `SELECT` statement, and print the returned value. The output `Query result: 1` confirms that the connection and query succeeded.

## Prerequisites

- A Node.js release with npm. If you need to install Node.js, use the [Node.js download page](https://nodejs.org/en/download).
- An Azure Database for PostgreSQL flexible server that uses PostgreSQL authentication. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- The server's fully qualified domain name (FQDN), administrator login, password, and target database name.
- Network access to the server. For public access, [add your client IP address to a server-level firewall rule](../security/security-firewall-rules.md), and make sure your local network allows outbound TCP connections on port `5432`. Firewall access doesn't replace authentication with valid credentials.
- A PEM file that contains the trusted Azure root Certificate Authority (CA) certificates. Follow the instructions to [download and combine the root CA certificates](../security/security-tls-how-to-connect.md#download-and-convert-root-ca-certificates). Don't use an intermediate CA or an individual server certificate as the trusted root.

## Create the Node.js project

Create a local project and install the [`pg` client library](concepts-connection-libraries.md). The commands don't pin a package version.

1. Open a terminal and create a project directory.

   ```bash
   mkdir postgresql-nodejs
   cd postgresql-nodejs
   ```

1. Initialize the Node.js project.

   ```bash
   npm init -y
   ```

1. Install `pg`.

   ```bash
   npm install pg
   ```

   After the command finishes, the project directory contains `package.json` and the installed `pg` package.

## Configure the connection values

Set environment variables so that the application can read connection values without storing credentials in source code. Replace each example value with the value for your server, database, and local CA bundle.

### Bash

Run these commands in the Bash terminal where you'll run the application:

```bash
export PGHOST="<server-name>.postgres.database.azure.com"
export PGPORT="5432"
export PGDATABASE="<database-name>"
export PGUSER="<administrator-login>"
export PGPASSWORD="<administrator-password>"
export PGSSLROOTCERT="/path/to/azure-root-ca-bundle.pem"
```

### PowerShell

Run these commands in the PowerShell session where you'll run the application:

```powershell
$env:PGHOST="<server-name>.postgres.database.azure.com"
$env:PGPORT="5432"
$env:PGDATABASE="<database-name>"
$env:PGUSER="<administrator-login>"
$env:PGPASSWORD="<administrator-password>"
$env:PGSSLROOTCERT="C:\path\to\azure-root-ca-bundle.pem"
```

The `PGHOST` value must be the server FQDN, not an IP address. The `PGSSLROOTCERT` value must be the path to the PEM file that contains the trusted Azure root CA certificates.

## Connect and run a SELECT query

Create and run one complete application that opens a TLS connection, verifies the server certificate, queries the database, and closes the client.

1. Create a file named `index.mjs` in the `postgresql-nodejs` directory.

1. Add the following code to `index.mjs`:

   ```javascript
   import { readFileSync } from 'node:fs';
   import pg from 'pg';

   const { Client } = pg;
   const requiredVariables = [
     'PGHOST',
     'PGPORT',
     'PGDATABASE',
     'PGUSER',
     'PGPASSWORD',
     'PGSSLROOTCERT',
   ];

   let client;

   try {
     for (const variable of requiredVariables) {
       if (!process.env[variable]) {
         throw new Error(`Set the ${variable} environment variable.`);
       }
     }

     client = new Client({
       host: process.env.PGHOST,
       port: Number(process.env.PGPORT),
       database: process.env.PGDATABASE,
       user: process.env.PGUSER,
       password: process.env.PGPASSWORD,
       ssl: {
         ca: readFileSync(process.env.PGSSLROOTCERT, 'utf8'),
         rejectUnauthorized: true,
       },
     });

     await client.connect();
     const result = await client.query('SELECT 1 AS result');
     console.log(`Query result: ${result.rows[0].result}`);
   } catch (error) {
     console.error('Connection or query failed:', error.message);
     process.exitCode = 1;
   } finally {
     if (client) {
       await client.end();
     }
   }
   ```

   The `ssl` object explicitly enables TLS. The `ca` value supplies trusted root certificates, and `rejectUnauthorized: true` requires certificate verification.

1. Run the application from the `postgresql-nodejs` directory.

   ```bash
   node index.mjs
   ```

1. Confirm that the terminal shows this output:

   ```output
   Query result: 1
   ```

   If the application instead prints `Connection or query failed`, use the error message to check the environment-variable values, CA bundle path, credentials, and network access before you run it again.

## Consider connection pooling

This quickstart uses one `pg.Client` for one query. For an application that makes frequent queries, `pg` also provides `Pool` through `pg-pool`; see the [node-postgres pooling guidance](https://node-postgres.com/features/pooling). Keep client-side `pg` pooling separate from the built-in PgBouncer service.

## Clean up resources

Remove the local project and secret-bearing environment variables when you no longer need the sample. Delete Azure resources only if you created them for this quickstart and don't want to retain them.

1. Close the terminal session to clear its environment variables, or remove the variables from the current session.

1. Delete the `postgresql-nodejs` project directory.

1. If you created a disposable resource group for this exercise, delete the resource group:

   ```azurecli
   az group delete --name <resource-group> --yes
   ```

   To retain the resource group and delete only the disposable flexible server, run this command instead:

   ```azurecli
   az postgres flexible-server delete \
     --resource-group <resource-group> \
     --name <server-name> \
     --yes
   ```

## Next step

> [!div class="nextstepaction"]
> [Review recommended TLS configurations for Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md)
