---
title: "Quickstart: Connect Node.js to PostgreSQL with pg"
description: Connect a Node.js application to Azure Database for PostgreSQL with the pg client, TLS certificate validation, and a test query.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/10/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.custom:
  - mvc
  - devx-track-js
  - mode-api
ms.devlang: javascript
---

# Quickstart: Connect and query data in Azure Database for PostgreSQL flexible server with Node.js

In this quickstart, you connect a Node.js application to an Azure Database for PostgreSQL flexible server by using the `pg` (node-postgres) client. The sample validates the server's TLS certificate, runs a `SELECT` query, prints the result, and closes the connection.

This quickstart is for Node.js developers who are familiar with JavaScript but are new to Azure Database for PostgreSQL.

## Prerequisites

- An Azure account with an active subscription. [Create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure Database for PostgreSQL flexible server. If you need one, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your client workstation to the server. For public access, add your client IP address to the server's firewall rules. For private access, connect from a resource in the server's virtual network. For more information, see [Firewall rules in Azure Database for PostgreSQL](../security/security-firewall-rules.md).
- A supported [Node.js](https://nodejs.org/en/download) installation that includes npm.

## Prepare the Node.js project

Create a Node.js project and install `pg` as its only PostgreSQL client library.

1. Create a folder named `postgresql-quickstart`, and change to that folder.

1. Initialize the project:

   ```bash
   npm init -y
   ```

1. Install `pg`:

   ```bash
   npm install pg
   ```

   When installation succeeds, npm adds `pg` to the `dependencies` section of `package.json`.

## Get the PostgreSQL connection values

Get the server endpoint and administrator login from the Azure portal. Keep credentials outside the application source code.

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Search for and open your Azure Database for PostgreSQL flexible server.

1. On the server's **Overview** page, copy the **Endpoint** and **Administrator login** values.

1. Use `postgres` as the database name, unless you created a different database.

1. Use the administrator password that you set when you created the server. If you don't remember it, select **Reset password** on the server's **Overview** page and set a new password before you continue.

## Configure TLS and environment variables

Azure Database for PostgreSQL requires TLS for client connections. Configure `pg` to verify the server certificate against trusted root Certificate Authorities (CAs).

1. Follow [Install trusted root Certificate Authorities](../security/security-tls-how-to-connect.md#install-trusted-root-certificate-authorities-cas) to download the current root CA certificates and combine them into a PEM file. Save the PEM file in a location that the Node.js process can read.

1. Set the connection values and the full path to the combined CA PEM file as environment variables. Replace each placeholder with the value described in the command.

   ### [Windows PowerShell](#tab/powershell)

   ```powershell
   $env:PGHOST="<endpoint-from-the-azure-portal>"
   $env:PGPORT="5432"
   $env:PGDATABASE="postgres"
   $env:PGUSER="<administrator-login-from-the-azure-portal>"
   $env:PGPASSWORD="<administrator-password>"
   $env:PGSSLROOTCERT="<full-path-to-combined-ca-pem>"
   ```

   ### [macOS or Linux](#tab/bash)

   ```bash
   export PGHOST="<endpoint-from-the-azure-portal>"
   export PGPORT="5432"
   export PGDATABASE="postgres"
   export PGUSER="<administrator-login-from-the-azure-portal>"
   export PGPASSWORD="<administrator-password>"
   export PGSSLROOTCERT="<full-path-to-combined-ca-pem>"
   ```

   ---

   These environment variables apply to the current terminal session. Keep this terminal open for the remaining steps.

## Connect and run a SELECT query

Create the application, connect with TLS certificate validation, and run a query that returns a stable test value.

1. In the `postgresql-quickstart` folder, create a file named `index.js`.

1. Add the following code to `index.js`:

   ```javascript
   const fs = require("fs");
   const { Client } = require("pg");

   const client = new Client({
     host: process.env.PGHOST,
     port: Number(process.env.PGPORT),
     database: process.env.PGDATABASE,
     user: process.env.PGUSER,
     password: process.env.PGPASSWORD,
     ssl: {
       rejectUnauthorized: true,
       ca: fs.readFileSync(process.env.PGSSLROOTCERT, "utf8"),
     },
   });

   async function main() {
     try {
       await client.connect();
       const result = await client.query("SELECT 1 AS connection_status;");
       console.log("Connection successful:", result.rows[0]);
     } catch (error) {
       console.error("Connection failed:", error.message);
       process.exitCode = 1;
     } finally {
       await client.end();
     }
   }

   main();
   ```

   The `ssl` object enables certificate validation. Don't set `rejectUnauthorized` to `false`.

1. From the same terminal where you set the environment variables, run the application:

   ```bash
   node index.js
   ```

   A successful connection produces the following output:

   ```output
   Connection successful: { connection_status: 1 }
   ```

   If the connection fails, use the printed error to check the endpoint, credentials, firewall or virtual network access, and CA PEM file path. Correct the value, set its environment variable again, and rerun the application.

## Clean up resources

If you created the flexible server in a dedicated resource group for this quickstart and no longer need it, delete the resource group and all its resources.

1. Replace `<resource-group>` with the name of the resource group that contains the server, and run:

   ```azurecli
   az group delete --name "<resource-group>" --yes
   ```

1. In the Azure portal, confirm that the resource group no longer appears in **Resource groups** before you consider cleanup complete.

## Related content

- [Connection libraries in Azure Database for PostgreSQL](concepts-connection-libraries.md)
