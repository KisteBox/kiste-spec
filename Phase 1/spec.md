# Kiste Phase 1 Spec

## Phase 1 Name

**Kiste Phase 1: Repo Scanner + Token Inspector**

Phase 1 does not deploy yet.

It proves that Kiste can:

- Read a Docker-based repo
- Understand the app structure
- Extract deployment requirements
- Inspect tokens
- Check whether the repo is ready for Kubernetes generation

---

## 1. Goal

Kiste scans a repo and tells the developer:

- What the app is
- What services it needs
- Which ports it exposes
- Which environment variables and secrets are required
- Which credentials are available
- Whether the repo is ready for Kubernetes generation

Input:

```text
Docker repo
GitHub token
Local config files
```

Output:

```text
Detected app spec
Detected services
Detected ports
Detected env vars
Detected secrets
Detected Docker setup
Token permission report
Kubernetes readiness report
```

---

## 2. Scope

Phase 1 includes:

- Local CLI
- Repo scanner
- Spec builder
- Token manager
- Token inspector
- Readiness checker
- Report generator

Phase 1 excludes:

- Real Kubernetes deployment
- Terraform apply
- Cloud provisioning
- Karmada integration
- Production secret manager
- Full cost engine
- Full GPU scheduler
- Multi-cloud deployment

---

## 3. Main User Story

As a developer, I want to run:

```bash
kiste check --repo . --target k8s --token github-dev
```

So that Kiste can tell me:

```text
This repo is a FastAPI Docker app.
It uses Postgres and Redis.
It exposes port 8000.
It needs DATABASE_URL, REDIS_URL, and SECRET_KEY.
The GitHub token can read the repo and push images.
The repo is mostly ready for Kubernetes generation.
Ingress host is missing.
```

---

## 4. Initial Project Structure

After running:

```bash
kiste init
```

Kiste creates:

```text
kiste.yaml
.kiste/
  spec.json
  tokens.json.enc
  scan-report.json
```

Example `kiste.yaml`:

```yaml
project:
  name: auto
  environment: dev

repo:
  path: .

target:
  runtime: kubernetes

scan:
  deep: false

security:
  inspect_tokens: true
  detect_secrets: true

output:
  directory: .kiste
```

---

## 5. Repo Scanner

Command:

```bash
kiste scan .
```

The scanner detects:

```text
Language
Framework
Dockerfile
docker-compose.yml
Ports
Services
Env vars
Secrets
Databases
Cache
Queues
GPU libraries
Build command
Start command
```

Example scan output:

```text
Repo scan result:

✅ Dockerfile found
✅ docker-compose.yml found
✅ Python detected
✅ FastAPI detected
✅ Port 8000 detected
✅ Postgres detected
✅ Redis detected
⚠️ SECRET_KEY required
❌ No Kubernetes files found
```

---

## 6. Repo Inspection Commands

### Summary

```bash
kiste repo summary
```

Example:

```text
Project: robot-api
Language: Python
Framework: FastAPI
Runtime: Docker
Main service: api
Port: 8000
Database: Postgres
Cache: Redis
GPU: No
```

### Services

```bash
kiste repo services
```

Example:

```text
SERVICE     TYPE       PUBLIC     PORT
api         web        yes        8000
postgres    database   no         5432
redis       cache      no         6379
```

### Environment Variables

```bash
kiste repo env
```

Example:

```text
ENV VAR        TYPE       SOURCE
DATABASE_URL   secret     docker-compose.yml
REDIS_URL      secret     docker-compose.yml
SECRET_KEY     secret     source code
DEBUG          config     .env.example
```

### Ports

```bash
kiste repo ports
```

Example:

```text
SERVICE     CONTAINER PORT     HOST PORT
api         8000               8000
postgres    5432               private
redis       6379               private
```

### Graph

```bash
kiste repo graph
```

Example:

```text
api -> postgres
api -> redis
frontend -> api
postgres -> private
redis -> private
```

This graph later becomes the base for Kubernetes `Service`, `Ingress`, and `NetworkPolicy`.

---

## 7. Token Manager

Kiste should support:

```bash
kiste token add github --name github-dev
kiste token list
kiste token inspect github-dev
kiste token check github-dev --repo owner/repo
```

Token metadata:

```text
provider
name
expected scopes
expiry date
created date
last checked date
encrypted secret reference
```

