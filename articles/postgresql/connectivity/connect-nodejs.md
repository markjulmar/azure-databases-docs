---
title: "Quickstart: Connect to Azure Database for PostgreSQL with Node.js"
description: Connect a Node.js application to Azure Database for PostgreSQL with pg, require TLS, run a SELECT query, and verify the printed result.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/22/2026
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

# Quickstart: Connect to Azure Database for PostgreSQL with Node.js

In this quickstart, you use the `pg` package to connect a Node.js application to an existing Azure Database for PostgreSQL flexible server. The application requires Transport Layer Security (TLS), runs one `SELECT` statement, and prints the database name so you can confirm the connection succeeded.

This quickstart creates only a local Node.js project. It doesn't create, change, or delete data or Azure resources.

## Prerequisites

- An Azure account with an active subscription. [Create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing Azure Database for PostgreSQL flexible server and a database that your user can access. If needed, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your development environment to the server. Configure [Azure Database for PostgreSQL networking](../network/how-to-networking.md) for your server's public or private access method.
- [Node.js and npm](https://nodejs.org/en/download/package-manager) installed. Run `node --version` and `npm --version` to confirm that both commands are available.
- The server host, database name, user name, and password. In the Azure portal, open your flexible server:
  - Copy **Server name** and **Server admin login name** from **Overview**.
  - Get the database name from **Databases**.
  - Use the password for the selected PostgreSQL user. Reset the server administrator password from **Overview** if necessary.

## Create the Node.js project

Create a local project and install `pg`, the PostgreSQL client package for Node.js.

1. Create a project directory and change to it.

   ```bash
   mkdir nodejs-postgresql-quickstart
   cd nodejs-postgresql-quickstart
   ```

1. Initialize the project and install `pg`.

   ```bash
   npm init -y
   npm install pg
   ```

1. Confirm that the project lists `pg` as an installed dependency.

   ```bash
   npm list pg
   ```

## Configure the connection values

Store the connection values in environment variables so the application source doesn't contain your password or server details. The sample uses PostgreSQL port `5432` and enables TLS explicitly.

1. In a Bash shell, set the nonsecret connection values. Replace each example value with the value from your flexible server.

   ```bash
   export AZURE_POSTGRESQL_HOST='myserver.postgres.database.azure.com'
   export AZURE_POSTGRESQL_DATABASE='postgres'
   export AZURE_POSTGRESQL_USER='myuser'
   export AZURE_POSTGRESQL_PORT='5432'
   ```

1. Enter the PostgreSQL password at the prompt and export it for the current shell session.

   ```bash
   read -s -p "PostgreSQL password: " AZURE_POSTGRESQL_PASSWORD
   echo
   export AZURE_POSTGRESQL_PASSWORD
   ```

   Keep the shell open for the remaining steps. Don't add the password or a file that contains it to source control.

## Create the application

Create a short application that opens a TLS connection, queries the current database name, prints the result, and closes the connection.

1. Create a file named `index.js` in the `nodejs-postgresql-quickstart` directory.

1. Add the following JavaScript to `index.js`.

   ```javascript
   const { Client } = require('pg');

   const client = new Client({
     host: process.env.AZURE_POSTGRESQL_HOST,
     database: process.env.AZURE_POSTGRESQL_DATABASE,
     user: process.env.AZURE_POSTGRESQL_USER,
     password: process.env.AZURE_POSTGRESQL_PASSWORD,
     port: Number(process.env.AZURE_POSTGRESQL_PORT),
     ssl: true,
   });

   async function main() {
     let connected = false;

     try {
       await client.connect();
       connected = true;

       const result = await client.query(
         'SELECT current_database() AS database_name',
       );
       console.log(`Connected to database: ${result.rows[0].database_name}`);
     } catch (error) {
       console.error('Connection or query failed:', error.message);
       process.exitCode = 1;
     } finally {
       if (connected) {
         try {
           await client.end();
         } catch (error) {
           console.error('Failed to close the connection:', error.message);
           process.exitCode = 1;
         }
       }
     }
   }

   main();
   ```

## Run the application

Run the Node.js application from the project directory and check the printed database name.

1. Run the application with `node index.js`.

1. Confirm that the output contains the database name you set in `AZURE_POSTGRESQL_DATABASE`.

   The expected output is `Connected to database: <database-name>`.

   If the connection or query fails, the application prints the error and exits unsuccessfully. Check the host, database, user, password, network access, and port values before you run it again.

## Clean up resources

This quickstart creates local project files and retains the password only in the current shell environment. It doesn't create Azure resources.

1. Clear the connection values from the current shell.

   ```bash
   unset AZURE_POSTGRESQL_HOST
   unset AZURE_POSTGRESQL_DATABASE
   unset AZURE_POSTGRESQL_USER
   unset AZURE_POSTGRESQL_PASSWORD
   unset AZURE_POSTGRESQL_PORT
   ```

1. If you don't need the sample, leave the project directory and delete it.

   ```bash
   cd ..
   rm -rf nodejs-postgresql-quickstart
   ```

1. If you stored or exposed the password anywhere outside the cleared shell session, remove the stored copy or rotate the password.

Keep the prerequisite flexible server if you need it for other work. Delete it or its resource group separately only when you no longer need those Azure resources.

## Next step

> [!div class="nextstepaction"]
> [Handle transient connectivity errors](../troubleshoot/concepts-connectivity.md)
