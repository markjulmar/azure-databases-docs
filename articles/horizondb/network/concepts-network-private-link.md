---
title: Azure Private Link Concepts for Azure HorizonDB
description: Understand Azure Private Link networking, cluster endpoints, DNS, failover, and connectivity choices for Azure HorizonDB.
#customer intent: As a database developer, I want to understand private connectivity to Azure HorizonDB so that I can design application networking.
author: milenak
ms.author: mpopovic
ms.reviewer: maghan
ms.date: 09/10/2026
ms.service: azure-horizondb
ms.subservice: networking
ms.topic: concept-article
ai-usage: ai-generated
---

# Azure HorizonDB (Preview) Private Link networking concepts

Azure Private Link provides private connectivity from a virtual network to an Azure service through a private endpoint, which is a network interface with a private IP address in your virtual network. For Azure HorizonDB, Private Link is the private-network part of the single **Public access (allowed IP addresses) and private endpoints** connectivity mode.

Understanding the cluster's logical endpoints and its Domain Name System (DNS) behavior helps you choose the correct host name for read/write and read-only workloads. This article explains the confirmed HorizonDB endpoint behavior, the DNS and topology design principles that apply to private connectivity, and the boundaries that require product-specific confirmation.

## Prerequisites

- Azure HorizonDB and its Private Link support are in preview.
- You need an existing HorizonDB cluster, valid PostgreSQL credentials, and client network reachability to TCP port 5432. Private connectivity grants a network path; it doesn't grant access to databases or other PostgreSQL objects.
- HorizonDB supports Private Link, but it doesn't support virtual network injection or virtual network integration. A private endpoint doesn't place the cluster in a delegated subnet. Instead, the endpoint places a network interface for the service connection in a subnet that you select.

<!-- TODO: [C3, C4] SME: provide the HorizonDB-specific permissions, private endpoint approval requirements, subnet constraints, supported regions, quotas, and current Private Link preview limitations from a validated product specification, portal flow, API schema, or deployment. -->

## Private endpoint and cluster endpoint model

An Azure HorizonDB cluster exposes two logical connection endpoints:

- The **primary endpoint (read/write)** routes to the current primary replica. Use this fully qualified domain name (FQDN) for writes and reads that require the latest committed data.
- The **reader endpoint (read-only)** balances connections across standby replicas. If the cluster has no standby replica, this endpoint connects to the primary and can accept writes. When the cluster has one or more standby replicas, it connects to those replicas and is read-only.

A generic Azure private endpoint targets a private-link resource and one of that resource type's subresources. Don't treat the HorizonDB primary and reader endpoints as Private Link subresources unless HorizonDB product information explicitly defines that mapping.

<!-- TODO: [CE2, CE3] SME: state whether one HorizonDB private endpoint serves both the primary and reader FQDNs or whether separate private endpoints are required. Provide the exact accepted Private Link group IDs or subresource names from a Microsoft.HorizonDB API response, CLI output, portal capture, or product specification. -->

## Name resolution in a virtual network

Applications should keep the HorizonDB FQDN in their connection strings instead of replacing it with a private IP address. HorizonDB-assigned IP addresses aren't guaranteed to remain static, while the logical FQDN preserves the application's connection target.

Private endpoint designs use private DNS so clients in linked virtual networks resolve a service FQDN to the endpoint's private IP address. The HorizonDB-specific private DNS zone, aliases, and address records determine how the primary and reader FQDNs reach that private IP. Those names must come from HorizonDB product information; the Azure Database for PostgreSQL Flexible Server zone and subresource values don't apply to HorizonDB.

<!-- TODO: [CE2, CE3] SME: provide the exact HorizonDB private DNS zone and the complete CNAME and A-record relationships for both the primary and reader FQDNs. Include representative nslookup or dig results from a linked VNet and the matching private endpoint network interface or private DNS records. -->

For a hybrid network, an on-premises DNS server can conditionally forward Azure private-zone queries to an Azure DNS Private Resolver inbound endpoint or to a DNS forwarder hosted in Azure. The Azure resolver can then query Azure private DNS, and the on-premises client should receive the private endpoint IP for the service FQDN.

<!-- TODO: SME review required [CE3]: confirm this hybrid DNS composition for HorizonDB after the exact private DNS forwarding zone is known, including the forwarding target and expected lookup result. -->

## Replica reads and replica changes

HorizonDB's reader endpoint provides application-independent connection balancing across standby replicas. Adding a standby connects it to the cluster's shared storage, after which it serves read queries through the reader endpoint. Removing a standby reduces read capacity and can disable high availability when it removes the only standby.

These logical behaviors don't establish the network path between a HorizonDB private endpoint and each replica. In particular, the cluster endpoint model doesn't show whether replicas have distinct private networking objects.

<!-- TODO: [CE3, CE5] SME: confirm whether reader-endpoint traffic to every readable replica traverses the same private endpoint network interface and private IP, whether replica-specific private endpoints or DNS records exist, and what private endpoint, IP, and DNS objects change when replicas are added or removed. Provide a product trace or before-and-after resource and DNS capture. -->

