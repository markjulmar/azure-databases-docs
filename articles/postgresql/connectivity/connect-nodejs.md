---
title: "Connect Node.js Apps to PostgreSQL with TLS"
description: Connect a Node.js application to Azure Database for PostgreSQL over TLS, run a SELECT query, and verify the returned result.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/09/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: javascript
---

# Quickstart: Connect Node.js to Azure Database for PostgreSQL

In this quickstart, you connect a Node.js application to an Azure Database for PostgreSQL flexible server over Transport Layer Security (TLS). You then run a `SELECT` query and print its result to confirm that the application can reach and query the database.

## Prerequisites

- An Azure account with an active subscription. [Create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure Database for PostgreSQL flexible server. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your client to the server:
  - For **Public access (allowed IP addresses)**, add your client's public IP address to the server firewall rules.
  - For **Private access (VNet Integration)**, run the sample from a client that can connect through the server's virtual network.
- [Node.js](https://nodejs.org/en/download) and npm.

## Set up the Node.js project

Create a local Node.js project and install `pg` (node-postgres) as the only PostgreSQL client package.

1. In a terminal, create and open a project directory:

   ```bash
   mkdir postgresql-node-quickstart
   cd postgresql-node-quickstart
   ```

1. Initialize the project and install `pg`:

   ```bash
   npm init -y
   npm install pg
   ```

## Get the PostgreSQL connection information

The Node.js sample needs the server endpoint, administrator login, database name, password, and PostgreSQL port.

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Search for and select your Azure Database for PostgreSQL flexible server.
1. On the server **Overview** page, copy the **Endpoint** and **Administrator login** values.
1. Use `postgres` as the database name unless you created another database.
1. Use `5432` as the `DBPORT` value.
1. Supply the administrator password you set when you created the server. If you don't know it, select **Reset password** on the server **Overview** page. Don't put the password in your source file.

## Configure the connection environment variables

Set the connection values in the current terminal session so the Node.js source file doesn't contain credentials.

1. Set the environment variables for your shell. Replace the placeholder values with the endpoint, administrator login, database name, and password from the preceding section.

   ### [Windows PowerShell](#tab/powershell)

   ```powershell
   $env:DBHOST = "<server-name>.postgres.database.azure.com"
   $env:DBPORT = "5432"
   $env:DBNAME = "postgres"
   $env:DBUSER = "<administrator-login>"
   $env:DBPASSWORD = "<password>"
   ```

   ### [macOS/Linux](#tab/bash)

   ```bash
   export DBHOST="<server-name>.postgres.database.azure.com"
   export DBPORT="5432"
   export DBNAME="postgres"
   export DBUSER="<administrator-login>"
   export DBPASSWORD="<password>"
   ```

   ---

1. Confirm that `DBHOST`, `DBPORT`, `DBNAME`, and `DBUSER` contain the expected values. Don't print `DBPASSWORD`.

   ### [Windows PowerShell](#tab/powershell)

   ```powershell
   Write-Output $env:DBHOST, $env:DBPORT, $env:DBNAME, $env:DBUSER
   ```

   ### [macOS/Linux](#tab/bash)

   ```bash
   printf '%s\n' "$DBHOST" "$DBPORT" "$DBNAME" "$DBUSER"
   ```

   ---

## Create the Node.js sample

The sample uses an object-form `pg.Client` configuration. Setting `ssl` to `true` requires TLS and, with `pg` 8 and later, validates the server certificate by using Node.js trusted certificate authorities.

1. In the project directory, create *index.js* with the following code:

   ```javascript
   const { Client } = require('pg');

   const requiredVariables = [
     'DBHOST',
     'DBPORT',
     'DBNAME',
     'DBUSER',
     'DBPASSWORD',
   ];

   const missingVariables = requiredVariables.filter(
     (name) => !process.env[name]
   );

   if (missingVariables.length > 0) {
     console.error(
       `Missing required environment variables: ${missingVariables.join(', ')}`
     );
     process.exit(1);
   }

   const client = new Client({
     host: process.env.DBHOST,
     port: Number(process.env.DBPORT),
     database: process.env.DBNAME,
     user: process.env.DBUSER,
     password: process.env.DBPASSWORD,
     ssl: true,
   });

   async function run() {
     let connected = false;

     try {
       await client.connect();
       connected = true;

       const result = await client.query(
         "SELECT 'Connected to Azure Database for PostgreSQL' AS message"
       );
       console.log(result.rows[0].message);
     } catch (error) {
       console.error('PostgreSQL operation failed:', error.message);
       process.exitCode = 1;
     } finally {
       if (connected) {
         await client.end();
       }
     }
   }

   run();
   ```

If your environment uses a custom certificate store or reports a certificate-chain validation error, follow [Configure TLS for Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md) to install the required root certificates. Don't disable certificate validation with `rejectUnauthorized: false`.

## Run the SELECT query

Run the Node.js application from the project directory and check the printed query result.

1. From the project directory, run the sample:

   ```bash
   node index.js
   ```

1. Confirm that the console prints the query result:

   ```output
   Connected to Azure Database for PostgreSQL
   ```

   This output confirms that `pg` established the TLS connection and returned the `SELECT` result. If the operation fails, use the visible error message to check the environment-variable values, firewall or private-network path, credentials, and trusted root certificates before you retry.

## Clean up resources

Remove the local sample when you no longer need it. Delete Azure resources only if you created disposable resources specifically for this quickstart.

1. From the project directory, move to its parent directory and delete the local project:

   ### [Windows PowerShell](#tab/powershell)

   ```powershell
   Set-Location ..
   Remove-Item -Recurse -Force postgresql-node-quickstart
   ```

   ### [macOS/Linux](#tab/bash)

   ```bash
   cd ..
   rm -rf postgresql-node-quickstart
   ```

   ---

1. If the flexible server is in a resource group that you created only for this quickstart, open **Resource groups** in the Azure portal, select that resource group, and then select **Delete resource group**. Don't delete a resource group that contains shared or pre-existing resources.

## Related content

- [Create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md)
- [Configure networking for Azure Database for PostgreSQL](../network/how-to-networking.md)
- [Choose a PostgreSQL connection library](concepts-connection-libraries.md)
- [Apply connection pooling best practices](concepts-connection-pooling-best-practices.md)
