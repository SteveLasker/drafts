# Proposal: Decoupling Registries from Specific Artifact Specs

When the [distribution-spec][distribution-spec] and [image-spec][image-spec] were created, they were uniquely focused on the container image workflows, which made sense at the time. Some generalization was built in, and it's gotten various implementations to a great steady-state. However, these are now at a point of success, where we need to consider how these efforts move forward.

This proposal brings forward a recognition that:

1. Registries are implementations of the distribution-spec
2. Registries store more than just container images
3. Registries, images and other artifacts are each trying to innovate and evolve independently
4. The definition of how content is stored in a registry is defined in the image-spec, through the [image-manifest][image-manifest] and the [image-index][image-index].
5. Changes require adoption to be effective, and not all implementors of the specs may adopt changes at the same rate.
6. Implementors of these specs account for extensibility, as defined in the versioned specs, and should have a means to opt-into new behaviors, with well defined expectations.
7. The tight coupling of these specs is creating undesired tension between groups that wish to add capabilities without impacting the other.
8. Changes must be capable, but done in a non-breaking means. Non breaking isn't simply defined as adding _optional_ behavior to an existing version, rather the change must be evaluated if use of a new capability will have negative impacts to adopters of a specific version of a spec.

Container images, registries and many of these new artifact types are still in the early phases of adoption. Virtual Machines, the predecessor to virtualized environments became standardized around 2000. 21 years later, adoption continues to grow, as containers mostly sit atop a virtualized VM. The usage of registries and images today are still small to the usage of tomorrow. The sooner we enable specs to evolve independently, the more innovation and usage will be enabled.

## OCI Artifacts Formalize Generic Registry Usage

As cloud native development has proceeded, users were storing additional content in container registries. They just made them look like container images by using the same oci image mediaTypes. While Container Images and Registries are looking to evolve independently, the problem is further exacerbated as other artifact types are also looking for how they can innovate independently, and benefit from new registry capabilities, like signing and establishing references.

The [OCI Artifacts][oci-artifacts] effort formalized the approach for identifying unique types within registries, leveraging the investments cloud providers, vendors and oss projects have provided. Between OCI Artifacts and [ORAS][oras], many new artifact types were enabled as they didn't have to create **Y**et **A**nother **S**torage **S**ervice (YASS) and ORAS (**O**CI **R**egistry **A**s **S**torage) provided a set of libraries to get artifact authors started without having to learn details of the distribution-spec or image-spec. They followed [Artifact Authors guidance][oci-artifact-guidance] for how to uniquely identify their type and they had support from nearly all registry products, services and products. This evolution happened relatively quickly, as the change to registry products was matched with the benefit to their users.

The OCI Artifacts project was created outside of image-spec and distribution-spec as a recognition that distribution was a generic storage services, and _oci images_ were specific _artifact types_. However, the project stopped short of defining a new manifest or added any elements to the spec as maintainers were concerned about commitments to versioned specs, as most of the changes were to be added to the image-spec as the definition for how to store content in a registry. The change was limited to registry implementations opening up their `manifest.config.mediaType` validation.

## OCI Image v2 Requirements

There are a set of folks [brainstorming around OCI Image v2](https://hackmd.io/@cyphar/ociv2-brainstorm). Many of these use-cases will require changes to the image-spec. Some will require to the distribution-spec. Most of the changes will have backwards compat issues for container image tool chains which will need a release valve to enable upgradeability, without breaking downlevel tool chains.

## Interdependency Between Specs

There will continue to be interdependency between various specs. While it's recognized that there will be some big changes needed, I would propose we structure the changes to be aligned with their value. Meaning, we introduce capabilities that enable new efforts, with opt-in behavior. For example, we know images need to be signed. Can we introduce that capability as an added opt-in feature that has zero impact to the current container tool chains and runtimes?

## Decouple the Persistance Spec from the Specific Artifact Spec

- An artifact MUST be capable of being defined in a means that makes sense for that artifact type.
- A registry MUST be capable of defining how it can store content, and serve it through discoverability APIs.

To achieve the above goals, I'd propose:

1. A new manifest for describing content stored in a registry is defined. There are a set of existing and new scenarios we'll want to support.
2. The new manifest defines a set of extensibility points, with clear expectations for what can be added without impacting clients or registries.
3. The new manifest is versioned, enabling various artifact types to validate and test against specific versions.
4. Maniefst and Registry validations are done through branches of [CNCF Distribution][cncf-distribution], enabling a baseline for testing designs, independent of any cloud provider or project. Artifact authors can use these branches to validate their types.
5. Cloud providers, vendors and products may implement anything they want, including pre-released versions of the evolving spec, or alternative designs they wish to validate and bring back to the group.

## Evolve OCI Artifacts, Distribution or Merge

Based on the currently known requirements, changes to any registry conforming to the distribution-spec will need to change. OCI Artifacts was only created to decouple changes to the existing specs, basically avoiding this refactoring. Now that we've validated a refactoring is required, I'd propose OCI Artifacts and the distribution-spec merge.

How they merge can be an evolved process. The current [oci.artifact.manifest proposal](https://github.com/opencontainers/artifacts/pull/29) defines a new manifest and a new references api. There's an even more [generic object spec PR](https://github.com/opencontainers/artifacts/pull/37) taking shape that attempts to solve the broader artifact versioning problem, including the challenges faced by the OCI Image v2 working group.

Considering distribution is on the cusp of a V1, I'd propose the distribution group works under OCI Artifacts repo until the proposal is declared stable enough by the OCI Artifacts, Distribution-spec, image-spec and OCI TOB maintainers and members. All efforts under the OCI Artifacts repo will be done with the full expectation of merger.

Alternatively we can just merge them after distribution-spec declares V1, and innovate under the distribution-spec repo, with a draft-v2 label.

## Refreshing Maintainers

The various projects need active maintainers. To that end, I'd also suggest the various projects need some charters to enable this decoupling in a stable way.

- The image-spec maintainers must be able to innovate the container image artifact type.
- The registry operators, products and projects must be able to innovate in distribution to store, discover and distribute all artifact types, including container images.
- While the two efforts are currently coupled by the image-spec, one set of maintainers must not force a change on the other. The sets of maintainers must collaborate on common goals, including the decoupling to enable independent innovation.

[cncf-distribution]:      https://github.com/distribution/distribution
[distribution-spec]:      https://github.com/opencontainers/distribution-spec
[image-index]:            https://github.com/opencontainers/image-spec/blob/master/image-index.md
[image-manifest]:         https://github.com/opencontainers/image-spec/blob/master/manifest.md
[image-spec]:             https://github.com/opencontainers/image-spec
[oci-artifacts]:          https://github.com/opencontainers/artifacts
[oras]:                   https://github.com/deislabs/oras
[oci-artifact-guidance]:  https://github.com/opencontainers/artifacts/blob/master/artifact-authors.md
