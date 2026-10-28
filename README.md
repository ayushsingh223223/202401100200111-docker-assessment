# FestPass Docker & Container Security Assessment

## Student and release details

| Item | Value |
| --- | --- |
| Name | Ayush Singh |
| REGNO | 202401100200111 |
| PORT | 8011 |
| Docker Hub username | ayushsingh5307 |
| Docker Hub repository | https://hub.docker.com/r/ayushsingh5307/fest-backend |
| Signed release | `ayushsingh5307/fest-backend:1.0.0` |
| 1.0.0 digest | Pending push - record the `sha256:` digest here after Stage 4 |

## Run the stack

1. Copy `.env.example` to `.env` and set a private database password.
2. Run `docker compose up -d`.
3. Browse to http://localhost:8011.

The frontend is isolated from the database: it is on `frontend_net`, while PostgreSQL is only on `backend_net`. The backend belongs to both networks and is the only service that can connect to PostgreSQL. Database data persists in the `festpass-data` named volume.

## Container hardening

The final backend image uses a version-pinned `python:3.13-alpine3.22` base, pinned and upgraded Python dependencies, `--no-cache-dir` package installation, and a dedicated non-root `festpass` user.

## Trivy before-and-after results

Run these commands after Docker Desktop and Trivy are ready:

```powershell
trivy image fest-backend:0.1 | Tee-Object reports/trivy-before.txt
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 ayushsingh5307/fest-backend:1.0.0 | Tee-Object reports/trivy-after.txt
$LASTEXITCODE
```

| Image | CRITICAL | HIGH | Size |
| --- | ---: | ---: | ---: |
| `fest-backend:0.1` | Pending scan | Pending scan | Pending build |
| `fest-backend:0.2` | Pending scan | Pending scan | Pending build |

## AI use declaration

OpenAI Codex was used to help create the Dockerfiles, Docker Compose configuration, README, and to explain the Docker, Trivy, and cosign commands. All commands, screenshots, pushes, scans, and signatures are run from Ayush Singh's own machine and accounts.

## Incomplete steps

- Scan totals, image sizes, Docker Hub digest, public push, cosign key generation, signing, verification, and screenshots will be recorded after Docker Desktop, Trivy, Docker Hub, and GitHub authentication are available.
