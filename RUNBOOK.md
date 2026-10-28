# Assessment command runbook

Run these commands from a normal PowerShell window (not the Codex terminal), because Docker Desktop grants that interactive shell access to its engine.

```powershell
Set-Location 'C:\Users\ayush\Documents\Codex\2026-10-28\t\202401100200111-docker-assessment'
function prompt { "[202401100200111] PS $(Get-Location)> " }
```

Before beginning, create an empty **public** GitHub repository named `202401100200111-docker-assessment` under `ayushsingh223223`, and run:

```powershell
docker login
trivy --version
cosign version
```

Install Trivy and Cosign before continuing if either command is unavailable. Use the official installation documentation for your Windows setup.

## Stage 1 - baseline backend

The first commit intentionally contains the Python 3.9 baseline Dockerfile. Build it from an isolated worktree without changing the final source tree:

```powershell
git worktree add ..\festpass-stage1 a896fff
Set-Location ..\festpass-stage1
function prompt { "[202401100200111] PS $(Get-Location)> " }
docker build -t fest-backend:0.1 .\backend
docker image ls fest-backend:0.1
docker run -d --name backend -p 8011:5000 -e REGNO=202401100200111 fest-backend:0.1
docker ps
curl.exe http://localhost:8011/api/health
docker logs backend
docker rm -f backend
Set-Location ..\202401100200111-docker-assessment
function prompt { "[202401100200111] PS $(Get-Location)> " }
```

Take `S1-2.png` at the image-list command and `S1-3.png` while `docker ps`, the health response, and logs are visible.

## Stage 2 - compose stack

```powershell
docker compose up -d --build
docker compose ps
docker compose exec frontend ping -c 1 db
```

The final command must fail because `frontend` has no database network. Open http://localhost:8011 and add two registrations. Capture `S2-2.png` for the failed ping and `S2-3.png` for the browser page, including its address bar.

```powershell
docker compose down
docker compose up -d
docker volume ls
```

Reload the page to confirm both registrations remain, then capture `S2-4.png`.

## Stage 3 - scan and harden

The current branch contains the hardened Python 3.13 Alpine Dockerfile and upgraded dependencies.

```powershell
New-Item -ItemType Directory -Force reports, screenshots | Out-Null
trivy image fest-backend:0.1 | Tee-Object reports/trivy-before.txt
docker build -t fest-backend:0.2 .\backend
trivy image fest-backend:0.2 | Tee-Object reports/trivy-after.txt
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
$LASTEXITCODE
docker image ls fest-backend:0.1,fest-backend:0.2
```

Capture the baseline totals as `S3-1.png` and the final Trivy command, its exit code of `0`, and image sizes as `S3-3.png`. Replace the three `Pending` fields in the vulnerability table in `README.md` with the actual CRITICAL/HIGH totals and image sizes.

## Stage 4 - tag and publish

```powershell
docker build --build-arg APP_VERSION=1.0.0 -t fest-backend:1.0.0 .\backend
docker tag fest-backend:1.0.0 ayushsingh5307/fest-backend:1.0.0
docker tag fest-backend:1.0.0 ayushsingh5307/fest-backend:1.0
docker push ayushsingh5307/fest-backend:1.0.0
docker push ayushsingh5307/fest-backend:1.0
$release = docker image inspect ayushsingh5307/fest-backend:1.0.0 --format '{{index .RepoDigests 0}}'
$release
```

Record the full `sha256:` digest in `README.md`, capture the push output as `S4-1.png`, and capture Docker Hub's public Tags page as `S4-2.png`. Then ask Codex to apply the final compose change to use the pushed `:1.0.0` image rather than a build context.

## Stage 5 - sign and verify

```powershell
cosign generate-key-pair
git add cosign.pub
git commit -m 'Stage 5: add public cosign key'
$digest = docker image inspect ayushsingh5307/fest-backend:1.0.0 --format '{{index .RepoDigests 0}}'
cosign sign --key cosign.key $digest
cosign verify --key cosign.pub $digest
docker build --build-arg APP_VERSION=1.0.1 -t ayushsingh5307/fest-backend:1.0.1 .\backend
docker push ayushsingh5307/fest-backend:1.0.1
$unsigned = docker image inspect ayushsingh5307/fest-backend:1.0.1 --format '{{index .RepoDigests 0}}'
cosign verify --key cosign.pub $unsigned
```

Use a private password when `cosign generate-key-pair` asks. Never add `cosign.key` to Git. Capture `S5-1.png` through `S5-4.png`, including the rejected verification for `1.0.1`.

## Final GitHub push

After all reports, screenshots, README values, and the final compose change are present:

```powershell
git add .
git commit -m 'Complete Docker and container security assessment'
git branch -M main
git remote add origin https://github.com/ayushsingh223223/202401100200111-docker-assessment.git
git push -u origin main
```

If `origin` already exists, use `git remote set-url origin https://github.com/ayushsingh223223/202401100200111-docker-assessment.git` instead of `git remote add`.
