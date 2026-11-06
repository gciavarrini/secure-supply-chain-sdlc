# secure-supply-chain-sdlc

Human-in-the-loop CVE remediation as a Red Hat Validated Pattern: ChatOps commands
(`/remediate`, `/generate-tests`) drive Tekton pipelines on OpenShift that detect
vulnerabilities, open remediation pull requests, generate regression tests, and promote
signed builds — with every component (scanner, AI test generation, Git host, registry,
signing) behind a documented, swappable contract.

## The loop

1. **Detect** — the pipeline builds an SBOM, scans it, and opens an issue for every
   vulnerability that actually affects the code (reachability, not just CVE noise).
2. **`/remediate`** — a developer types the command on the issue; the pipeline bumps the
   dependency to a safe (or Lightwell-remediated) version and opens a pull request.
3. **`/generate-tests`** — the AI test generator writes coverage-driven regression tests
   for the change and opens a tests-only pull request.
4. **Build, sign, promote** — after merge: hermetic offline build, cosign signature with
   SLSA L3 provenance, then GitOps promotion dev → test → prod, where pre-prod and prod
   require a human-approved promotion PR.

## Replaceable at every step

Every external tool sits behind a contract with a reference stub, so the whole loop runs
on a fresh cluster with zero external services — then community tools (SonarQube,
cosign/Rekor, Nexus, Vault, Quay) form the working default, and Red Hat products
(RHTPA, RHTAS) ship as values-selected optional tiers. Customers can bring their own
scanner, AI service, Git host, registry, or CI runner by implementing the same contract.

## Status

**Scaffold in progress** (Epic E3.1 of the implementation plan) — structure only,
not yet installable. The design lives with ADR-002 / the Lightwell effort docs; this
repository is the pattern implementation itself.

## Repository layout

| Path | Purpose |
|---|---|
| `pattern.sh` | Framework installer entry point (runs the utility container) |
| `Makefile`, `Makefile-common` | `make install` / `make show` targets |
| `pattern-metadata.yaml` | Pattern metadata: name, tier, cluster requirements, variants |
| `values-global.yaml` | Global values applied to every install |
| `values-secret.yaml.template` | Template for secrets (never commit the real file) |
| `variants/local/` | The single-cluster variant (decision #4) |
| `charts/all/` | The pattern's own charts (contract stubs, ChatOps listener) — E3.2+ |

## Testing locally (no cluster needed)

Before every push, render the pattern with the clustergroup chart and check the
output — this catches values-structure mistakes without an OpenShift cluster:

```bash
git clone --depth 1 https://github.com/validatedpatterns/clustergroup-chart.git /tmp/clustergroup-chart
cd /tmp/clustergroup-chart && helm dependency update && cd -
helm template ssf /tmp/clustergroup-chart \
  -f values-global.yaml -f variants/local/values-local.yaml
```

Green output = the values wiring is structurally sound. Actually installing
(`make install` / `pattern.sh`) requires an OpenShift cluster — operators and Argo CD
are cluster-side. OpenShift Local (CRC) works for development.

## License

[Apache-2.0](LICENSE)
