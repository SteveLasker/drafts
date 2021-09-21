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


```bash
curl -u "$username:$password" -X PUT --data-binary "@modified.json" -H "Content-Type: application/vnd.oci.image.manifest.v1+json" "https://$registry/v2/oras/empty/manifests/modified2"
curl -u "$username:$password" -H "Accept: application/vnd.oci.image.manifest.v1+json" "https://$registry/v2/oras/empty/manifests/modified2" | jq


curl -u "$username:$password" -X PUT --data-binary "@artifact.json" -H "Content-Type: application/vnd.oci.artifact.manifest.v1+json" "https://$registry/v2/artifact/manifests/test"

DIGEST=$(oras discover $IMAGE -o json | jq -r .digest)
DIGEST=`docker images --digests --filter=reference=${IMAGE} --no-trunc --format "{{.ID}}"`
```

### Query for linked artifacts

```bash
GET /v2/_ext/{repository}/manifests/{digest}/links?artifactType=application/vnd.oci.notary.v2.config+json&n=10
GET registry.wabbit-networks.io/v2/_ext/net-monitor/manifests/{digest}/links?artifactType=application/vnd.oci.notary.v2.config+json