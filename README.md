# DevSecOps Security Pipeline

A GitHub Actions CI/CD security pipeline built from scratch with zero 3rd-party actions — a deliberate supply-chain hardening decision.

The pipeline targets [OWASP Juice Shop](https://github.com/mustafkgl/juice-shop) as its intentionally vulnerable application, demonstrating end-to-end container security scanning integrated directly into the build process.

---

## Pipeline Architecture

```
git fetch (manual, no actions/checkout)
       │
       ▼
 Docker build
  juice-shop:$SHA
       │
       ├──► Syft ──► SBOM (CycloneDX JSON + SPDX)
       │
       ├──► Grype ──► Vulnerability scan (JSON + SARIF)
       │
       └──► Quality Gate (bash/jq)