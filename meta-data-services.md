# Adding Metadata Services to OCI Distribution

A work in progress proposal for adding Metadata Services to the [OCI distribution spec][oci-distribution-spec]

## Table of Contents

## Summary

Every registry has some set of metadata services. Without a common API, there's no means to move content across registries, creating a lack of interchange for [best practices of moving artifacts alongside your deployment environments][registry-best-practices] (from dev through production). It also stalls an ecosystem of tools. If there was a common metadata api across all registries, you could imaging the community building solutions as their investments are leveraged everywhere.

## Prior Art

There are a number of different efforts, including:

- [manifesto from AquaSec](https://blog.aquasec.com/kubernetes-metadata-and-manifesto)
- [grefeas from google](https://grafeas.io/)

Similar to supporting artifact signatures, unless the capabilities are available across all registries, the value prop is minimized. 

With this effort, we hope to:

- Provide a common metadata API across all [OCI distribution-spec][oci-distribution-spec] based registries that implement [OCI Artifacts][oci-artifacts]
- Enable an ecosystem of tools to build upon these metadata services

## Existing metadata on images

While the goal supports any [OCI Artifact][oci-artifacts], there's a couple of additional data elements to consider. 

### Docker Image Labels

Dockerfiles support labels, which can provide name/value pairs

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

## Scenarios

To frame the design, we'll track the following scenarios:

> credit to [manifesto for their use cases](https://github.com/aquasecurity/manifesto#use-cases), and [Liz Rice](https://twitter.com/lizrice) for her insight.

- **Managing QA approval status:** After an image has been built, it needs to go through various testing and approval processes before your organization is ready to use it in production. Keep track of approval status, and who has given sign-off by storing it alongside the artifact itself.  
This information is additive, as the artifact moves through a workflow, with each successful step pushing additional metadata to the registry, associated with the artifact.
- **Storing security profiles for an image**: Make it easy to associate a Seccomp or AppArmor profile with an image, so that you can automatically retrieve the correct profile at the point you want to run a container.
- **Storing vulnerability scan reports**: Artifacts should be scanned regularly for vulnerabilities as new ones may be found in existing code. Enable storing the latest scan report for an artifact without modifying the image itself.  
This information is additive as scan results change over time. Should all scan results be available, captured by date/time?
- **Support contacts**: Store the phone number or Slack channel to contact in the event this artifact starts causing problems in your live deployment. Update these details without needing to update the artifact.
- **Tracking active images**: With CI/CD it's easy to end up with gazillions of artifacts in your registry. Store whether an artifact was tagged as deployed.
- **Tracking last pull**; Knowing an image was set for production deployment is good to separate from the images built and never deployed, but is it still active? This 

## Open Questions

1. Which types of metadata should be signed, vs. arbitrary values?
2. Do we support blobs of content to be indexed, such as an SBOM being submitted as a JSON object, or a scan result? What's a realistic expectation for a registry operator to index all, or a subset of information submitted as a blob? What motivates a customer to not index everything? Not to be defined in the spec, but how do we enable the ability to set expectations on an attribute type, or artifact type to be indexed? A registry may charge for the amount of data they index, promoting a reasonable practice for users to effectively use the feature.

## Scoping

As with any set of name/value pairs, a means to isolate groups will be required:

- properties defined by the customer
- properties defined as a standards (OCI, CNCF, SecureFoundation)
- properties defined by the registry operator

Using namespaces has the most flexibility.

## Requirements

- Property locking - ability to lock the value. This may be locked in the means it was stored (dockerfile label, oci annotation), or a metadata value that is locked by RBAC.


[docker-object-labels]:         https://docs.docker.com/config/labels-custom-metadata/
[docker-labels]:                https://docs.docker.com/engine/reference/builder/#label
[oci-artifacts]:                https://github.com/opencontainers/artifacts
[oci-annotations]:              https://github.com/opencontainers/image-spec/blob/master/annotations.md
[oci-distribution-spec]:        https://github.com/opencontainers/distribution-spec
[oci-image-spec]:               https://github.com/opencontainers/image-spec/
[registry-best-practices]:      https://stevelasker.blog/2018/11/14/choosing-a-docker-container-registry/
