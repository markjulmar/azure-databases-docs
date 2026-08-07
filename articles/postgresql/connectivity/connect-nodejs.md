---
title: Connect Node.js to Azure Database for PostgreSQL
description: Connect to Azure Database for PostgreSQL from Node.js, protect the connection with TLS, run a SELECT query, and verify the result.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/06/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.custom:
  - devx-track-js
ms.devlang: "javascript"
---

# Quickstart: Connect and query Azure Database for PostgreSQL with Node.js

Azure Database for PostgreSQL flexible server is a managed PostgreSQL service that Node.js applications can access with the `pg` client. This quickstart is for developers who are familiar with Node.js but are new to Azure Database for PostgreSQL.

In this quickstart, you create a minimal Node.js project, connect to an existing flexible server over a Transport Layer Security (TLS) connection, run a `SELECT` query, print the result, and close the connection.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing Azure Database for PostgreSQL flexible server and a database on that server. If you need a server, see [Create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- A network path from your development computer to the server. For public access, add your current client IP address to the server's firewall rules. For private access, run the sample from a resource that can reach the server's virtual network. For more information, see [Networking in Azure Database for PostgreSQL flexible server](../network/how-to-networking.md).
- A currently supported [Node.js](https://nodejs.org/en/download) release and `npm`.
- The server administrator login and password, or credentials for another PostgreSQL role that can connect to the database.
- A local PEM file that contains the trusted root certificates required for Azure Database for PostgreSQL. Follow [Configure TLS connections and install trusted root certificates](../security/security-tls-how-to-connect.md#install-trusted-root-certificate-authorities-cas), and note the file's full path.

## Create the Node.js project

Create a minimal Node.js project and install the `pg` package.

1. Open a terminal and create a project directory:

   ```bash
   mkdir postgres-node-quickstart
   cd postgres-node-quickstart
   ```

1. Initialize the project:

   ```bash
   npm init -y
   ```

1. Install the `pg` package:

   ```bash
   npm install pg
   ```

   Confirm that `npm` adds `pg` to the `dependencies` section of *package.json*. If installation fails, resolve the reported `npm` or network error before you continue.

## Get the PostgreSQL connection values

Get the server endpoint and administrator login from your flexible server resource in the Azure portal.

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Search for and select your Azure Database for PostgreSQL flexible server.
1. On the server's **Overview** page, copy the **Endpoint** and **Administrator login** values.
1. Identify the database to query. You can use the default `postgres` database if you haven't created another database.
1. Use the administrator password you set when you created the server. If you no longer have it, select **Reset password** on the server's **Overview** page and set a new password before you continue.

Keep the **Endpoint**, **Administrator login**, database name, and password available for the next section.

## Configure the connection environment

Store the connection values and trusted root certificate path in environment variables instead of putting credentials in source code. Set `PGHOST` to the **Endpoint** value and `PGUSER` to the **Administrator login** value you copied from the portal.

### [Windows](#tab/windows)

1. In Command Prompt, set the connection environment variables:

   ```cmd
   set "PGHOST=<server-endpoint>"
   set "PGPORT=5432"
   set "PGDATABASE=<database-name>"
   set "PGUSER=<administrator-login>"
   set "PGPASSWORD=<administrator-password>"
   set "PGSSLROOTCERT=<full-path-to-combined-ca-pem-file>"
   ```

1. Replace each placeholder:

   - `<server-endpoint>`: The **Endpoint** value from the server's **Overview** page, such as `myserver.postgres.database.azure.com`.
   - `<database-name>`: The database to query, such as `postgres`.
   - `<administrator-login>`: The **Administrator login** value from the server's **Overview** page.
   - `<administrator-password>`: The password for the administrator login.
   - `<full-path-to-combined-ca-pem-file>`: The full path to the PEM file you prepared in the prerequisites.

### [macOS/Linux](#tab/macos-linux)

1. In a terminal, set the connection environment variables:

   ```bash
   export PGHOST="<server-endpoint>"
   export PGPORT="5432"
   export PGDATABASE="<database-name>"
   export PGUSER="<administrator-login>"
   export PGPASSWORD="<administrator-password>"
   export PGSSLROOTCERT="<full-path-to-combined-ca-pem-file>"
   ```

1. Replace each placeholder:

   - `<server-endpoint>`: The **Endpoint** value from the server's **Overview** page, such as `myserver.postgres.database.azure.com`.
   - `<database-name>`: The database to query, such as `postgres`.
   - `<administrator-login>`: The **Administrator login** value from the server's **Overview** page.
   - `<administrator-password>`: The password for the administrator login.
   - `<full-path-to-combined-ca-pem-file>`: The full path to the PEM file you prepared in the prerequisites.

---

The environment variables apply only to the current terminal session. Keep that terminal open for the remaining steps.

## Add the TLS connection and SELECT query

Create a `pg` client that validates the server certificate against your trusted root certificate file. The query returns the connected database and role without creating or changing database objects.

1. In the project directory, create a file named *index.js*.
1. Add the following code to *index.js*:

   ```javascript
   const fs = require('node:fs');
   const { Client } = require('pg');

   const requiredVariables = [
     'PGHOST',
     'PGPORT',
     'PGDATABASE',
     'PGUSER',
     'PGPASSWORD',
     'PGSSLROOTCERT',
   ];

   for (const variable of requiredVariables) {
     if (!process.env[variable]) {
       throw new Error(`Set the ${variable} environment variable before running the sample.`);
     }
   }

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

   async function queryDatabase() {
     let connected = false;

     try {
       await client.connect();
       connected = true;

       const result = await client.query(
         'SELECT current_database() AS database, current_user AS user'
       );
       console.log('Connected with TLS:', result.rows[0]);
     } finally {
       if (connected) {
         await client.end();
       }
     }
   }

   queryDatabase().catch((error) => {
     console.error('Database connection failed:', error.message);
     process.exitCode = 1;
   });
   ```

1. Save *index.js*. The `finally` block closes the client after the query, including when the query returns an error.

## Run the Node.js application

Run the application from the same terminal session where you set the connection environment variables.

1. From the *postgres-node-quickstart* project directory, run:

   ```bash
   node index.js
   ```

1. Confirm that the output contains the database and PostgreSQL role you configured:

   ```output
   Connected with TLS: { database: 'postgres', user: '<administrator-login>' }
   ```

   Your database and user values can differ. If the connection fails, use the error message to check the corresponding prerequisite:

   - A host lookup or timeout error indicates that you should verify `PGHOST` and the server's network or firewall configuration.
   - An authentication error indicates that you should verify `PGUSER`, `PGPASSWORD`, and the role's access to `PGDATABASE`.
   - A certificate error indicates that you should verify `PGSSLROOTCERT` and update the PEM file by following the linked TLS guidance.

## Clean up resources

The sample closes the database client in its `finally` block. Delete local or Azure resources only if you no longer need them.

1. Delete the *postgres-node-quickstart* project directory to remove the local sample and installed packages.
1. If you created the flexible server or its resource group only for this quickstart, delete that server or resource group in the Azure portal. Don't delete shared resources.

## Related content

- [Configure TLS connections in Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md)
- [Manage Azure Database for PostgreSQL by using the Azure portal](../configure-maintain/how-to-manage-server-portal.md)
- [PgBouncer in Azure Database for PostgreSQL](concepts-pgbouncer.md)
- [Quickstart: Use Python to connect and query data](connect-python.md)
- [Quickstart: Use Java and JDBC](connect-java.md)
- [Quickstart: Use .NET (C#) to connect and query data](connect-csharp.md)
