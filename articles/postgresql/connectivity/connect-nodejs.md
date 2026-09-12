---
title: "Quickstart: Connect to Azure Database for PostgreSQL with Node.js"
description: Connect a Node.js application to Azure Database for PostgreSQL by using node-postgres, verified TLS, and a simple SELECT query.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 09/12/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ms.devlang: javascript
ai-usage: ai-generated
---

# Quickstart: Connect to Azure Database for PostgreSQL with Node.js

Azure Database for PostgreSQL flexible server is a managed PostgreSQL service for running, managing, and scaling PostgreSQL databases in the cloud. The `pg` package, also known as node-postgres, lets a Node.js application connect to and query PostgreSQL.

In this quickstart, you install `pg`, configure a client to connect to an existing flexible server by using Transport Layer Security (TLS), run a simple `SELECT` statement, and print `Hello world!` in the console. The procedure creates only local project files and doesn't provision Azure resources.

## Prerequisites

- An Azure account with an active subscription. If you don't have one, [create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure Database for PostgreSQL flexible server and a database user that uses PostgreSQL password authentication. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your development environment to the server. For a public-access server, add your client IP address to the server firewall rules. For a private-access server, run the application from a client that has private network connectivity to the server. For setup guidance, see [Networking in Azure Database for PostgreSQL flexible server](../network/how-to-networking.md).
- Node.js 24 LTS with npm. Get Node.js from the [Node.js download page](https://nodejs.org/en/download).
- A trusted root certificate authority (CA) certificate installed locally for Azure Database for PostgreSQL certificate validation. Follow [Install trusted root CAs](../security/security-tls-how-to-connect.md#install-trusted-root-certificate-authorities-cas).

## Prepare the Node.js project

Create a local project folder and install the `pg` package.

<!-- TODO: [QS1] SME: confirm the exact clean-project initialization command and resulting package.json ESM setting. -->

1. Open a terminal and create a project folder:

   ```bash
   mkdir postgres-node-quickstart
   cd postgres-node-quickstart
   ```

1. Install `pg`:

   ```bash
   npm install pg
   ```

## Configure connection values

Set environment variables for the flexible-server fully qualified domain name (FQDN), database, user, password, and local CA certificate path. Use the FQDN instead of an IP address because the server IP address might change.

1. In the Azure portal, open your flexible server's **Overview** page and copy the **Server name** and administrator login or database user name.

1. In the terminal where you'll run the application, set the following environment variables. Replace each placeholder with your server and local certificate values:

   ```bash
   export PGHOST="<server-name>.postgres.database.azure.com"
   export PGDATABASE="<database-name>"
   export PGUSER="<user-name>"
   export PGPASSWORD="<password>"
   export PGSSLROOTCERT="<path-to-trusted-root-ca.pem>"
   ```

   - `<server-name>.postgres.database.azure.com` is the FQDN from the server **Overview** page.
   - `<database-name>` is the database to query.
   - `<user-name>` and `<password>` are the PostgreSQL authentication credentials for that database.
   - `<path-to-trusted-root-ca.pem>` is the local path to the trusted root CA certificate.

Keep credentials out of source code and source control. The sample reads them from environment variables.

## Connect and query with TLS

Create an ECMAScript module that loads the trusted CA, connects with TLS certificate validation, queries the database, prints the result, and closes the client.

<!-- TODO: SME review required [QS1, QS2]: Run this integrated ESM sample against Azure Database for PostgreSQL and confirm CA loading, hostname verification, environment-variable mapping, client closure after success or failure, and the displayed output. -->

1. Create a file named `index.mjs` in the `postgres-node-quickstart` folder.

1. Add the following code to `index.mjs`:

   ```javascript
   import fs from 'node:fs';
   import { Client } from 'pg';

   let client;

   try {
     client = new Client({
       host: process.env.PGHOST,
       database: process.env.PGDATABASE,
       user: process.env.PGUSER,
       password: process.env.PGPASSWORD,
       ssl: {
         ca: fs.readFileSync(process.env.PGSSLROOTCERT, 'utf8'),
       },
     });

     await client.connect();
     console.log('Connected to Azure Database for PostgreSQL.');

     const result = await client.query(
       'SELECT $1::text AS message',
       ['Hello world!'],
     );
     console.log(result.rows[0].message);
   } catch (error) {
     console.error('Database operation failed:', error);
     process.exitCode = 1;
   } finally {
     if (client) {
       await client.end();
     }
   }
   ```

   The `ssl` object explicitly supplies the trusted CA to the Node.js TLS socket. Don't add `sslmode`, `sslrootcert`, `sslcert`, or `sslkey` to a connection string used with this configuration because those options replace the separately supplied `ssl` object.

1. Run the application:

   ```bash
   node index.mjs
   ```

## Verify the connection

A successful connection and query print the following output:

```output
Connected to Azure Database for PostgreSQL.
Hello world!
```

If the application instead prints `Database operation failed`, use the reported error to check the FQDN, credentials, network access, and CA certificate path before you run it again.

## Clean up resources

The quickstart creates only the local `postgres-node-quickstart` folder and its files. Delete that folder when you no longer need the sample.

Keep any existing shared server or resource group. If you created a dedicated flexible server or resource group only for this quickstart, follow the cleanup instructions in [Create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md#clean-up-resources) to delete only those resources.

## Next step

> [!div class="nextstepaction"]
> [Configure TLS connections for Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md)

## Related content

- [Connect and query overview](how-to-connect-query-guide.md)
- [Connection libraries for Azure Database for PostgreSQL](concepts-connection-libraries.md)
