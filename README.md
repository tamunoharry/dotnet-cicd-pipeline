# .NET DevSecOps CI/CD Pipeline

A GitHub Actions pipeline for .NET 8 applications that integrates security scanning at every stage — not just at the end.

## Pipeline Stages

```
Push / PR
    │
    ├── 1. Build & Test ──── xUnit + code coverage
    │
    ├── 2. Security Scan ─── Gitleaks (secrets) + Trivy (dependencies) + NuGet CVE check
    │
    ├── 3. Docker Build ──── Container image build + Trivy image scan → GitHub Security tab
    │
    └── 4. Deploy ────────── SSH deploy to staging (main branch only)
```

## Security Tools

| Tool | What it catches |
|------|----------------|
| Gitleaks | Secrets and credentials accidentally committed |
| Trivy (filesystem) | Vulnerable NuGet packages, misconfigs |
| Trivy (image) | OS-level CVEs in the container image |
| `dotnet list package --vulnerable` | Known CVEs in direct + transitive dependencies |

Results from the container scan are uploaded to the GitHub Security tab as SARIF.

## Dockerfile

The included Dockerfile follows security best practices:
- Multi-stage build (SDK not shipped in final image)
- Runs as a non-root user
- Minimal `aspnet` runtime base image

## Setup

1. Fork or copy `.github/workflows/pipeline.yml` into your repo
2. Set repository secrets:
   - `STAGING_HOST` — your staging server IP
   - `STAGING_USER` — SSH username
   - `STAGING_SSH_KEY` — private key (store public key on server)
3. Push to `main` to trigger the full pipeline

## Local Security Scan

```bash
# Install Trivy
brew install trivy  # macOS
# or: https://aquasecurity.github.io/trivy/latest/getting-started/installation/

# Scan your project
trivy fs .

# Scan a built image
trivy image your-app:latest
```

## License

MIT
