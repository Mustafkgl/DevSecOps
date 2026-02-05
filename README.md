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
                 │
                 ├── PASS → commit reports to branch
                 └── FAIL → exit 1 (build blocked)
```

---

## Key Design Decisions

| Decision | Rationale |
|---|---|
| **Zero 3rd-party GitHub Actions** | Eliminates supply-chain attack surface (compromised actions in CI are a known vector — e.g., tj-actions/changed-files incident) |
| **CycloneDX JSON SBOM** | Industry-standard format (NTIA-compliant), machine-parseable, supports downstream vulnerability correlation |
| **SARIF output** | Standard format for security tool results; compatible with GitHub Security tab and external SIEM ingestion |
| **Configurable severity threshold** | Quality gate accepts `none\|low\|medium\|high\|critical` — allows teams to tune acceptable risk per environment (dev vs. prod) |
| **`[skip ci]` on report commits** | Prevents infinite pipeline loop when workflow commits scan results back to the repo |

---

## Tools

| Tool | Role | Output Format |
|---|---|---|
| [Syft](https://github.com/anchore/syft) | SBOM generation | CycloneDX JSON, SPDX |
| [Grype](https://github.com/anchore/grype) | Vulnerability scanning (CVE matching against SBOM) | JSON, SARIF |
| Docker | Image build | — |
| bash + jq | Quality gate logic | Exit code (0 = pass, 1 = fail) |

---

## Quality Gate

The pipeline enforces a configurable severity threshold. Set `SEVERITY_THRESHOLD` in the workflow:

```yaml
env:
  SEVERITY_THRESHOLD: high   # none | low | medium | high | critical
```

If any finding meets or exceeds the threshold, the pipeline exits with code `1`, blocking the build. This mirrors production-grade security gate behavior in enterprise DevSecOps environments.

---

## Workflow File

[`.github/workflows/devsecops-noapi.yml`](.github/workflows/devsecops-noapi.yml)

---

## Target Application