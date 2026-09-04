---
title: "Connect to PostgreSQL With Node.js and pg"
description: Connect a Node.js app to Azure Database for PostgreSQL by using pg, verified TLS, and environment-based connection settings.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 09/03/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ms.devlang: javascript
ai-usage: ai-generated
---

# Quickstart: Connect to Azure Database for PostgreSQL with Node.js

In this quickstart, you connect a Node.js application to an existing Azure Database for PostgreSQL flexible server. You use the `pg` package to open a Transport Layer Security (TLS) connection that validates the server certificate, run a `SELECT` statement, and print the returned row.

The printed query result confirms that the application reached the flexible server and queried the database. This quickstart creates only a local Node.js project and certificate file. It doesn't create or change Azure resources.

## Prerequisites

- An Azure account with an active subscription. If you don't have an account, [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing Azure Database for PostgreSQL flexible server. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- The server's fully qualified host name, database name, user name, and password. The host name uses the form `<server-name>.postgres.database.azure.com`.
- Network access from the Node.js host to the flexible server on port `5432`. For public access, add an appropriately limited firewall rule for the client. For private access, run the application from a network that can reach the server. For setup details, see [Networking in Azure Database for PostgreSQL](../network/how-to-networking.md).
- Node.js and npm installed on your development host.
- The current complete set of Azure root certificate authority (CA) certificates, saved together in Privacy-Enhanced Mail (PEM) format as `azure-postgresql-ca.pem`. Download only the root CAs listed in [TLS in Azure Database for PostgreSQL](../security/security-tls.md#root-cas-used-by-azure-database-for-postgresql). Don't use an intermediate CA or an individual server certificate.

## Create the Node.js project

Create a local project and install `pg` without pinning a package version.

1. Open a Bash shell, and create the project directory:

   ```bash
   mkdir nodejs-postgresql
   cd nodejs-postgresql
   npm init -y
   ```

1. Install the `pg` package:

   ```bash
   npm install pg
   ```

1. Copy `azure-postgresql-ca.pem` into the `nodejs-postgresql` directory. Confirm that the file contains the current Azure root CA certificates and doesn't contain intermediate or server certificates.

## Configure the connection values

Set the flexible server connection values as environment variables so that credentials don't appear in the JavaScript source.

1. In the same Bash shell, set the environment variables. Replace each placeholder with the value for your flexible server:

   ```bash
   export AZURE_POSTGRESQL_HOST="<server-name>.postgres.database.azure.com"
   export AZURE_POSTGRESQL_PORT="5432"
   export AZURE_POSTGRESQL_DATABASE="<database-name>"
   export AZURE_POSTGRESQL_USER="<user-name>"
   export AZURE_POSTGRESQL_PASSWORD="<password>"
   export AZURE_POSTGRESQL_CA_PATH="./azure-postgresql-ca.pem"
   ```

1. Confirm that the variables are available without printing the password:

   ```bash
   printf 'Host: %s\nDatabase: %s\nUser: %s\nCA file: %s\n' \
     "$AZURE_POSTGRESQL_HOST" \
     "$AZURE_POSTGRESQL_DATABASE" \
     "$AZURE_POSTGRESQL_USER" \
     "$AZURE_POSTGRESQL_CA_PATH"
   ```

   The output shows the host, database, user, and CA file that the application uses. If a value is empty or incorrect, set that environment variable again before you continue.

## Add the connection code

Create a complete JavaScript application that reads the connection values, validates the TLS certificate against the Azure root CAs, and queries the flexible server.

1. Create a file named `index.mjs` in the `nodejs-postgresql` directory.

1. Add the following code to `index.mjs`:

   ```javascript
   import { readFileSync } from 'node:fs';
   import { Client } from 'pg';

   const requiredVariables = [
     'AZURE_POSTGRESQL_HOST',
     'AZURE_POSTGRESQL_PORT',
     'AZURE_POSTGRESQL_DATABASE',
     'AZURE_POSTGRESQL_USER',
     'AZURE_POSTGRESQL_PASSWORD',
     'AZURE_POSTGRESQL_CA_PATH',
   ];

   for (const variable of requiredVariables) {
     if (!process.env[variable]) {
       throw new Error(`Set the ${variable} environment variable before running the application.`);
     }
   }

   const port = Number.parseInt(process.env.AZURE_POSTGRESQL_PORT, 10);
   if (!Number.isInteger(port)) {
     throw new Error('AZURE_POSTGRESQL_PORT must be an integer.');
   }

   const client = new Client({
     host: process.env.AZURE_POSTGRESQL_HOST,
     port,
     database: process.env.AZURE_POSTGRESQL_DATABASE,
     user: process.env.AZURE_POSTGRESQL_USER,
     password: process.env.AZURE_POSTGRESQL_PASSWORD,
     ssl: {
       ca: readFileSync(process.env.AZURE_POSTGRESQL_CA_PATH, 'utf8'),
       rejectUnauthorized: true,
     },
   });

   let connected = false;

   try {
     await client.connect();
     connected = true;

     const result = await client.query(
       "SELECT 'Connection successful' AS message"
     );
     console.log(result.rows[0]);
   } catch (error) {
     console.error('PostgreSQL operation failed:', error);
     process.exitCode = 1;
   } finally {
     if (connected) {
       try {
         await client.end();
       } catch (error) {
         console.error('Failed to close the PostgreSQL connection:', error);
         process.exitCode = 1;
       }
     }
   }
   ```

   The `ssl.ca` property supplies the trusted Azure root CAs, and `rejectUnauthorized: true` requires certificate validation. The `finally` block closes an established client after a successful query or a query failure.

## Run the query

Run the application from the project directory and check the returned row.

1. Run the Node.js application:

   ```bash
   node index.mjs
   ```

1. Confirm that the output contains the row returned by the `SELECT` statement:

   ```output
   { message: 'Connection successful' }
   ```

   If the application reports a connection error, verify the environment-variable values, the CA file, and network access to port `5432`. For more diagnosis guidance, see [Troubleshoot connection issues](../troubleshoot/how-to-troubleshoot-common-connection-issues.md).

## Clean up resources

Remove the local project and its certificate file when you no longer need them. These steps don't remove or change the pre-existing flexible server.

1. Clear the connection values from the current Bash shell:

   ```bash
   unset AZURE_POSTGRESQL_HOST \
     AZURE_POSTGRESQL_PORT \
     AZURE_POSTGRESQL_DATABASE \
     AZURE_POSTGRESQL_USER \
     AZURE_POSTGRESQL_PASSWORD \
     AZURE_POSTGRESQL_CA_PATH
   ```

1. Change to the parent directory, and remove the local project, including `index.mjs`, installed packages, and `azure-postgresql-ca.pem`:

   ```bash
   cd ..
   rm -r nodejs-postgresql
   ```

## Related content

> [!div class="nextstepaction"]
> [Review TLS configuration for Azure Database for PostgreSQL](../security/security-tls.md)

- [Connection libraries for Azure Database for PostgreSQL](concepts-connection-libraries.md)
- [Networking in Azure Database for PostgreSQL](../network/how-to-networking.md)