Do not store raw tokens in plain text.

Example token metadata:

```yaml
name: github-dev
provider: github
scopes:
  - repo:read
  - packages:write
expires_at: 2026-08-01
created_at: 2026-05-16
last_checked_at: 2026-05-16
status: active
secret_reference: local-encrypted://github-dev
```

---

## 8. Token Inspector

Token inspection should verify:

```text
Token validity
Provider identity
Expiry date if available
Configured expiry metadata
Available scopes if provider exposes them
Required permissions through safe API checks
Repo access
Package/container registry access
```

Example:

```bash
kiste token inspect github-dev
```

Output:

```text
Token: github-dev
Provider: GitHub
Status: active
Expires: 2026-08-01

Checks:
✅ Token is valid
✅ Can read repository
✅ Can read workflow files
✅ Can push package/image
❌ Cannot manage repository secrets
```

---

## 9. Combined Check

Command:

```bash
kiste check --repo . --target k8s --token github-dev
```

This runs:

```text
Repo scan
Spec extraction
Token inspection
Kubernetes readiness check
```

Example output:

```text
Kiste Phase 1 Check

Repo:
✅ Dockerfile found
✅ docker-compose.yml found
✅ FastAPI detected
✅ Port 8000 detected
✅ Postgres detected
✅ Redis detected

Secrets:
⚠️ DATABASE_URL required
⚠️ REDIS_URL required
⚠️ SECRET_KEY required

Token:
✅ GitHub token valid
✅ Can read repo
✅ Can push container image
⚠️ Token expires in 20 days

Kubernetes readiness:
✅ Can generate Deployment
✅ Can generate Service
✅ Can generate ConfigMap
✅ Can generate Secret template
✅ Can generate NetworkPolicy
❌ Ingress host not configured
```

---

## 10. Internal Spec Output

Kiste should write `.kiste/spec.json` after scanning.

Example:

```json
{
  "project": {
    "name": "robot-api",
    "path": "."
  },
  "app": {
    "language": "python",
    "framework": "fastapi",
    "entrypoint": "main.py",
    "port": 8000,
    "start_command": "uvicorn main:app --host 0.0.0.0 --port 8000"
  },
  "docker": {
    "dockerfile": true,
    "compose": true,
    "image_name": "robot-api"
  },
  "services": [
    {
      "name": "api",
      "type": "web",
      "public": true,
      "port": 8000
    },
    {
      "name": "postgres",
      "type": "database",
      "public": false,
      "port": 5432
    },
    {
      "name": "redis",
      "type": "cache",
      "public": false,
      "port": 6379
    }
  ],
  "env": [
    {
      "name": "DATABASE_URL",
      "type": "secret",
      "required": true
    },
    {
      "name": "REDIS_URL",
      "type": "secret",
      "required": true
    },
    {
      "name": "SECRET_KEY",
      "type": "secret",
      "required": true
    }
  ],
  "gpu": {
    "required": false,
    "detected_libraries": []
  },
  "k8s_readiness": {
    "deployment": true,
    "service": true,
    "ingress": false,
    "configmap": true,
    "secret_template": true,
    "network_policy": true
  }
}
```

---

## 11. Architecture

```text
Kiste CLI
   ↓
Repo Scanner
   ↓
Spec Builder
   ↓
Token Manager
   ↓
Readiness Checker
   ↓
Report Generator
```

Recommended implementation:

```text
Language: Python
CLI: Typer
Config parsing: PyYAML
Output: JSON + YAML
Token encryption: keyring or local encrypted file
Repo scan: pathlib + regex + parsers
```

---

## 12. Acceptance Criteria

Phase 1 is complete when the following works:

```bash
kiste init
kiste scan .
kiste repo summary
kiste token add github --name github-dev
kiste token check github-dev --repo owner/repo
kiste check --repo . --target k8s --token github-dev
```

And Kiste can produce:

```text
✅ Repo summary
✅ Service list
✅ Env var list
✅ Port list
✅ App dependency graph
✅ Token validity report
✅ Permission report
✅ Kubernetes readiness report
✅ .kiste/spec.json
```

---

## 13. One-Sentence Spec

**Kiste Phase 1 scans a Docker-based repo, extracts an application deployment spec, inspects tokens, and reports whether the repo is ready to become a Kubernetes deployment package.**
