# OCI Artifact Goals

## Goals

1. Leverage cloud provider implementations of production secured, reliable registries to distribute other artifact types
2. Make it so easy for the community to define new types, without registries having to know or do anything to support them
    * Just as users can save any filetype to their local computer, with the optional behavior of getting nice icons and localized display text
3. Support individual artifact types like:
     * platform specific container images
     * helm charts
     * singularity
     * wasm
4. Support collection artifact types
     * Multi-arch container images
     * CNAB which represent an invocation image, and a collection of artifacts that could include
       * container images
       * helm charts
       * terraform templates
5. Support artifacts that reference other artifacts
     * Notary Signatures - which reference the thing they are signing
     * SBoMs that associate with the thing they are documenting
6. Minimize potential impact to the existing container runtimes that *assume* content coming from a registry must be a runnable image
7. Support registry scanning solutions that need to know what the artifact is to properly identify if it's secure, vulnerable or not supported

## Work in Place

* What we've accomplished thus far with [OCI Artifacts][oci-artifacts] addresses #3, the individual artifact types.
* To minimize breaking changes to existing registries, we settled on using the `manifest.config.mediaType` to differentiate "they type".

## Minimizing Breaking Changes - What a Registry Knows About

Today, registries process manifest and index submissions through a well known and orchestrated set of events

A registry client must:

1. Upload blobs (layers)
2. Upload optional config blobs
3. Upload a manifest that references blobs (as layers) and optional config blobs

To support Index, the registry must

1. Do everything above, _and_ accept a manifest-list/index that also must reference existing manifests or other indexes that reference layer or config blobs

To support individual artifacts, a registry only needed to open up the `manifest.config.mediaType` validations as many/most registries would only accept the required [image-spec mediaTypes][oci-image-spec-mediaTypes]:

* `application/vnd.oci.image.index.v1+json`: Image Index
* `application/vnd.oci.image.manifest.v1+json`: Image manifest

To support additional individual artifact types, they just needed to open up the config validation.

## The Dreaded Garbage Collector

One of the biggest challenges registries face is how they de-dupe, reference count and eventually garbage collect incomplete/failed artifact push events, or customer intended deletions. At the pace customers automate the pushing of content, with their requirement to maintain a subset for compliance requirements, minimizing the impact to garbage collection is a key challenge.

Since OCI Artifacts (v1) uses the existing manifest validation on push, there were no changes required for registries to support additional artifact types. All the validations, reference counting and garbage collection remains unchanged.

## Supporting Collection and Reference Artifact types

The conversations for supporting collections has [been focused on leveraging OCI-Index](https://github.com/notaryproject/nv2/blob/prototype-1/docs/distribution/persistance-discovery-options.md) as it already supports cross artifact references (an index tracks a collection of manifests or other indexes). Thus, no impact to garbage collection.

However, as noted in these conversations, the number of API calls it will take to:

1. Start with a tag `registry.acme-rockets.io/wabbit-networks/net-monitor:v1`
2. Convert the tag to a digest
3. Request for referenced objects of said digest, hopefully filtering down to `application/vnd.notary.config.v2+jwt` artifact types
4. Iterating through the collection to identify which signatures exist, and if the one(s) the client cares about exists
5. For each descriptor returned, fetch the config object which contains the signature

The [current proposal (*Options 2*)](https://github.com/notaryproject/nv2/blob/prototype-1/docs/distribution/persistance-discovery-options.md#signature-persistence---option-2-oci-index) for storing the signature in a `index.config` minimizes the calls as the referenced descriptor in the returned collection is an index, so we just need to find the config of that index.

[Option 3](https://github.com/notaryproject/nv2/blob/prototype-1/docs/distribution/persistance-discovery-options.md#signature-persistence---option-3-oci-manifest-linked-through-oci-index) where we store the signature as another manifest, linked through an index is quite overkill.

Additional options:


[oci-artifacts]:    https://github.com/opencontainers/artifacts
[oci-image-spec-mediaTypes]:  https://github.com/opencontainers/image-spec/blob/master/media-types.md