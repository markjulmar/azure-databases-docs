---
title: "Quickstart: Connect and Query Azure Database for PostgreSQL with Node.js"
description: Connect to Azure Database for PostgreSQL from Node.js with the pg client and verify the connection with a TLS-protected SQL query.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/07/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.custom:
  - mvc
  - devx-track-js
  - mode-api
  - sfi-image-nochange
ms.devlang: javascript
---

# Quickstart: Connect and query Azure Database for PostgreSQL with Node.js

In this quickstart, you use the `pg` (node-postgres) client to connect a Node.js application to an Azure Database for PostgreSQL flexible server. You configure a Transport Layer Security (TLS) connection, run a simple `SELECT` statement, print the result, and close the connection.

This quickstart is for developers who are familiar with Node.js but are connecting to Azure Database for PostgreSQL for the first time.

## Prerequisites

To complete this quickstart, you need:

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing [Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your workstation to the server. For public access, add your workstation's IP address to the server firewall rules. For private access, run the sample from a resource that can reach the server's virtual network. For more information, see [Networking in Azure Database for PostgreSQL](../network/how-to-networking.md).
- A supported version of [Node.js and npm](https://nodejs.org/en/download).

## Get the PostgreSQL connection information

Get the server endpoint and administrator sign-in from the Azure portal. You'll store these values outside the application source code in a later section.

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Select **All resources**, and then search for your Azure Database for PostgreSQL flexible server.
1. Select the server.
1. On the **Overview** page, copy the **Endpoint** and **Administrator login** values.
1. Use `postgres` for the database name, or use the name of another existing database on the server.
1. Use the administrator password that you set when you created the server. If you don't remember it, select **Reset password**, set a new password, and wait for the update to finish before continuing.

## Create the Node.js project and install pg

Create a minimal Node.js project and add the `pg` package.

1. Open a terminal and create a project folder:

   ```bash
   mkdir postgres-nodejs-quickstart
   cd postgres-nodejs-quickstart
   ```

1. Initialize the project and install `pg`:

   ```bash
   npm init -y
   npm install pg
   ```

## Configure the connection and TLS

Store the connection values in environment variables so that the application doesn't contain credentials. The sample uses a separate `ssl` configuration object and doesn't use SSL parameters such as `sslmode`, `sslcert`, `sslkey`, or `sslrootcert` in a connection string. In node-postgres, those connection-string parameters replace a separate `ssl` object and discard its extra options.

1. Set the variables to the values from the Azure portal. Replace each sample value before you run the application.

   ### [Windows](#tab/windows)

   ```cmd
   set PGHOST=<server-endpoint>
   set PGDATABASE=postgres
   set PGUSER=<administrator-login>
   set PGPASSWORD=<administrator-password>
   ```

   ### [macOS/Linux](#tab/macos-linux)

   ```bash
   export PGHOST=<server-endpoint>
   export PGDATABASE=postgres
   export PGUSER=<administrator-login>
   export PGPASSWORD=<administrator-password>
   ```

   ---

1. Keep the terminal open. These variables apply only to the current terminal session. Don't add the password or a complete connection string to source control.

The sample explicitly enables TLS certificate validation with `rejectUnauthorized: true`. For root certificate setup, regional certificate details, and advanced validation options, see [Configure TLS connections in Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md).

## Connect and run a SELECT query

Create a `pg.Client`, open the TLS connection, and run `SELECT 1`. The `finally` block ends the client after either success or failure.

1. Create a file named *index.js* in the project folder.
1. Add the following code to *index.js*:

   ```javascript
   const { Client } = require("pg");

   const requiredVariables = ["PGHOST", "PGDATABASE", "PGUSER", "PGPASSWORD"];
   const missingVariables = requiredVariables.filter(
     (name) => !process.env[name]
   );

   if (missingVariables.length > 0) {
     console.error(
       `Set the following environment variables: ${missingVariables.join(", ")}`
     );
     process.exit(1);
   }

   const client = new Client({
     host: process.env.PGHOST,
     port: 5432,
     database: process.env.PGDATABASE,
     user: process.env.PGUSER,
     password: process.env.PGPASSWORD,
     ssl: {
       rejectUnauthorized: true,
     },
   });

   async function main() {
     try {
       await client.connect();

       const result = await client.query("SELECT 1 AS connected");
       console.log(`Connected: ${result.rows[0].connected}`);
     } catch (error) {
       console.error("Connection or query failed:", error.message);
       process.exitCode = 1;
     } finally {
       try {
         await client.end();
       } catch (error) {
         console.error("Client cleanup failed:", error.message);
         process.exitCode = 1;
       }
     }
   }

   main();
   ```

1. In the same terminal where you set the environment variables, run the sample:

   ```bash
   node index.js
   ```

1. Confirm that the command prints the following result:

   ```output
   Connected: 1
   ```

If the connection fails, check the error message, confirm that all four environment variables contain the portal values, and verify that the workstation can reach the server through its firewall or private network.

## Clean up resources

The sample closes the database client after the query. Remove local values and resources that you no longer need.

1. Close the terminal to clear the environment variables from the current session.
1. Delete the *postgres-nodejs-quickstart* folder if you no longer need the local sample.
1. If you created the flexible server only for this quickstart, follow [Delete an Azure Database for PostgreSQL flexible server](../configure-maintain/how-to-delete-server.md).

## Related content

- [Connection libraries in Azure Database for PostgreSQL](concepts-connection-libraries.md)
- [Configure TLS connections in Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md)
- [Networking in Azure Database for PostgreSQL](../network/how-to-networking.md)
