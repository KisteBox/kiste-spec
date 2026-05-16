# Kiste Phase 1 Command Reference

Phase 1 focuses on:

- Scanning a Docker-based repository
- Extracting an application spec
- Inspecting tokens
- Checking whether the repo is ready for Kubernetes generation

Phase 1 does not deploy to Kubernetes yet.

---

## 1. Init

```bash
kiste init
```

Creates the local Kiste project structure.

Generated files:

```text
kiste.yaml
.kiste/
  spec.json
  tokens.json.enc
  scan-report.json
```

Example:

```bash
kiste init --name robot-api
```

---

## 2. Repo Scan

Scan the current repo:

```bash
kiste scan .
```

Scan and save the extracted spec:

```bash
kiste scan . --output .kiste/spec.json
```

Scan with JSON output:

```bash
kiste scan . --format json
```

Scan with YAML output:

```bash
kiste scan . --format yaml
```

Deep scan:

```bash
kiste scan . --deep
```

Config-only scan:

```bash
kiste scan . --no-code
```

Expected detection:

```text
Dockerfile
docker-compose.yml
language
framework
ports
services
environment variables
secrets
database dependencies
cache dependencies
GPU libraries
build command
start command
```

---

## 3. Repo Inspection

Show repo summary:

```bash
kiste repo summary
```

Show detected services:

```bash
kiste repo services
```

Show environment variables:

```bash
kiste repo env
```

Show detected ports:

```bash
kiste repo ports
```

Show Docker analysis:

```bash
kiste repo docker
```

Show dependency graph:

```bash
kiste repo graph
```

Example graph:

```text
frontend -> api
api -> postgres
api -> redis
worker -> redis
```

---

## 4. Token Management

Add a GitHub token interactively:

```bash
kiste token add github
```

Add token with name:

```bash
kiste token add github --name github-dev
```

Add token with scope and expiry metadata:

```bash
kiste token add github \
  --name github-dev \
  --scope repo:read,packages:write \
  --expires 2026-08-01
```

List tokens:

```bash
kiste token list
```

List tokens for one provider:

```bash
kiste token list --provider github
```

List expired tokens:

```bash
kiste token list --expired
```

List tokens expiring soon:

```bash
kiste token list --expiring-soon
```

Inspect one token:

```bash
kiste token inspect github-dev
```

Inspect with provider-specific checks:

```bash
kiste token inspect github-dev --provider github
```

Show scopes:

```bash
kiste token inspect github-dev --show-scopes
```

Check expiry only:

```bash
kiste token inspect github-dev --check-expiry
```

---

## 5. Token Checks

Check whether a token is valid:

```bash
kiste token check github-dev
```

Check whether a GitHub token can access a repo:

```bash
kiste token check github-dev --repo khoa/robot-api
```

Check whether a token has a needed permission:

```bash
kiste token check github-dev --need repo:read
```

Check whether token can push package/container image:

```bash
kiste token check github-dev --need packages:write
```

Check whether token supports repo scanning:

```bash
kiste token check github-dev --for scan
```

Check whether token supports build pipeline generation:

```bash
kiste token check github-dev --for build
```

---

## 6. Combined Repo + Token Check

Run full Phase 1 check:

```bash
kiste check
```

Check a specific repo path:

```bash
kiste check --repo .
```

Check repo with token:

```bash
kiste check --repo . --token github-dev
```

Check Kubernetes readiness:

```bash
kiste check --repo . --target k8s
```

Check repo, Kubernetes readiness, and token:

```bash
kiste check --repo . --target k8s --token github-dev
```

Expected output:

```text
Repo check:
✅ Dockerfile found
✅ FastAPI detected
✅ Port 8000 detected
✅ Postgres detected
✅ Redis detected
⚠️ SECRET_KEY required but no value configured

Token check:
✅ GitHub token valid
✅ Can read repo
✅ Can read workflow files
✅ Can push package/image
⚠️ Token expires in 17 days

K8s readiness:
✅ Can generate Deployment
✅ Can generate Service
✅ Can generate ConfigMap
✅ Can generate Secret template
✅ Can generate NetworkPolicy
❌ Ingress host not configured
```

---

## 7. Phase 1 Minimal Command Tree

```text
kiste
  init
  scan
  check
  repo
    summary
    services
    env
    ports
    docker
    graph
  token
    add
    list
    inspect
    check
```

---

## 8. Recommended Phase 1 Workflow

```bash
kiste init
kiste scan .
kiste repo summary
kiste repo services
kiste repo env
kiste token add github --name github-dev
kiste token check github-dev --repo owner/repo
kiste check --repo . --target k8s --token github-dev
```
