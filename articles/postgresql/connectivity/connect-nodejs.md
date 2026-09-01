---
title: "Connect With Node.js Using pg - Quickstart"
description: Connect a Node.js application to Azure Database for PostgreSQL by using pg, certificate-validating TLS, and environment-based credentials.
#customer intent: As a Node.js developer, I want to connect securely to Azure Database for PostgreSQL, so that I can query data from my application.
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

In this quickstart, you use the `pg` package in a local Node.js application to connect over Transport Layer Security (TLS) to an existing Azure Database for PostgreSQL flexible server. This first connection validates that your application can access the managed PostgreSQL database before you add database-dependent features. You run a `SELECT` statement and print its returned value to confirm that the application connected and queried the database.

This quickstart uses PostgreSQL authentication and creates only local project files. It doesn't create or change Azure resources or database objects.

## Prerequisites

- An Azure account with an active subscription. [Create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure Database for PostgreSQL flexible server and a database. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- The server's fully qualified domain name (FQDN), database name, PostgreSQL username, and password. Get the FQDN from the server's **Overview** page in the Azure portal. You can use the default `postgres` database.
- Network access from your development computer. For public access, add your client IP address to the server's firewall rules, and allow outbound traffic on TCP port 5432. For private access, run the application from a resource in the same virtual network. For setup instructions, see [Azure Database for PostgreSQL networking](../network/how-to-networking.md).
- A currently supported [Long-Term Support (LTS) release of Node.js](https://nodejs.org/en/download) with npm installed.
- A PEM file that contains the trusted root Certificate Authorities (CAs) used by Azure Database for PostgreSQL. Follow [Install trusted root CAs](../security/security-tls-how-to-connect.md#install-trusted-root-certificate-authorities-cas), and note the full path to the resulting PEM file.

## Create a Node.js project and install pg

Create a local project folder, and install the `pg` package in that folder.

1. Create a folder named `nodejs-postgresql-quickstart`, open a terminal in the folder, and make it your current directory.

1. Install `pg`:

   ```bash
   npm install pg
   ```

1. Confirm that the package is installed:

   ```bash
   npm list pg
   ```

   The output lists `pg` and its installed version:

   ```output
   nodejs-postgresql-quickstart
   └── pg@<installed-version>
   ```

## Get the connection values

Collect the Azure Database for PostgreSQL values that the application uses. Keep the password outside the JavaScript source file.

1. In the [Azure portal](https://portal.azure.com/), open your Azure Database for PostgreSQL flexible server.

1. On the server's **Overview** page, copy the fully qualified **Server name**. It has the form `<server-name>.postgres.database.azure.com`.

1. Note the database name and PostgreSQL username that you use to sign in. Use the corresponding PostgreSQL password when you set the environment variables.

1. Note the full path to the trusted root CA PEM file that you prepared in the prerequisites.

## Set the connection environment variables

Set the connection values in the terminal where you'll run the application. The `pg` client reads the standard PostgreSQL environment variables, and the application reads `PGSSLROOTCERT` to load the trusted root CA file.

1. Set the environment variables. Replace each placeholder with the value you collected:

   ```bash
   export PGHOST="<server-name>.postgres.database.azure.com"
   export PGPORT="5432"
   export PGDATABASE="<database-name>"
   export PGUSER="<username>"
   export PGPASSWORD="<password>"
   export PGSSLROOTCERT="/full/path/to/azure-postgresql-root-ca.pem"
   ```

1. Confirm that the nonsecret values are set:

   ```bash
   printf 'Host: %s\nPort: %s\nDatabase: %s\nUser: %s\nCA file: %s\n' \
     "$PGHOST" "$PGPORT" "$PGDATABASE" "$PGUSER" "$PGSSLROOTCERT"
   ```

   Each label should show the value that you entered. If a value is empty, set that environment variable again before you continue.

   Don't print `PGPASSWORD` or commit credentials to source control.

## Add the Node.js application

The application creates one `pg` client, loads the trusted root CA file into an explicit TLS configuration, runs one query, and closes the client in a `finally` block.

1. Create a file named `index.js` in the project folder.

1. Add the following code to `index.js`:

   ```javascript
   const fs = require('node:fs')
   const { Client } = require('pg')

   let client

   async function main() {
     try {
       client = new Client({
         host: process.env.PGHOST,
         port: Number(process.env.PGPORT),
         database: process.env.PGDATABASE,
         user: process.env.PGUSER,
         password: process.env.PGPASSWORD,
         ssl: {
           ca: fs.readFileSync(process.env.PGSSLROOTCERT, 'utf8'),
           rejectUnauthorized: true,
         },
       })

       await client.connect()

       const result = await client.query(
         "SELECT 'Connected to Azure Database for PostgreSQL' AS message"
       )
       console.log(result.rows[0].message)
     } catch (error) {
       console.error('Connection or query failed:', error.message)
       process.exitCode = 1
     } finally {
       if (client) {
         await client.end()
       }
     }
   }

   main()
   ```

   The separate connection fields preserve the explicit `ssl` object. Don't add `sslmode`, `sslcert`, `sslkey`, or `sslrootcert` parameters to a connection string because those parameters replace the `ssl` object in `pg`.

## Run the Node.js application

Run the application from the project folder and check the value returned by PostgreSQL.

1. Run the application:

   ```bash
   node index.js
   ```

1. Confirm that the application prints:

   ```output
   Connected to Azure Database for PostgreSQL
   ```

   This value comes from `result.rows[0].message`. Seeing it confirms that the application connected over certificate-validating TLS and completed the `SELECT` query.

If the application prints an error instead, check that the FQDN and credentials are correct, the client network can reach port 5432, and `PGSSLROOTCERT` points to the trusted root CA PEM file. Correct the value, and then run the application again.

## Use connection pooling for frequent queries

This quickstart uses a single `Client` for one query. For an application that makes frequent queries, use the `Pool` class included in `pg` to manage and reuse connections.

## Clean up local resources

The quickstart creates local project files and environment variables. It doesn't create the existing Azure Database for PostgreSQL flexible server.

1. Clear the connection environment variables from the current shell:

   ```bash
   unset PGHOST PGPORT PGDATABASE PGUSER PGPASSWORD PGSSLROOTCERT
   ```

1. Delete the `nodejs-postgresql-quickstart` folder when you no longer need the sample.

## Next steps

> [!div class="nextstepaction"]
> [Review TLS certificate verification for Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md)
