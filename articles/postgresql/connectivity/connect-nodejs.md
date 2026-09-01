---
title: Connect to Azure PostgreSQL with Node.js
description: Connect a Node.js app to Azure Database for PostgreSQL with node-postgres, TLS certificate validation, and parameterized queries.
#customer intent: As a Node.js developer, I want to connect to an Azure Database for PostgreSQL flexible server, so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 09/01/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.custom:
  - mvc
  - mode-api
  - devx-track-js
ms.devlang: javascript
---

# Quickstart: Use Node.js to connect and query data in Azure Database for PostgreSQL

In this quickstart, you use the `pg` package for Node.js to connect to an existing Azure Database for PostgreSQL flexible server. The sample creates an `inventory` table, inserts three rows, and runs a parameterized query over a TLS connection.

Successful output confirms the connection and returns the inserted banana row with a quantity of 150. This quickstart changes data in an existing database but doesn't create an Azure resource.

## Prerequisites

- An Azure account with access to an existing Azure Database for PostgreSQL flexible server and database. If you don't have a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md) before you begin.
- A PostgreSQL user that has permission to create a table and query data in the target database. Get the fully qualified server name and administrator login from the server's **Overview** page in the Azure portal.
- Network access from your development computer to the server. For public access, add your client IP address to the server's firewall rules. For private access, connect from a host that can route traffic to the server's virtual network. For setup guidance, see [Networking in Azure Database for PostgreSQL flexible server](../network/how-to-networking.md).
- A supported long-term support (LTS) release of [Node.js with npm](https://nodejs.org/en/download/current) installed on macOS, Linux, or Windows.
- A PEM file that contains the trusted Azure root Certificate Authority (CA) certificates. Follow [Install trusted root Certificate Authorities](../security/security-tls-how-to-connect.md#install-trusted-root-certificate-authorities-cas) to create the file. Don't use an intermediate CA or an individual server certificate as the trust root.

## Set up the Node.js project

Create a local npm project and install the current generally available `pg` package.

1. Open a terminal and create the project folder:

   ```bash
   mkdir nodejspostgres
   cd nodejspostgres
   ```

1. Initialize the npm project:

   ```bash
   npm init -y
   ```

1. Install `pg` without a version pin:

   ```bash
   npm install pg
   ```

1. Confirm that npm lists `pg` as an installed dependency:

   ```bash
   npm list pg
   ```

   Continue when the output shows `pg` under the `nodejspostgres` project. If npm reports that `pg` is missing, run `npm install pg` again from the folder that contains `package.json`.

## Configure the connection environment

Store the connection details and trusted root CA path in environment variables so that the JavaScript file doesn't contain credentials or machine-specific values. The sample maps these custom variables explicitly into `pg.Client`.

1. Set the variables for your operating system. Replace each placeholder with the value described in the code comments.

   **macOS or Linux**

   ```bash
   # Fully qualified name from the server's Overview page.
   export DBHOST="<server-name>.postgres.database.azure.com"
   # An existing database, such as postgres.
   export DBNAME="postgres"
   # The server administrator login or another PostgreSQL user.
   export DBUSER="<username>"
   export DBPASSWORD="<password>"
   export DBPORT="5432"
   # Local path to the PEM file of trusted Azure root CA certificates.
   export DBSSLROOTCERT="/path/to/azure-root-ca.pem"
   ```

   **Windows PowerShell**

   ```powershell
   # Fully qualified name from the server's Overview page.
   $env:DBHOST = "<server-name>.postgres.database.azure.com"
   # An existing database, such as postgres.
   $env:DBNAME = "postgres"
   # The server administrator login or another PostgreSQL user.
   $env:DBUSER = "<username>"
   $env:DBPASSWORD = "<password>"
   $env:DBPORT = "5432"
   # Local path to the PEM file of trusted Azure root CA certificates.
   $env:DBSSLROOTCERT = "C:\path\to\azure-root-ca.pem"
   ```

1. Confirm that all six variables are set in the same terminal session where you'll run the application. Don't print `DBPASSWORD` or commit credentials and certificate files to source control.

## Create the Node.js application

Use one `pg.Client` because this quickstart runs a single sequence of operations. Applications that query the database frequently should use `pg.Pool` to reuse connections and must release checked-out clients.

> [!WARNING]
> The sample runs `DROP TABLE IF EXISTS inventory` each time. Rerunning it permanently deletes any existing table named `inventory` and its data in the target database.

1. Create a file named `app.js` in the `nodejspostgres` folder.

1. Copy the following complete CommonJS application into `app.js`:

   ```javascript
   const fs = require('node:fs');
   const { Client } = require('pg');

   const requiredVariables = [
     'DBHOST',
     'DBNAME',
     'DBUSER',
     'DBPASSWORD',
     'DBPORT',
     'DBSSLROOTCERT',
   ];

   const missingVariables = requiredVariables.filter((name) => !process.env[name]);
   if (missingVariables.length > 0) {
     throw new Error(`Set these environment variables: ${missingVariables.join(', ')}`);
   }

   const client = new Client({
     host: process.env.DBHOST,
     database: process.env.DBNAME,
     user: process.env.DBUSER,
     password: process.env.DBPASSWORD,
     port: Number(process.env.DBPORT),
     ssl: {
       ca: fs.readFileSync(process.env.DBSSLROOTCERT, 'utf8'),
       rejectUnauthorized: true,
     },
   });

   async function main() {
     let connected = false;
     let operationError;

     try {
       await client.connect();
       connected = true;
       console.log('Connection established');

       await client.query('DROP TABLE IF EXISTS inventory');
       await client.query(`
         CREATE TABLE inventory (
           id serial PRIMARY KEY,
           name VARCHAR(50),
           quantity INTEGER
         )
       `);
       console.log('Created inventory table');

       const items = [
         ['banana', 150],
         ['orange', 154],
         ['apple', 100],
       ];

       for (const item of items) {
         await client.query(
           'INSERT INTO inventory (name, quantity) VALUES ($1, $2)',
           item
         );
       }
       console.log('Inserted 3 rows');

       const result = await client.query(
         'SELECT name, quantity FROM inventory WHERE name = $1',
         ['banana']
       );
       console.log('Parameterized query result:', result.rows);
     } catch (error) {
       operationError = error;
       throw error;
     } finally {
       if (connected) {
         try {
           await client.end();
         } catch (closeError) {
           if (operationError) {
             console.error('Connection close failed:', closeError.message);
           } else {
             throw closeError;
           }
         }
       }
     }
   }

   main().catch((error) => {
     console.error('Operation failed:', error.message);
     process.exitCode = 1;
   });
   ```

1. Save `app.js`. The application awaits each operation in order and closes the client when the operations finish or an operation fails after connection.

## Run the application

Run the Node.js application from the terminal session that contains your environment variables.

1. From the `nodejspostgres` folder, run:

   ```bash
   node app.js
   ```

1. Confirm that the output includes the connection, table, insert, and query checkpoints:

   ```output
   Connection established
   Created inventory table
   Inserted 3 rows
   Parameterized query result: [ { name: 'banana', quantity: 150 } ]
   ```

   If the application fails before it runs the SQL statements, verify the server name, credentials, trusted root CA file, and public-firewall or private-network path before you retry.

## Clean up resources

The `inventory` table and its rows remain in the existing database. If the Azure server or another Azure resource exists only for temporary testing, delete that resource when you're finished.

## Next step

> [!div class="nextstepaction"]
> [Configure and validate TLS connections](../security/security-tls-how-to-connect.md)
