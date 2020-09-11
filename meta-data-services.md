# Adding Metadata Services to OCI Distribution

A work in progress proposal for adding Metadata Services to the [OCI distribution spec][oci-distribution-spec]

## Summary

Every registry has some set of metadata services. Without a common API, there's no means to move content across registries, creating a lack of interchange for [best practices of moving artifacts alongside your deployment environments][registry-best-practices]. With a common metadata API across all registries, you could imagine the community building a suite of projects as their investments would be leveraged everywhere.  

This document outlines a set of discussions for what would evolve into a metadata-spec enhancement to the [OCI distribution spec][oci-distribution-spec].

## Table of Contents

- [Adding Metadata Services to OCI Distribution](#adding-metadata-services-to-oci-distribution)
  - [Summary](#summary)
  - [Table of Contents](#table-of-contents)
  - [Prior Art](#prior-art)
  - [Goals](#goals)
    - [Non Goals](#non-goals)
  - [Scenarios](#scenarios)
  - [Open Questions](#open-questions)
  - [Existing metadata on images](#existing-metadata-on-images)
    - [Docker Image Labels](#docker-image-labels)
    - [OCI distribution-spec Annotations](#oci-distribution-spec-annotations)
  - [Scoping](#scoping)
    - [Scoping with Namespaces](#scoping-with-namespaces)
  - [Metadata Applications](#metadata-applications)
  - [Metadata Values](#metadata-values)
    - [strings](#strings)
    - [Specific datatypes](#specific-datatypes)
    - [Collections](#collections)
    - [JSON Data Structure](#json-data-structure)
  - [Metadata Attributes](#metadata-attributes)
  - [Metadata RBAC](#metadata-rbac)
  - [Requirements](#requirements)

## Prior Art

There are a number of different efforts, including:

- [manifesto from AquaSec](https://blog.aquasec.com/kubernetes-metadata-and-manifesto)
- [grefeas from google](https://grafeas.io/)

Both efforts have demonstrated the need for metadata. However, neither project has risen to common adoption across all registries. Unless the capabilities are available across all registries, the value prop is minimized. Similar to Notary v2, we hope defining a reasonably scoped metadata service enhancement across registries, adoption will grow.

## Goals

With this effort, we hope to:

- Provide a common metadata API across all [OCI distribution-spec][oci-distribution-spec] based registries that implement [OCI Artifacts][oci-artifacts]
- Enable an ecosystem of tools to build upon these metadata services
- Support write, read and query capabilities, enabling the ability to discover artifacts with given metadata values

### Non Goals

- Sign every metadata value

## Scenarios

To frame the design, we'll track the following scenarios:

> credit to [manifesto for their use cases](https://github.com/aquasecurity/manifesto#use-cases), and [Liz Rice](https://twitter.com/lizrice) for her insight.

- _**Searching Metadata:** -placeholder- in addition to/and including all the scenarios below, capture the ability to find a list of artifacts, based on searching metadata_
- **Managing QA approval status:** After an image has been built, it needs to go through various testing and approval processes before your organization is ready to use it in production. Keep track of approval status, and who has given sign-off by storing information alongside the artifact.  
This information is additive, and sometimes updated, as the artifact moves through a workflow, with each successful step pushing additional metadata to the registry, associated with the artifact.
- **Storing security profiles for an image**: Make it easy to associate a Seccomp or AppArmor profile with an image, so that you can automatically retrieve the correct profile at the point you want to run a container.
- **Storing vulnerability scan reports**: Artifacts should be scanned regularly for vulnerabilities as new ones may be found in existing code. Enable storing the latest scan report for an artifact without modifying the image itself.  
This information is additive as scan results change over time. Should all scan results be available, captured by date/time?
- **Support contacts**: Store the phone number or Slack channel to contact in the event this artifact starts causing problems in your live deployment. Update these details without needing to update the artifact.
- **Tracking active images**: With CI/CD it's easy to end up with gazillions of artifacts in your registry. Store whether an artifact was tagged as deployed.
- **Tracking last pull**; Knowing an artifact was set for production deployment is good to separate from the images built and never deployed, but is it still active? These values may be managed by the registry, and surfaced through a common metadata API, enabling users to write queries across a set of metadata
- **Role Based Access Control**: does the identity of the entity making the query have a given set of access rights?
- **Features of a registry and/or repository**: As users configure their registry, a set of capabilities may be available. To enable common developer workflows, the registry operator may surface properties to the overall registry, repository, tag or digest.
- **Copying with the Artifact**: As artifacts are copied from one registry to another, a set of metadata, such as the `sourceCode` location, the `maintainers`, `artifactType` or initial `creationDate` may be copied with the artifact. Other values specific to the copy in that registry, such as the `pullCount` may not be applicable to being copied.
- **Docker Labels & OCI Annotations**: Existing metadata is already defined for Docker and OCI image-spec artifacts. Through a common metadata API, users can discover all metadata values in a common API.
  
## Open Questions

1. Which types of metadata should be signed, vs. arbitrary values?
2. Do we support blobs of content to be indexed, such as an SBOM being submitted as a JSON object, or a scan result? What's a realistic expectation for a registry operator to index all, or a subset of information submitted as a blob? What motivates a customer to not index everything? Not to be defined in the spec, but how do we enable the ability to set expectations on an attribute type, or artifact type to be indexed? A registry may charge for the amount of data they index, promoting a reasonable practice for users to effectively use the feature.

## Existing metadata on images

While the goal supports any [OCI Artifact][oci-artifacts], there's a couple of additional data elements to consider.  Using namespaces, we believe we can surface this information in a common metadata API.

### Docker Image Labels

Dockerfile supports labels, which can provide name/value pairs

**Pros:**

- Baked into the image, which becomes immutable as the image always has the same label/value regardless of where it's instanced

**Cons:**

- Baked into the image, which means different instances can't have different values

Good usage scenarios:

- **git hash** for the source of the build
- **build id** for the specific build
- **general build info** as it seems relevant to have build info in the built artifact

### OCI distribution-spec Annotations

The [OCI image-spec][oci-image-spec] defines a set of [OCI annotations][oci-annotations]. While these are a bit more generic than dockerfile labels, they have similar constraints to dockerfile labels.

**Pros:**

- Baked into the manifest pushed to a registry, which becomes immutable.
- Becomes signed, along with the artifact as the values are in the signed manifest.

**Cons:**

- Basked into the manifest, which means different instances can't have different values
- Unable to update values as information related to the artifact changes over time.

## Scoping

As with any set of name/value pairs, a means to isolate groups will be required:

- properties defined by the customer
- properties defined as a standards (OCI, CNCF, ...)
- properties defined by the registry operator

Using namespaces, users, organizations and operators will have the ability to scope their name/value pairs within a larger ecosystem.

### Scoping with Namespaces

Name/Value pairs are scoped with namespaces, enabling different groups to represent their name/values uniquely.

Using a similar pattern as [oci annotations][oci-annotations], `org` and vendor (`vnd`) namespaces would be maintained.

- `org.opencontainers.artifact.created (dateTime)` date and time on which the image was built (string, date-time as defined by [RFC 3339](https://tools.ietf.org/html/rfc3339#section-5.6).
- `org.opencontainers.artifact.authors (semicolon delimitated string)` contact details of the people or organization responsible for the image (freeform string)
- `org.opencontainers.artifact.url (url)` provides more information on the image
- `org.opencontainers.artifact.documentation (url)` URL to get documentation on the image
- `org.opencontainers.artifact.source (url)` URL to get source code for building the image
- `org.opencontainers.artifact.type (string)` the manifest/index.mediaType (string) eg: application/vnd.opencontainers.image.
- `org.opencontainers.artifact.transferable (bool)` - a boolean property indicating if the property is transferable across registries when copied. `pullCount`, `teleportEnabled` are types of properties that wouldn't apply when an artifact is copied as they represent state for the given registry.

Using this pattern, a registry may expose their own unique values:

- `vnd.microsoft.azure.registry.pullCount (number)` - if not surfaced as a `org.opencontainers` property, a registry might surface pullCounts
- `vnd.docker.certified (bool)` - a boolean property indicating if the artifact was docker certified

Customers may choose their own namespaces, or simply use the root:

- `acme-rockets.marketing.campaignId` - a specific attribute to the Acme Rockets marketing team, to know which marketing campaign the artifact was associated with.
- `buildId` - a generic buildId, not scoped to Open Containers, CNCF, or other groups.

As the spec evolves, a set of `org.opencontainers.*` names will be provided. Similar to OCI Artifacts, other organizations may submit their namespaces through an IANA.org registration.

## Metadata Applications

Metadata may be assigned to any scope of objects within a registry, including:

- The registry: `registry.acme-rockets.io`
- A repository within the registry: `registry.acme-rockets.io/net-monitor`
- A tag and/or digest, within a repository: `registry.acme-rockets.io/net-monitor:v1`

Examples:

- Registry: `registry.acme-rockets.io`: `vnd.microsoft.azure.registry.features.replications (string array)` - is the registry geo-replicated. This shows a collection scenario.
- Repository: `registry.acme-rockets.io/net-monitor`: `vnd.microsoft.azure.registry.features.teleport (bool)` - is the artifact, or repository enabled for [Azure Teleportation](https://aka.ms/acr/teleport)

## Metadata Values

Values range from simple strings, including numbers and dates persisted as strings, to structured objects.
What are the range of types to be supported?

`org.opencontainers.artifact.authors` contact details of the people or organization responsible for the image (freeform string)

- String  
  `org.opencontainers.artifact.source` =  
  ```
  "https://github.com/notaryproject/nv2"
  ```
- Collection Strings  
  `vnd.microsoft.azure.registry.features.replications` =  
  ```
  "eastus", "westus", "westeu"
  ```
- Date using [ISO 8601 standard](http://en.wikipedia.org/wiki/ISO_8601)  
  `vnd.microsoft.azure.registry.expirationDate` =  
  ```
  "2021-03-27T09:00Z"
  ```
- Bool  
  `org.opencontainers.artifact.transferable` =  
  ```
  "false"
  ```
- Collection Strings, as JSON  
  `vnd.microsoft.azure.registry.features.replications` =  
  ```json
  {
    [
      "eastus",
      "westus",
      "westeu"
    ]
  }
  ```
- Collection of contacts option 1:  
  `org.opencontainers.artifact.authors` =
  ```json
  {
    "contacts": [
      {
        "name":"Steve Lasker",
        "email":"stevenlasker@hotmail.com",
        "gitHubId":"stevelasker",
        "company":"microsoft"
      },
      {
        "name":"Justin Cormack",
        "email":"justin.cormack@docker.com",
        "gitHubId":"justincormack",
        "company":"docker"
      }
    ]
  }
  ```
- Collection of contacts option 2:  
  `org.opencontainers.artifact.authors` =
  ```json
  {
    [
      {
        "name":"Steve Lasker",
        "email":"stevenlasker@hotmail.com",
        "gitHubId":"stevelasker",
        "company":"microsoft"
      },
      {
        "name":"Justin Cormack",
        "email":"justin.cormack@docker.com",
        "gitHubId":"justincormack",
        "company":"docker"
      }
    ]
  }
  ```
- Graph of values, as JSON  
  `vnd.securefoundation.scanresults` =  
  ```json
  {
    "lastScan":"2020-05-10T10:00Z",
    "low":"5",
    "medium":"2",
    "high":"1",
    "critical":"0",
    "detailedReport":"example.io/scanresult{identityToken}"
  }
  ```
- Nested Collections, as JSON  
  `vnd.microsoft.azure.registry.deployed.to` =  
  ```json
  {
    [
      "AKS": [
      "westus",
      "westeu",
      "norwayeast"
      ],
      "ACI": [
        "westeu",
        "japaneast"
      ]
    ]
  }
  ```

### strings

The simplest persistance is a simple string. The string could also represent a number, date or boolean. Simple strings are likely too simple.

### Specific datatypes

To expand on strings, we could support other intrinsic types, like dateTime, number, bool

### Collections

In addition to strings or specific dataTypes, many scenarios call for a collection of values.

- approvers for the promotion of the artifact
- list of locations the artifact is believed to be deployed

### JSON Data Structure

An alternative approach would be to support JSON data structures as the value. This could provide strings, specific datatypes and collections in one structure.

- collections of strings/numbers/dates
- json objects, which would support dotted query notation
- files

## Metadata Attributes

For any property, a set of attributes that define its capabilites may be required:

- `readOnly` - the property is maintained by a backend system of the registry, such as `pullCount`, `created`, `lastTagUpdate`
- `writeOnce` - the property can be set, but never updated
- `readWrite` - the property is updatable
- `created` - when the property was initially set
- `lastUpdate` - when the property was last updated
- `history` - a collection of historical values. This is likely a reserved name that registry operators may choose to implement.

## Metadata RBAC

Registry operators will likely want to support Role Based Access Control to specific properties. For instance, the identity assigned to a build system may set the `buildId` or `tagLocked` property. Another user can query the `buildId` property, but can't edit it.

Similar to [MeataData Attributes](#metadata-attributes), a set or RBAC roles may be exposed. The challenge here will be finding a set of role names that are consistent across registries.

Based on the identity of the user requesting:

- `canRead (bool)`
- `canWrite (bool)`
- `canUpdate (bool)`
- `canDelete (bool)`

## Requirements

- Property locking - ability to lock the value. This may be locked in the means it was stored (dockerfile label, oci annotation), or a metadata value that is locked by RBAC.
- Property movement - a set of properties would be relevant to moving/copying between repositories or registries, while others are not relevant. The metadata services must be able to identify which properties would move, while others not. A default of copy is likely appropriate.

[docker-object-labels]:         https://docs.docker.com/config/labels-custom-metadata/
[docker-labels]:                https://docs.docker.com/engine/reference/builder/#label
[oci-artifacts]:                https://github.com/opencontainers/artifacts
[oci-annotations]:              https://github.com/opencontainers/image-spec/blob/master/annotations.md
[oci-distribution-spec]:        https://github.com/opencontainers/distribution-spec
[oci-image-spec]:               https://github.com/opencontainers/image-spec/
[registry-best-practices]:      https://stevelasker.blog/2018/11/14/choosing-a-docker-container-registry/
