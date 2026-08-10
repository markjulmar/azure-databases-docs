---
title: Connect to PostgreSQL with Node.js Using pg
description: Connect a Node.js application to Azure Database for PostgreSQL with the pg client, explicit TLS, and a verification query.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/10/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: "javascript"
---

# Quickstart: Connect to Azure Database for PostgreSQL with Node.js

In this quickstart, you use pg (node-postgres) to connect a Node.js application to an existing Azure Database for PostgreSQL flexible server. You configure Transport Layer Security (TLS), run a `SELECT` query, and print the result to confirm the connection.

This article assumes that you're familiar with Node.js but new to Azure Database for PostgreSQL.

## Prerequisites

- [Node.js](https://nodejs.org/en/download) and npm installed on your development computer.
- An Azure Database for PostgreSQL flexible server. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your development computer to the server. Review [Azure Database for PostgreSQL networking](../network/how-to-networking.md) to configure the applicable firewall rule or private network access.
- The server endpoint, port, database name, user name, and valid PostgreSQL password. The default PostgreSQL port is `5432`.

Azure Database for PostgreSQL requires TLS for client connections. This quickstart enables TLS explicitly in the pg client configuration. For certificate and TLS details, see [Transport Layer Security in Azure Database for PostgreSQL](../security/security-tls.md).

## Create the Node.js project

Create a small Node.js project and install the pg package.

1. Open a terminal and create a project directory:

   ```bash
   mkdir postgres-nodejs-quickstart
   cd postgres-nodejs-quickstart
   ```

1. Initialize the project:

   ```bash
   npm init -y
   ```

   The command creates a *package.json* file in the project directory.

1. Install pg:

   ```bash
   npm install pg
   ```

   The command adds pg to the project's dependencies.

## Configure the PostgreSQL connection

Store the connection settings in environment variables so the application source doesn't contain your password.

1. Set the connection environment variables for your current terminal session.

   ### [Windows PowerShell](#tab/powershell)

   ```powershell
   $env:PGHOST="<server-name>.postgres.database.azure.com"
   $env:PGPORT="5432"
   $env:PGDATABASE="postgres"
   $env:PGUSER="<username>"
   $env:PGPASSWORD="<password>"
   ```

   ### [macOS/Linux](#tab/bash)

   ```bash
   export PGHOST="<server-name>.postgres.database.azure.com"
   export PGPORT="5432"
   export PGDATABASE="postgres"
   export PGUSER="<username>"
   export PGPASSWORD="<password>"
   ```

   ---

   Replace `<server-name>` with your server name, `<username>` with your PostgreSQL user name, and `<password>` with the user's password. Replace `postgres` if you want to connect to a different database.

1. Confirm that all five environment variables are set in the same terminal where you'll run the application. If a value is missing, set it before you continue.

## Add the connection and query code

The Node.js application creates a pg `Client` with explicit TLS certificate validation, opens the connection, and runs a verification query.

1. Create a file named *index.js* in the project directory.

1. Add the following code to *index.js*:

   ```javascript
   const { Client } = require("pg");

   const requiredVariables = [
     "PGHOST",
     "PGPORT",
     "PGDATABASE",
     "PGUSER",
     "PGPASSWORD",
   ];

   const missingVariables = requiredVariables.filter(
     (name) => !process.env[name]
   );

   if (missingVariables.length > 0) {
     throw new Error(
       `Set the following environment variables: ${missingVariables.join(", ")}`
     );
   }

   const client = new Client({
     host: process.env.PGHOST,
     port: Number(process.env.PGPORT),
     database: process.env.PGDATABASE,
     user: process.env.PGUSER,
     password: process.env.PGPASSWORD,
     ssl: {
       rejectUnauthorized: true,
     },
   });

   async function queryDatabase() {
     let connected = false;

     try {
       await client.connect();
       connected = true;
       console.log("Connection established.");

       const result = await client.query("SELECT 1 AS result;");
       console.log(`Query result: ${result.rows[0].result}`);
     } catch (error) {
       console.error("Connection or query failed:", error.message);
       process.exitCode = 1;
     } finally {
       if (connected) {
         await client.end();
       }
     }
   }

   queryDatabase();
   ```

1. Save *index.js*. Keep the terminal session with the connection environment variables open for the next section.

## Run the Node.js application

Run the application from the project directory to verify the TLS-protected database connection.

1. In the terminal where you set the environment variables, run:

   ```bash
   node index.js
   ```

1. Confirm that the terminal shows:

   ```output
   Connection established.
   Query result: 1
   ```

   If the connection or query fails, the application prints the error. Confirm that the environment variables are set and that your development computer has network access to the server, then run the command again.

> [!NOTE]
> The pg package also includes the `Pool` API for applications that reuse database connections. This quickstart uses one `Client` to keep the first connection workflow focused.

## Related content

- [Connection libraries in Azure Database for PostgreSQL](concepts-connection-libraries.md)
- [Networking in Azure Database for PostgreSQL](../network/how-to-networking.md)
- [Transport Layer Security in Azure Database for PostgreSQL](../security/security-tls.md)
- [Create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md)
