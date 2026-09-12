---
title: Connect to PostgreSQL with Node.js - Quickstart
description: Connect a Node.js application to Azure Database for PostgreSQL flexible server with pg, TLS certificate validation, and a test query.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 09/12/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: javascript
---

# Quickstart: Connect to Azure Database for PostgreSQL with Node.js

Azure Database for PostgreSQL flexible server is a managed PostgreSQL service. The `pg` package, also known as node-postgres, lets a Node.js application connect to the service and run PostgreSQL queries.

In this quickstart, you create a local Node.js project, configure a Transport Layer Security (TLS) connection to an existing flexible server, run a `SELECT` statement, and print its returned value. The query doesn't create or change any database objects.

<!-- Evidence conflict [C2]: The submitted audience defines this PostgreSQL article for first-time Node.js developers, while the available Node.js sibling covers MySQL. The submitted audience governs; the MySQL sibling informs structure only, not product or driver behavior. -->

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing Azure Database for PostgreSQL flexible server. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- The server's fully qualified domain name (FQDN), target database name, PostgreSQL user name, and password.
- Network access from your workstation to the server:
  - For public access, [add your workstation's public IP address to the server firewall](../network/how-to-networking-servers-deployed-public-access-add-firewall-rules.md). Firewall changes can take up to five minutes to apply.
  - For private access, run the application from a network that can resolve and reach the server's private FQDN.
- An [Active LTS or Maintenance LTS release of Node.js](https://nodejs.org/en/about/previous-releases) with npm.
- A PEM file that contains the complete set of trusted Azure root certification authority (CA) certificates. Follow the [Azure Database for PostgreSQL TLS guidance](../security/security-tls.md#recommended-configurations-for-tls) to prepare the trusted root certificates.

## Create the Node.js project

Create a project folder and install the `pg` package in that folder.

1. Open a command line in the directory where you keep local projects.
1. Create a project directory, and change to that directory:

   ```bash
   mkdir postgres-node-quickstart
   cd postgres-node-quickstart
   ```

1. Create the Node.js project:

   ```bash
   npm init -y
   ```

1. Install `pg`:

   ```bash
   npm install pg
   ```

   Confirm that the command completes without an npm installation error before you continue.

## Configure the connection values

Store the connection values outside the application source. You use the following environment variables:

| Variable | Value |
| --- | --- |
| `DBHOST` | The server FQDN, such as `myserver.postgres.database.azure.com`. Don't use an IP address. |
| `DBPORT` | `5432` for a direct connection to the flexible server. |
| `DBNAME` | The target database name, such as the default `postgres` database. |
| `DBUSER` | The PostgreSQL user name. |
| `DBPASSWORD` | The password for the PostgreSQL user. |
| `DBSSLROOTCERT` | The local path to the PEM file that contains the trusted Azure root CA certificates. |

Replace the example values, and then set the variables in the command line where you'll run the application:

### [Windows](#tab/cmd)

```cmd
set "DBHOST=myserver.postgres.database.azure.com"
set "DBPORT=5432"
set "DBNAME=postgres"
set "DBUSER=myadmin"
set "DBPASSWORD=<password>"
set "DBSSLROOTCERT=C:\path\to\azure-root-cas.pem"
```

### [macOS/Linux](#tab/bash)

```bash
export DBHOST="myserver.postgres.database.azure.com"
export DBPORT="5432"
export DBNAME="postgres"
export DBUSER="myadmin"
export DBPASSWORD="<password>"
export DBSSLROOTCERT="/path/to/azure-root-cas.pem"
```

---

Keep the password and certificate configuration out of source control. The application uses the server FQDN during TLS hostname validation.

## Connect and run a query

Create a JavaScript application that opens one `pg` client connection, runs a non-mutating query, prints the selected value, and closes the client.

1. Create a file named `index.js` in the project directory.
1. Add the following code to `index.js`:

   ```javascript
   const { readFileSync } = require('node:fs');
   const { Client } = require('pg');

   const requiredVariables = [
     'DBHOST',
     'DBPORT',
     'DBNAME',
     'DBUSER',
     'DBPASSWORD',
     'DBSSLROOTCERT',
   ];

   for (const variable of requiredVariables) {
     if (!process.env[variable]) {
       throw new Error(`Set the ${variable} environment variable before running the application.`);
     }
   }

   const client = new Client({
     host: process.env.DBHOST,
     port: Number(process.env.DBPORT),
     database: process.env.DBNAME,
     user: process.env.DBUSER,
     password: process.env.DBPASSWORD,
     ssl: {
       ca: readFileSync(process.env.DBSSLROOTCERT, 'utf8'),
       rejectUnauthorized: true,
     },
   });

   async function main() {
     try {
       await client.connect();
       const result = await client.query(
         "SELECT 'Connected to Azure Database for PostgreSQL' AS message;"
       );
       console.log(result.rows[0].message);
     } catch (error) {
       console.error('Connection or query failed:', error);
       process.exitCode = 1;
     } finally {
       await client.end();
     }
   }

   main();
   ```

   The `ssl` object explicitly enables TLS. The trusted roots in `ssl.ca`, `rejectUnauthorized: true`, and the server FQDN configure certificate-chain and hostname validation. This validation is stronger than encryption-only configurations, which don't validate the server's identity.

   <!-- Evidence conflict [QS2]: Azure TLS sources use both verify-all and verify-full for comparable full-validation guidance. This Node.js procedure doesn't map either label to pg; it configures certificate and hostname validation through the pg ssl object. -->

   <!-- TODO: SME review required [QS2]: Confirm that this composed pg configuration connects to a current Azure Database for PostgreSQL flexible server and validates the certificate chain and server FQDN by using the complete Azure trusted root CA set. -->

1. Run the application:

   ```bash
   node index.js
   ```

1. Confirm that the application prints the value selected by the query:

   ```output
   Connected to Azure Database for PostgreSQL
   ```

   If the application reports a connection error, confirm that all environment variables are set, the FQDN and credentials are correct, and the public firewall rule or private network path is ready. Then run the application again.

   <!-- TODO: SME review required [QS3]: Confirm the end-to-end TLS connection and printed SELECT result against a current flexible server. -->

## Clean up resources

This quickstart creates local project files and doesn't create database objects. Remove sensitive local configuration when you no longer need it.

1. Unset the environment variables, especially `DBPASSWORD`, by using the syntax for your command-line environment.
1. Delete the `postgres-node-quickstart` directory if you don't need the local project.
1. If you created a flexible server or resource group only for this quickstart, follow the cleanup guidance in [Create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md) to avoid continued charges.

## Next step

> [!div class="nextstepaction"]
> [Configure and maintain TLS certificate validation](../security/security-tls-how-to-connect.md)

## Related content

- [Connection pooling best practices for Azure Database for PostgreSQL](concepts-connection-pooling-best-practices.md)
- [Use connection pooling with node-postgres](https://node-postgres.com/features/pooling)
- [Connection libraries for Azure Database for PostgreSQL](concepts-connection-libraries.md)
