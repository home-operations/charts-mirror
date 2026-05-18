# OCI Helm Charts Mirror

This is our stop-gap mirror of OCI Helm Charts that can be used until maintainers of upstream charts publish them. See the issue [here](https://github.com/home-operations/charts-mirror/issues/8) for tracking the progress of upstream support for OCI charts added here.

> [!CAUTION]
> **Subscribe to the upstream issues or PRs tracking OCI support** because if you wish to use these charts understand it is **your responsiblity to make sure to change to the official OCI chart as soon as possible** as they will be deprecated here. I bear **no resposibility** for you **not paying close attention to this repository and the changes herein**. Once there is support upstream the OCI charts will remain published to this repo for 6 months, after which they will be pruned.

## Usage

### CLI

```sh
helm install ${RELEASE_NAME} --namespace ${NAMESPACE} oci://ghcr.io/home-operations/charts-mirror/${CHART_NAME} --version ${CHART_VERSION}
```

### Flux

> [!WARNING]
> Even though these charts are signed via cosign it will not prevent against malicious code being pushed from upstream ending up in a release here. For example if cert-managers Helm chart is compromised, there's nothing stopping that release from **NOT** being mirrored here.

```yaml
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: ${CHART_NAME}
  namespace: ${NAMESPACE}
spec:
  interval: 1h
  layerSelector:
    mediaType: application/vnd.cncf.helm.chart.content.v1.tar+gzip
    operation: copy
  ref:
    tag: ${CHART_VERSION}
  url: oci://ghcr.io/home-operations/charts-mirror/${CHART_NAME}
  verify:
    provider: cosign
    matchOIDCIdentity:
      - issuer: ^https://token.actions.githubusercontent.com$
        subject: ^https://github.com/home-operations/charts-mirror.*$
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: ${RELEASE_NAME}
  namespace: ${NAMESPACE}
spec:
  interval: 1h
  chartRef:
    kind: OCIRepository
    name: ${CHART_NAME}
    namespace: ${NAMESPACE}
  values:
...
```

## Contributing

To add a new chart to this repository:

1. **Check for an existing OCI Helm Chart**

   Confirm that the application you want to add does **not** already provide an official OCI Helm Chart.

2. **Create a chart directory**

   Make a new directory under `apps/` named after the chart.

3. **Add chart metadata**

   Inside the new directory, create a `metadata.yaml` file. The `source:` block tells the builder where to fetch the chart from — set exactly one of `helm` or `git`.

   **Helm-sourced** (the upstream publishes a traditional Helm repository):

   ```yaml
   ---
   artifactName: <name of the published chart>
   source:
     helm:
       registry: <upstream Helm repository URL>
       version: <upstream chart version>
       # chart: <upstream chart name>   # optional, defaults to artifactName
   ```

   **Git-sourced** (the upstream ships chart manifests in a git repository but does not publish a Helm chart):

   ```yaml
   ---
   artifactName: <name of the published chart>
   source:
     git:
       repository: <upstream git repository URL>
       path: <path to the chart directory within the repository>
       tag: <git tag to clone>
   ```

   The chart is published to GHCR under `artifactName`, with any leading `v` stripped from `source.helm.version` (or `source.git.tag`) so the OCI tag is SemVer-clean. The upstream version/tag is still fetched verbatim.

4. **Request upstream OCI support**

   If the upstream project does not yet publish OCI Helm Charts (or any Helm chart at all, for git-sourced entries), open an issue in their application or chart repository requesting OCI Helm Chart support.

5. **Submit a pull request**

   Open a PR in this repository:
   - Include the link to the upstream issue (from step 4) in the PR description.
   - Ensure your PR only adds the new chart directory and metadata.
