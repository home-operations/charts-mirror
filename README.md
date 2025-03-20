# OCI Helm Charts Mirror

This is our stop-gap mirror of OCI Helm Charts that can be used until maintainers of upstream charts publish them. See the issue [here](https://github.com/home-operations/charts-mirror/issues/8) for tracking the progress of upstream support for OCI charts added here.

> [!CAUTION]
> If you wish to use these charts understand it is **your responsiblity to make sure to change to the official OCI chart as soon as possible** as they will be deprecated here. I bare **no resposibility** for you **not paying close attention to this repository and the changes herein**.

## Usage

### CLI

```sh
helm install ${NAME} --namespace ${NAMESPACE} oci://ghcr.io/home-operations/charts-mirror/${CHART} --version ${VERSION}
```

### Flux

```yaml
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: ${CHART}
  namespace: ${NAMESPACE}
spec:
  interval: 1h
  layerSelector:
    mediaType: application/vnd.cncf.helm.chart.content.v1.tar+gzip
    operation: copy
  ref:
    tag: ${VERSION}
  url: oci://ghcr.io/home-operations/charts-mirror/${CHART}
  verify:
    provider: cosign
    matchOIDCIdentity:
      - issuer: "^https://token.actions.githubusercontent.com$"
        subject: "^https://github.com/home-operations/charts-mirror.*$"
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: ${NAME}
  namespace: ${NAMESPACE}
spec:
  interval: 1h
  chartRef:
    kind: OCIRepository
    name: ${CHART}
    namespace: ${NAMESPACE}
  values:
...
```

## Contributing

1. Verify the chart doesn't already have an official OCI Helm Chart.
2. Create a new directory under `charts/` with the chart name.
3. Add a `metadata.yaml` to that new directory file with the contents and update the variables to reflect the chart you are adding:

    ```yaml
    ---
    registry: ${REGISTRY_URL}
    chart: ${CHART_NAME}
    version: ${CHART_VERSION}
    ```

4. Open a PR with the link in the description to the upstream issue tracking OCI Helm Chart support.
