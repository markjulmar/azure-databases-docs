---
title: "Quickstart: Connect and Query With Node.js and pg"
description: Connect a Node.js application to Azure Database for PostgreSQL flexible server with pg and TLS, then run a SELECT query and print its result.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL flexible server, so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/24/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: javascript
---

# Quickstart: Connect and query with Node.js in Azure Database for PostgreSQL

In this quickstart, you use the `pg` package for Node.js to connect to an existing Azure Database for PostgreSQL flexible server. Azure Database for PostgreSQL is a managed service for running, managing, and scaling PostgreSQL servers in Azure.

You create a minimal local project, configure a Transport Layer Security (TLS) connection, run one `SELECT` statement, and print its returned message. The printed message confirms that the application connected and completed the query.

## Prerequisites

Before you begin, make sure you have:

- An Azure account with an active subscription. If you don't have one, [create a free Azure account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing Azure Database for PostgreSQL flexible server. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your client to the server. For a private-access server, run the application from a resource in the same virtual network.
- The server host name, database name, administrator user name, and password. In the Azure portal, open your flexible server and copy the server name and administrator login from the **Overview** page. Use the password that you set for the administrator and the name of the database you want to query.
- Node.js and npm installed on your client. If needed, [install Node.js and npm](https://nodejs.org/en/download/package-manager).
- A Bash-compatible shell for running the commands in this quickstart.
- A local PEM file that contains the trusted root certificate authorities for the server. Follow [Configure TLS connections in Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md#install-trusted-root-certificate-authorities-cas) to prepare the file.

## Create the Node.js project

Create a project directory and install `pg`, the PostgreSQL client package for Node.js.

1. Open a Bash-compatible shell.
1. Create a directory named *nodejspostgresql*, and then change to that directory.

   ```bash
   mkdir nodejspostgresql
   cd nodejspostgresql
   ```

1. Initialize the project and install `pg`.

   ```bash
   npm init -y
   npm install pg
   ```

1. Confirm that npm lists `pg` as an installed dependency.

   ```bash
   npm list pg
   ```

## Configure the connection values

Store the connection values in environment variables so that the application doesn't contain your password or server-specific settings.

1. In the same terminal, set the following environment variables. Replace each placeholder with your server value, and set `AZURE_POSTGRESQL_SSLROOTCERT` to the path of the PEM file from the prerequisites.

   ```bash
   export AZURE_POSTGRESQL_HOST="<server-name>.postgres.database.azure.com"
   export AZURE_POSTGRESQL_USER="<administrator-login>"
   export AZURE_POSTGRESQL_DATABASE="<database-name>"
   export AZURE_POSTGRESQL_PORT="5432"
   export AZURE_POSTGRESQL_SSLROOTCERT="<path-to-root-ca-pem-file>"
   read -s -p "PostgreSQL password: " AZURE_POSTGRESQL_PASSWORD
   export AZURE_POSTGRESQL_PASSWORD
   printf '\n'
   ```

1. Confirm that the nonsecret settings contain your values.

   ```bash
   printf '%s\n' \
     "$AZURE_POSTGRESQL_HOST" \
     "$AZURE_POSTGRESQL_USER" \
     "$AZURE_POSTGRESQL_DATABASE" \
     "$AZURE_POSTGRESQL_PORT" \
     "$AZURE_POSTGRESQL_SSLROOTCERT"
   ```

> [!IMPORTANT]
> The password command prompts for the value without showing it in the terminal. Don't hard-code the password in *index.js* or commit it to source control.

## Create the connection application

Create a JavaScript application that validates the server certificate, connects with `pg.Client`, runs one query, prints its result, and closes the client.

1. Create a file named *index.js* in the *nodejspostgresql* directory.
1. Add the following code to *index.js*:

   ```javascript
   const fs = require('node:fs');
   const { Client } = require('pg');

   const client = new Client({
     host: process.env.AZURE_POSTGRESQL_HOST,
     user: process.env.AZURE_POSTGRESQL_USER,
     password: process.env.AZURE_POSTGRESQL_PASSWORD,
     database: process.env.AZURE_POSTGRESQL_DATABASE,
     port: Number(process.env.AZURE_POSTGRESQL_PORT),
     ssl: {
       rejectUnauthorized: true,
       ca: fs.readFileSync(
         process.env.AZURE_POSTGRESQL_SSLROOTCERT,
         'utf8'
       ),
     },
   });

   async function main() {
     try {
       await client.connect();
       const result = await client.query(
         "SELECT 'Connected to Azure Database for PostgreSQL' AS message"
       );
       console.log(result.rows[0].message);
     } finally {
       await client.end();
     }
   }

   main().catch((error) => {
     console.error(error.message);
     process.exitCode = 1;
   });
   ```

1. Save *index.js*.

## Run the query and verify the result

Run the application from the terminal where you configured the environment variables.

1. Run the Node.js application.

   ```bash
   node index.js
   ```

1. Confirm that the terminal prints the message returned by the `SELECT` statement:

   ```output
   Connected to Azure Database for PostgreSQL
   ```

If the command returns a connection or certificate error instead, check the host, credentials, network access, and PEM file path before you run the application again.

## Clean up local artifacts

This quickstart creates only the local *nodejspostgresql* project and doesn't create the prerequisite flexible server.

Choose one of these cleanup options:

- **Keep the project:** Keep the *nodejspostgresql* directory if you plan to extend the application.
- **Delete the project:** If you no longer need it, delete the *nodejspostgresql* directory. Deleting this directory removes *index.js*, *package.json*, *package-lock.json*, and the installed packages without affecting your flexible server.

## Related content

- [Review PostgreSQL connection libraries](concepts-connection-libraries.md)
- [Configure TLS connections in Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md)
- [Connect with the private access connectivity method](quickstart-create-connect-server-vnet.md)
