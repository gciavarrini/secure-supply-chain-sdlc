# Pattern-owned charts

This directory holds the charts the pattern itself owns — everything that implements
our contracts rather than deploying a third-party component:

- **contract stubs** (E1.6): the reference implementations behind each seam — scanner
  stub, AI test-generator stub, ABI-check stub — so the whole loop runs with zero
  external services (the Sandbox baseline)
- **ChatOps machinery** (E3.6): Tekton trigger/event-listener definitions routing
  `/remediate` and `/generate-tests`
- **pipeline tasks** (E3.2–E3.5): detection, remediation, test-gen, build & sign

Third-party components (cluster-pipelines, gitea, quay, sonarqube, nexus, vault) are
NOT charted here — they are referenced as VP/org chart dependencies from the
clustergroup values (see `variants/local/values-local.yaml`).

Each chart is referenced from the clustergroup by path, e.g.:

```yaml
applications:
  scanner-stub:
    name: scanner-stub
    namespace: ssf-pipelines
    argoProject: ssf-pipelines
    path: charts/all/scanner-stub
```
