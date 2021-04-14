### Convert the tag to a digest

```bash
curl -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -vvv -k registry.wabbit-networks.io/v2/net-monitor/manifests/v1
curl -H "Accept: application/vnd.oci.image.manifest.v2+json" -vvv -k registry.wabbit-networks.io/v2/net-monitor/manifests/v1
curl -vvv -k registry.wabbit-networks.io/v2/net-monitor/manifests/v1

curl -v -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
    https://mcr.microsoft.com/v2/aks/hcp/etcd-azure/manifests/v3.1.19 2>&1 | \
    grep -i 'Docker-Content-Digest:' | \
    awk '{print $3}'
curl -v -H "Accept: application/vnd.oci.image.manifest.v2+json" \
    https://mcr.microsoft.com/v2/aks/hcp/etcd-azure/manifests/v3.1.19 2>&1 | \
    grep -i 'Docker-Content-Digest:' | \
    awk '{print $3}'

curl -v -H "Accept: application/vnd.oci.image.manifest.v2+json" \
    https://mcr.microsoft.com/v2/aks/hcp/etcd-azure/manifests/v3.1.19 2>&1 | \
    grep -i 'Docker-Content-Digest:' | \
    awk '{print $3}'
   


curl -v -H "Accept: application/vnd.oci.image.manifest.v2+json" \
    registry.wabbit-networks.io/v2/net-monitor/manifests/v1 2>&1 | \
    grep -i 'Docker-Content-Digest:' | \
    awk '{print $3}'
```

### Query for linked artifacts

```bash
GET /v2/_ext/{repository}/manifests/{digest}/links?artifactType=application/vnd.oci.notary.v2.config+json&n=10
GET registry.wabbit-networks.io/v2/_ext/net-monitor/manifests/{digest}/links?artifactType=application/vnd.oci.notary.v2.config+json