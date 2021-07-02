# Enabling Artifact CLIs to Reference Environment Specific Registries Through Configuration

As cloud native development matures, and more focus is placed on securing the supply chain, more focus is being placed on securing the content used for deployments. As outlined in the below article, users wish to consume public content, but not directly from their production resources. They need a means to import, scan, secure the content they depend upon. In addition to the import process, one of the many challenges is how should artifacts be referenced? How are artifact references changed from their public urls, or default docker hub registry, to private registry urls? How are they changed as content may move between dev, staging, production and archival registries and/or repos within a shared registry?

This article outlines the requirements and design options for mapping existing references and parameterizing new artifact references.

### Background

- [Is It Time to Change How We Reference Container Images?](https://stevelasker.blog/2020/10/21/is-it-time-to-change-default-registry-references/)
- [OCI-Consuming Public Content](https://opencontainers.org/posts/blog/2020-10-30-consuming-public-content/)
- [How to reference artifacts that move](https://github.com/notaryproject/nv2/discussions/31)

## Goals

1. Enable existing artifact references to be deterministically re-routed from one `registry/repo` to another `registry/repo`.
2. Define a parameter pattern, enabling environment based configurations for new artifact references, including `defaultRegistry`, `namespace` and `repo` variables.

## Non Goals

1. Provide a fall-through search path. These patterns have led to squatting security vulnerabilities. The design will provide explicit mappings.

## Registry Categories

Registries are broken down into the following categories:

- **Distributor**: An aggregation of multiple vendors and/or users in a single registry.
- **ISV**: A specific software vendor that distributes their content directly from their own registry. ISV may syndicate their content through distributors in addition to hosting their own content.
- **Private Hosted**: Vendor hosted registries, where the user has their own private scoped artifacts. Hosted registries either use a sub-domain or a namespace to provide scoped artifacts.
- **Private Instanced**: The user hosts their own registry on cloud infrastructure or on-premise hardware.

Some examples include: 

| Category | Registry | Example Reference |
|-|-|-|
| Distributor | [Docker Hub](https://hub.docker.com/) | `node:16` |
| Distributor | [GitHub](https://github.com/search?package_type=container&q=nodean&type=RegistryPackages) | `ghcr.io/homebrew/core/node:16` |
| Distributor | [Quay](https://quay.io/search) | `quay.io/calico/node:16` |
| Distributor | [ECR Public](https://gallery.ecr.aws/) | `public.ecr.aws/bitnami/node:16` |
| ISV | [MCR](https://aka.ms/mcr) | `mcr.microsoft.com/windows/servercore:20H2` |
| ISV | [nvidia](https://ngc.nvidia.com/catalog/) | `nvcr.io/nvidia/pytorch:21.06-py3` |
| ISV | [RedHat](https://catalog.redhat.com/software/containers/explore) | `registry.access.redhat.com/rhel7-atomic:latest` |
| Private Hosted | [Docker Hub](https://hub.docker.com/) | `acmerockets/net-monitor:v1` |
| Private Hosted | [ACR](https://aka.ms/acr) | `acmerockets.azurecr.io/net-monitor:v1` |
| Private Hosted | [ECR](https://aws.amazon.com/ecr/) | `acmerockets.dkr.ecr.us-west-2.amazonaws.com/net-monitor:v1` |
| Private Hosted | [GCR](https://cloud.google.com/container-registry) | `gcr.io/acmerockets-191217/net-monitor:v1` |
| Product Instanced | [JFrog](https://jfrog.com/container-registry/) | `registry.acme-rockets.io/net-monitor:v1` |
| Product Instanced | [Project Harbor](https://goharbor.io/) | `registry.acme-rockets.io/net-monitor:v1` |

## Common Registry Workflows

As artifacts are consumed or moved across the above registries, artifacts can be referenced consistently, enabling the registry location as an environmental configuration option.

Users typically have variations on the following workflows:

1. Consume content from a public registry (Distributor or ISV)
2. Import public content to a privately managed registry (Private Hosted or Private Instanced)
3. Build private content in a private registry
4. Promote content within the same private registry, or other private registries for different environments.
5. Promote content to a public registry.


## Package Manager References

Registries are just **Y**et **A**nother **P**ackage **M**anager (YAPM). Most package managers have solve this problem through configuration. 

Most package managers have the ability to alter the default location for where to pull packages. As reference, the following examples are provided:

- [npm](https://docs.npmjs.com/configuring-your-registry-settings-as-an-npm-enterprise-user)  
    - `npm config set registry https://registry.your-registry.npme.io/`  
    - `npm login --scope=@company-scope --registry=https://registry.company-registry.npme.io/`
    - npm uses scopes to identify a location for a package `@org-name/package-name`
- nuget
- pypi
- debian
- rpm

## Scenarios

To ground the design conversations in common use cases, the following scenarios are outlined.

### Scenario 1: Direct Image Reference  

Pat runs [nginx](https://hub.docker.com/_/nginx) in their k8s environment. Pat doesn't think about running a private registry as they're just focused on getting something running in k8s. Since nginx is a docker official image, the reference is simply `nginx`, using the default docker hub registry.  
As Pat realizes nginx should be run from a private registry, alongside their k8s instance, how is the nginx reference changed to use the local instance? 

**Implications:**
- a means to redefine a default registry for images

### Scenario 2: Helm Chart Reference

Pat learns about helm and wishes to use the [helm cli for deploying nginx](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/). The [maintainers of the nginx helm chart have done a great job parameterizing the image reference](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/#installing-via-helm-repository). Pat can use `helm install my-release nginx-stable/nginx-ingress --set controller.image.repository=registry.acme-rockets.io/nginx-plus-ingress` However this isn't consistent across all helm charts, and the `--set` must be made on each chart, making it easy for a user to misconfigure a deployment.

**Implications:**
- a means to change helm chart references through configuration
- a means to change the registry url for the images defined within a helm chart

### Scenario 3: Build Reference

Pat moves on to building and deploying their own image, built from the public [mcr.microsoft.com/dotnet/aspnet](https://hub.docker.com/_/microsoft-dotnet-aspnet/) image. In this case Pat uses a dockerfile to build their image.

The dockerfile would look similar to:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0
COPY bin/Release/net5.0/publish/ App/
WORKDIR /App
ENTRYPOINT ["dotnet", "NetCore.Docker.dll"]
```

Following best practices for consuming public content, Pat should be referencing the `aspnet` image from their private registry. For the `nginx` image, Pat would just need a means to redefine the default registry. While the dockerfile can use [build args](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact), this pattern is limited to docker build. Using public software registries like MCR, Nvidia, Oracle, users will need to remap an existing registry url to another within their build and deployment files.

**Implications:**
- a means to redefine one registry url to another

### Scenario 4: Environmental Promotion

As Pat finishes the building of their `net-monitor` image, they need to promote the image from their dev environment through staging to production. At wabbit-networks, the dev environment is in a VNet which has selective public access. However the staging and production environments are hosted in a private VNet, with no public access. Since Pat is creating the reference for all new services, Pat uses parameters to define the registry and or the namespace within the registry to the `net-monitor` repo.

**Implications:**
- a means to parameterize the path to an artifact

## Configuration Proposal

To start the conversation, the following proposal is made with the following examples supporting:

- [Configuration & Variables](#configuration-and-variables) for new references
- [Mappings](#mappings) for existing references

### Example Artifacts

The following examples are provided as references.
| Location | Initial Reference | Desired Reference |
| - | - |- | 
| Docker Hub | `node:16` | `registry.acme-rockets.io/base-artifacts/node:16` |
| ISV Registry | `registry.wabbit-networks.io/released/net-monitor:v1` |  `registry.acme-rockets.io/base-artifacts/net-monitor:v1` | 
| Internal Company Image | `registry.acme-rockets.io/alpha-team/dev/my-app:1.0` | `registry.acme-rockets.io/staging/my-app:1.0` |
|  | └──>  | `registry.acme-rockets.io/prod/my-app:1.0` |

### Configuration and Variables

To account for configuration based references, a `registry.config` file will include the values, while artifact references may use $(value) to reference values within the `registry.config` file.

### Standard variables:

- `_default`: the default registry to use, when the registry url is not provided. In practice, this value SHOULD be used to redirect docker hub references to an alternative location. Examples include:
  - `base.registry.acme-rockets.io/`
    ```json
    {
      "_default": "base.registry.acme-rockets.io/"
    }
    ```
  - `registry.acme-rockets.io/base-artifacts/`
    ```json
    {
      "_default": "registry.acme-rockets.io/base-artifacts/"
    }
    ```

Using `"_default": "registry.acme-rockets.io/base-artifacts/"` would enable `oras pull node:16` to expand to `oras pull registry.acme-rockets.io/base-artifacts/node:16`

### User Defined Variables

Additional user defined variables may be configured:

```json
{
  "vars": [
    {
      "devRegistry": "dev-registry.acme-rockets.io/",
      "baseArtifacts": "base-registry.acme-rockets.io/",
      "alphaTeamDev": "alpha-team/dev/",
      "betaTeamDev": "beta-team/dev/",
      "staging": "registry.acme-rockets.io/staging/",
      "prod": "registry.acme-rockets.io/prod/"
    }
  ]
```

The following configuration would enable building from the `baseArtifacts` registry, while pushing to the `alpha-team-dev` registry:

**`registry.config`**
```json
{
  "vars": [
    {
      "_default": "registry.acme-rockets.io/base-artifacts/",
      "dev-registry": "dev-registry.acme-rockets.io/",
      "alpha-team-dev": "alpha-team/dev/"
    }
  ]
}
```

**`dockerfile`**
```dockerfile
FROM node:16
...
```

When building, the configuration and variable references would enable:

`docker build . --tag {{devRegistry}}myapp:1` to build an image named: `dev-registry.acme-rockets.io/myapp:1`

## Deterministic Mappings

For existing images, mappings enabled registry/namespace/repo mappings from a source to a target. Tag mappings are not included as these are considered outside the scope of registry mappings goals.

```json
{
  "vars": [
    {
      "_default": "registry.acme-rockets.io/base-artifacts/",
      "dev-registry": "dev-registry.acme-rockets.io/",
      "alpha-team-dev": "alpha-team/dev/"
    }
  ],
  "mappings": [
    {
      "source": "node",
      "target": "{{_default}}node"
    },
    {
      "source": "registry.wabbit-networks.io/released/net-monitor",
      "target": "{{_default}}net-monitor"
    },
    {
      "source": "mysql",
      "target": "{{_default}}databases/mysql"
    }
  ]
}
```

## Precedence

Precedence to mappings will have the following order:

1. Deterministic mappings, which apply variables, including the default registry
2. Default registry mappings

