---
title: include file
description: include file
services: container-registry
author: stevelas

ms.service: container-registry
ms.topic: include
ms.date: 09/20/2021
ms.author: stevelas
ms.custom: include file
---
# Registry Capabilities

Registries are production secured and reliable resources. New capabilities are continually being added to improve the security, performance, reliability and usability of them from development through production.

While the features below are specific to the [Azure Container Registry][acr], all registry products either have variations on the [capabilities below](#registry-capabilities--features), or have a backlog to achieve them.

## Artifacts

When considering building a new artifact type, consider what focus you wish to bring to your users. Are you building a new package manager they must run and maintain? Or, are you building a new artifact, you want managed within a service they already run? 

[OCI Artifacts][oci-artifacts] enables existing OCI Distribution based registries to store and manage artifacts, while [ORAS Artifacts][oras-artifacts] continues investments by enabling references between artifacts. The references enable graphs of content to be persisted, managed and promoted across registries.

## Artifact Services

When building new artifacts services, such as secure supply chain services, you might wonder where to start. There are many types of package managers available from npm, debian, nuget, and many more. Most of which have been underfunded to serve the complexities of capabilities listed below. While registries are far short of being a complete solution, they do offer a rich base of capabilities that nearly every user and customer has instanced. While one could suggest a new more focused service should be built for task ___, you might also ask, do I really need to start from scratch, or should I extend what already exists?

For more context, see [Artifact Services, the Case for a Generalized Package Manager](https://stevelasker.blog/2021/08/26/artifact-services/)

## Registry Capabilities & Features

| Capability | Feature |
| - | - |
| **Security** | |
| Private Networking | [Private Links][private-links]
| Sovereignty | [Geo-replication][geo-replication] |
| Limit public endpoints | [Firewall Rules][firewall-rules]
| Data exfiltration | [Dedicated data endpoints][dedicated-data-endpoints] |
| Double encryption at rest with Customer Managed Keys | [Customer Manage Keys][cmk] |
| Secure by default content | [Quarantine][quarantine] |
| Policy management | [Policy management][policy] | 
| Integrated Security | [Azure AD Objects][authentication]
| IoT/Device Security | [Token Support][tokens]
| Repo Granularity | [Repo Scoped Permissions][repo-permissions] |
| Public/Anonymous Access | [Anonymous access][anonymous-access] |
| Locking a tag to a digest | [Tag locking][tag-locking]
| Artifact Signing | [Docker Content Trust][docker-content-trust] & [Notary v2][notary-v2] |
| Automated content updating | [Base Image Update Notifications][base-image-updates] |
| **Reliability** | |
| Cross region redundancy | [Geo-replication][geo-replication] |
| In-region redundancy | [Availability Zones][az] |
| **Performance** | |
| Locality | [Geo-replication][geo-replication] |
| In-network performance | [Telport][teleport]
| **Diagnostics & Troubleshooting** | | 
| Diagnostics & Audit Logs | [Diagnostics & Audit Logs][diagnostics-logs] |
| Health Check | [Health Check CLI][health-check] |
| **Content Management** | |
| Auto-purging of aged out content | [Auto Purge][auto-purge] |
| **Workflows** | |
| Asynchronous notifications | [Regional Webhooks][webhooks] & [Event Grid][event-grid] |
| Server side content promotion | [Import][import] |

[acr]:                      https://aka.ms/acr
[anonymous-access]:         https://docs.microsoft.com/en-us/cli/azure/acr?view=azure-cli-latest#az_acr_update-optional-parameters
[auto-purge]:               https://aka.ms/acr/auto-purge
[authentication]:           https://aka.ms/acr/authentication
[az]:                       https://aka.ms/acr/availability-zone
[base-image-updates]:       https://aka.ms/acr/tasks
[cmk]:                      https://aka.ms/acr/cmk
[dedicated-data-endpoints]: http://aka.ms/acr/dedicated-data-endpoints
[diagnostics-logs]:         https://aka.ms/acr/audit-logs
[docker-content-trust]:     https://aka.ms/acr/content-trust
[event-grid]:               https://docs.microsoft.com/azure/container-registry/container-registry-event-grid-quickstart
[firewall-rules]:           https://aka.ms/acr/vnet
[geo-replication]:          https://aka.ms/acr/geo-replication
[health-check]:             https://aka.ms/acr/health-check
[import]:                   https://aka.ms/acr/import
[notary-v2]:                https://github.com/notaryproject/notation
[oci-artifacts]:            https://github.com/opencontainers/artifacts
[oras-artifacts]:           https://github.com/oras-project/artifacts-spec/
[policy]:                   https://aka.ms/acr/azurepolicy
[private-links]:            https://aka.ms/acr/private-link
[quarantine]:               https://aka.ms/acr/quarantine
[repo-permissions]:         https://aka.ms/acr/repo-permissions
[tag-locking]:              https://aka.ms/acr/tag-locking
[teleport]:                 https://aka.ms/acr/teleport
[tokens]:                   https://aka.ms/acr/tokens
[webhooks]:                 https://aka.ms/acr/webhooks
