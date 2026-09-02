---
title: "Quickstart: Connect to Azure Database for PostgreSQL with Node.js"
description: Connect a Node.js application to Azure Database for PostgreSQL by using pg, TLS certificate validation, and environment-based credentials.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 09/01/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: "javascript"
---

# Quickstart: Connect to Azure Database for PostgreSQL with Node.js

In this quickstart, you use the `pg` package to connect a Node.js application to an existing Azure Database for PostgreSQL flexible server. The application uses PostgreSQL password authentication and a Transport Layer Security (TLS) connection that validates the server certificate.

You run a `SELECT` statement and print the value returned by PostgreSQL. The printed connection message and query result confirm that the TLS-protected connection works. This quickstart creates only a local Node.js project and doesn't create or change database objects.

## Prerequisites

- An Azure account with an active subscription.
- An existing Azure Database for PostgreSQL flexible server. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- [Node.js with npm](https://nodejs.org/en/download) installed.
- Network access from the computer that runs the application:
  - For a server with public access, add the computer's source IP address to the server firewall rules, and allow outbound TCP traffic on port 5432.
  - For a server with private access, run the application from a resource on the same virtual network as the server.
- The server endpoint, administrator login, password, and name of an existing database. Azure Database for PostgreSQL flexible server creates a database named `postgres` by default.

## Create the Node.js project

Create a local project and install the `pg` package from your terminal.

1. Create a directory named `postgres-nodejs` and change to that directory:

   ```bash
   mkdir postgres-nodejs
   cd postgres-nodejs
   ```

1. Create a default `package.json` file:

   ```bash
   npm init -y
   ```

1. Install `pg`:

   ```bash
   npm install pg
   ```

1. Confirm that npm lists `pg` as an installed dependency:

   ```bash
   npm list pg
   ```

   The output contains an entry for `pg`. The installed version can vary.

## Get the connection values

Get the connection values from your flexible server before you configure the application.

1. In the [Azure portal](https://portal.azure.com), open your Azure Database for PostgreSQL flexible server.

1. On the server **Overview** page, copy the **Endpoint** and **Administrator login** values. The endpoint is a fully qualified domain name such as `my-server.postgres.database.azure.com`; use this name instead of an IP address.

1. Identify the database to query. Use `postgres` if you want to connect to the default database.

1. Use the administrator password you set when you created the server. If you don't remember it, reset the password from the server page in the Azure portal.

## Set the environment variables

Store the connection values in environment variables so that credentials don't appear in the JavaScript source.

### PowerShell

Set the variables in the PowerShell session where you'll run the application. Replace each example value with the value from your server.

```powershell
$env:DBHOST = "my-server.postgres.database.azure.com"
$env:DBNAME = "postgres"
$env:DBUSER = "my-admin-user"
$env:DBPASSWORD = "your-password"
```

### Command Prompt

Set the variables in the Command Prompt session where you'll run the application. Replace each example value with the value from your server.

```cmd
set DBHOST=my-server.postgres.database.azure.com
set DBNAME=postgres
set DBUSER=my-admin-user
set DBPASSWORD=your-password
```

### macOS and Linux

Set the variables in the Bash session where you'll run the application. Replace each example value with the value from your server.

```bash
export DBHOST="my-server.postgres.database.azure.com"
export DBNAME="postgres"
export DBUSER="my-admin-user"
export DBPASSWORD="your-password"
```

## Connect and run the query

Create an application that opens a TLS connection, runs a nonmutating query, prints the result, and closes the client.

1. Create a file named `index.js` in the `postgres-nodejs` directory.

1. Add the following JavaScript to `index.js`:

   ```javascript
   const { Client } = require('pg');

   const client = new Client({
     host: process.env.DBHOST,
     port: 5432,
     database: process.env.DBNAME,
     user: process.env.DBUSER,
     password: process.env.DBPASSWORD,
     ssl: {
       rejectUnauthorized: true,
     },
   });

   async function main() {
     let connected = false;

     try {
       await client.connect();
       connected = true;

       const result = await client.query('SELECT NOW() AS current_time');
       console.log('Connection and query succeeded.');
       console.log('Database time:', result.rows[0].current_time);
     } catch (error) {
       console.error('Connection or query failed:', error.message);
       process.exitCode = 1;
     } finally {
       if (connected) {
         try {
           await client.end();
         } catch (error) {
           console.error('Failed to close the database connection:', error.message);
           process.exitCode = 1;
         }
       }
     }
   }

   main();
   ```

   The `ssl` configuration uses the trusted root certificates available to Node.js and rejects a server certificate that it can't validate. Don't set `rejectUnauthorized` to `false`. Don't add SSL parameters to a connection string alongside this `ssl` object because those parameters replace the object.

1. Run the application from the same terminal session where you set the environment variables:

   ```bash
   node index.js
   ```

   A successful run prints output in this form:

   ```output
   Connection and query succeeded.
   Database time: <timestamp returned by PostgreSQL>
   ```

   The timestamp varies for each run. If the connection or query fails, the application prints the error and exits with a nonzero status. Check the server endpoint, credentials, firewall or virtual network path, and trusted root certificates before you retry.

## Clean up resources

Delete the local `postgres-nodejs` directory when you no longer need the sample. The quickstart doesn't create Azure resources or database objects, so you can keep the existing flexible server and its network resources for other work.

If you created Azure resources only to complete this quickstart, remove them when you no longer need them. Delete the resource group to remove all resources in it, or delete only the flexible server if the resource group contains resources you want to keep.

## Related content

> [!div class="nextstepaction"]
> [Connection libraries for Azure Database for PostgreSQL](concepts-connection-libraries.md)

- [Configure Azure Database for PostgreSQL networking](../network/how-to-networking.md)
- [Transport Layer Security in Azure Database for PostgreSQL](../security/security-tls.md)