## Failover and client reconnection

During failover, HorizonDB promotes a standby replica and redirects the read/write endpoint to the new primary. Existing connections drop, but applications reconnect with the same connection string. Applications should retry transient connection failures rather than connect directly to a replica or cache an IP address.

The logical endpoint redirection doesn't establish what happens to the Private Link resources or DNS data during failover.

<!-- TODO: [CE3, CE4] SME: document what happens to the private endpoint connection, network interface, private IP, private DNS records, and primary FQDN resolution during planned and automatic failover. Pair a failover trace with before-and-after endpoint and DNS observations. -->

## Private endpoints and public firewall access

Creating a private endpoint doesn't disable public access to HorizonDB. Both access paths coexist in the cluster's single connectivity mode. Public clients use publicly resolvable DNS and must match an allowed IPv4 range in the cluster-level firewall rules. Their traffic uses general internet paths rather than a private network.

Network reachability on either path remains separate from PostgreSQL authorization. Every client must use valid credentials, and PostgreSQL roles continue to control access to databases, tables, and other objects. The presence of a private endpoint doesn't by itself establish any further change to HorizonDB public DNS or public firewall behavior.

## Common private connectivity topologies

Azure Private Link generally supports clients in the same virtual network, clients in peered virtual networks, and on-premises clients connected through VPN or ExpressRoute. Each topology also needs DNS to return the private endpoint IP to the client. A hub-and-spoke design commonly centralizes private DNS or DNS forwarding in the hub while peered spokes provide application connectivity.

<!-- TODO: SME review required [CE4, CE5]: verify that no HorizonDB preview restriction narrows same-VNet, peered or hub-and-spoke, VPN, or ExpressRoute reachability, and confirm that each topology uses the HorizonDB private DNS zone supplied by the product team. -->

## Network security groups and route tables

Network security groups (NSGs) and user-defined routes (UDRs) on a private endpoint subnet apply only when private endpoint network policies are enabled for the relevant policy type. Route selection follows longest-prefix matching. Keep these controls separate from HorizonDB firewall rules, which govern the public access path.

<!-- TODO: SME review required [CE3, CE4]: verify the NSG directionality, UDR prefix guidance, and private endpoint network-policy settings against current Azure Private Link behavior, and confirm that they apply to a HorizonDB private endpoint. -->

## Troubleshoot connectivity and DNS

Start with the FQDN your application uses, and compare DNS results from an affected client with results from a client where connectivity works.

| Client-visible symptom | Discriminating check | Corrective direction |
| --- | --- | --- |
| `nslookup` or `dig` returns `NXDOMAIN`. | Check whether the expected private DNS zone is linked to the client's virtual network and contains the required address record. | Restore the missing virtual network link or private DNS record after the HorizonDB zone and record names are confirmed. |
| A DNS query returns `SERVFAIL` or times out. | Query the client's configured DNS server directly, and check conditional forwarding plus UDP and TCP port 53 between the client, forwarder, and resolver. | Correct the forwarding target or the network rule that blocks DNS traffic. On-premises resolvers can't query the Azure platform DNS address directly. |
| The HorizonDB FQDN resolves to a public IP instead of the expected private IP. | Check whether the client uses the intended resolver and whether that resolver can reach the linked private DNS zone. | Correct the private-zone link or conditional forwarding path. |
| DNS returns the expected private IP, but PostgreSQL authentication fails. | Confirm the user name, password, and PostgreSQL role permissions separately from network reachability. | Correct the database credentials or role grants; changing DNS or the private endpoint doesn't grant database access. |

<!-- TODO: [CE3, CE5] SME: add concrete HorizonDB or PostgreSQL client errors for an unapproved private endpoint, a blocked TCP path, and an incorrect network path. For each error, include a diagnostic that distinguishes the cause and an SME-tested psql or driver transcript with endpoint connection-state output. -->

## Examples

A transactional application uses the primary endpoint FQDN for writes. After a failover, the application retries dropped connections against that same FQDN, and HorizonDB routes new connections to the promoted primary. A reporting application uses the reader endpoint FQDN so HorizonDB can balance connections across available standby replicas.

## Non-examples

A fixed private IP in a connection string isn't a substitute for the cluster FQDN. A delegated subnet or VNet-injected cluster also isn't a HorizonDB Private Link design. Finally, PostgreSQL Flexible Server subresource names and private DNS zones aren't templates for HorizonDB because the products use different cluster architectures and FQDN naming.

## Related content

- [Networking overview with public access (allowed IP addresses) and private endpoints in Azure HorizonDB (Preview)](concepts-network-public.md)
- [Connection endpoints in Azure HorizonDB (Preview)](../connectivity/concepts-connection-endpoints.md)
- [Compute replicas in Azure HorizonDB (Preview)](../configure-maintain/concepts-compute-replicas.md)
- [High availability in Azure HorizonDB (Preview)](../high-availability/concepts-high-availability-failover.md)
- [Networking in Azure HorizonDB (Preview)](how-to-network.md)
