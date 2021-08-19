Nesting is enabled, 

```json
{
  "dev-registry": "dev.acme-rockets.io/",
  "prod-registry": "prod.acme-rockets.io/",
  "baseArtifacts": "$(_default)base-artifacts/",
  "alpha-team-dev": "$(dev-registry)alpha-team/dev/",
  "beta-team-dev": "$(dev-registry)beta-team/dev/",
  "staging": "$(prod-registry)staging/",
  "prod": "$(prod-registry)"
}
```


`oras copy docker.io/library/node:16` expands to
Using the above variables, the following mappings may be made:

| Initial Reference | Output |
| - | - | 
| `$(_default)$(alpha-team-dev)/my-app:1.0` | `registry.acme-rockets.io/alpha-team/dev/my-app:1.0` |
| `$(_default)$(prod)/my-app:1.0` | `registry.acme-rockets.io/prod/my-app:1.0` |
