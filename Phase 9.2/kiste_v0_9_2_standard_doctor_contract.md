# Kiste v0.9.2 Standard Doctor Contract

Status: canonical engineering specification  
Repository source of truth: `KisteBox/kiste_spec`  
Target implementation repo: `KisteBox/kiste-py`  
Release: `0.9.2`

## 1. Decision

From v0.9.2 onward, the Kiste spec repository is the source of truth.

Every implementation repository, including `kiste-py`, must follow the contract defined here before adding or changing commands, config sections, output files, or Python APIs.

The v0.9.2 standard is:

```text
One quality command: kiste doctor
Two target scopes: repo / workspace
Four profiles: safe / default / strict / release
Stable groups: config / repo / workspace / lint / security / performance / integration / custom / release
One report model: DoctorReport
One config file: kiste.yaml
One lock file: kiste.lock
One output root: .kiste/
```

Do not add these as separate top-level commands in v0.9.2:

```bash
kiste test
kiste lint
kiste security
kiste performance
kiste doctor --scope self
```

`self` is not a scope. Kiste's own repo is just a repo with Kiste-specific metadata.

---

## 2. Current repo review

### 2.1 Repos reviewed

- `KisteBox/kiste-py`
- `KisteBox/kiste_spec`

### 2.2 Existing upside in `kiste-py`

`kiste-py` already has useful foundations that should be preserved:

1. It already exposes a CLI named `kiste`.
2. It already contains a `kiste_core` package.
3. It already has a `doctor` entry point in `kiste_core.doctor`.
4. It already has config validation in `kiste_core.config`.
5. It already has static plugin registry concepts.
6. It already supports many mature Kiste areas: repo scan, workspace, GitOps, data, blob, Argo CD, cloud, release, and module commands.
7. It already has an implementation path for `kiste config validate`, `kiste config print`, `kiste config schema`, and `kiste config migrate`.

So v0.9.2 should be an alignment release, not a rewrite.

### 2.3 Architecture drift found

#### Drift 1: version is still `0.9.1`

`pyproject.toml` still declares:

```toml
version = "0.9.1"
```

v0.9.2 requires:

```toml
version = "0.9.2"
```

#### Drift 2: config schema is still v0.9.1

`kiste_core.config` still uses:

```python
KISTE_VERSION = "0.9.1"
CONFIG_SCHEMA = "kiste.config/v0.9.1"
```

v0.9.2 requires:

```python
KISTE_VERSION = "0.9.2"
CONFIG_SCHEMA = "kiste.config/v0.9.2"
```

#### Drift 3: `doctor` is too small

Current `run_doctor()` only checks:

```text
kiste.yaml exists
kiste.lock exists
.kiste exists
.kiste/modules.json exists
.kiste/spec.json exists
config validation
static plugin registry
```

v0.9.2 requires `doctor` to become the unified repo/workspace quality gate for:

```text
config
repo
workspace
lint
security
performance
integration
custom
release
```

#### Drift 4: doctor output path is wrong for v0.9.2

Current output path:

```text
.kiste/doctor-report.json
```

v0.9.2 standard output path:

```text
.kiste/doctor/doctor-report.json
```

Optional group reports:

```text
.kiste/doctor/lint-report.json
.kiste/doctor/security-report.json
.kiste/doctor/performance-report.json
.kiste/doctor/integration-report.json
.kiste/doctor/custom-report.json
.kiste/doctor/release-report.json
```

#### Drift 5: doctor report status is old style

Current status values:

```text
OK
WARN
FAIL
```

v0.9.2 standard status values:

```text
pass
warn
fail
skip
info
error
```

Top-level report status:

```text
pass
warn
fail
error
```

#### Drift 6: CLI `doctor` has no standard arguments

Current CLI registers:

```python
doctor = sub.add_parser("doctor", help="Diagnose the Kiste project contract and generated state")
doctor.set_defaults(handler=cmd_doctor)
```

v0.9.2 requires:

```bash
kiste doctor <path>
kiste doctor <path> --scope repo
kiste doctor <path> --scope workspace
kiste doctor <path> --profile safe
kiste doctor <path> --profile default
kiste doctor <path> --profile strict
kiste doctor <path> --profile release
kiste doctor <path> --group lint
kiste doctor <path> --group security
kiste doctor <path> --group performance
kiste doctor <path> --group integration
kiste doctor <path> --group custom
kiste doctor <path> --execute-custom
kiste doctor <path> --json
```

#### Drift 7: `check` still exists as alias

Current CLI has:

```bash
kiste check --repo . --target k8s
```

v0.9.2 decision:

```text
Keep `kiste check` only as a deprecated alias for `kiste doctor`.
Do not document it as the primary command.
```

#### Drift 8: old `self` command group remains conceptually confusing

Current README mentions:

```bash
kiste self init
kiste self scan
kiste self check
kiste self improve
```

v0.9.2 does not remove implementation immediately, but the canonical standard must not build new doctor behavior around `self` scope.

Correct replacement:

```bash
kiste doctor ./kiste --scope repo --profile release --execute-custom
```

#### Drift 9: plugin registry does not include v0.9.2 doctor capability groups

Current registry includes plugins such as:

```text
repo
tokens
workspace
gitops
secrets
images
targets
data
blob
cloud
argocd
self
review
```

v0.9.2 should add or represent doctor capability groups:

```text
doctor
lint
security
performance
integration
custom
release
```

These may be internal doctor groups, not necessarily separate external plugin packages.

#### Drift 10: `kiste_spec` currently has PDF-only v0.9.2 source

The spec repo indexes `Phase 9.2` but currently points to a PDF artifact. v0.9.2 should also have editable Markdown source so implementation repos can track exact contracts.

This file becomes the editable source.

---

## 3. Standard config contract

### 3.1 Required files

Every Kiste repo should have:

```text
kiste.yaml
kiste.lock
```

Every Kiste workspace should have either:

```text
kiste.workspace.yaml
```

or a `kiste.yaml` with:

```yaml
kind: Workspace
```

### 3.2 Standard output root

All generated local state goes under:

```text
.kiste/
```

Doctor output goes under:

```text
.kiste/doctor/
```

Do not write doctor reports directly to `.kiste/doctor-report.json` in v0.9.2.

---

## 4. Standard `kiste.yaml`

This is the canonical v0.9.2 YAML shape for a Kiste implementation repo.

```yaml
kiste: "0.9.2"
schema: "kiste.config/v0.9.2"
kind: Project

project:
  name: kiste
  type: platform
  repository: github
  description: Repo-first platform engineering toolkit
  languages:
    - python
  runtimes:
    - cli
    - library

architecture:
  mode: split-ready
  packages:
    core: packages/kiste-core
    python_sdk: packages/kiste-py
    cli: apps/kiste-cli

modules:
  enabled:
    - repo
    - workspace
    - doctor
    - lint
    - security
    - performance
    - integration
    - data
    - blob
    - gitops
    - argocd
    - cloud

doctor:
  default_scope: repo

  profiles:
    safe:
      description: Read-only checks for unknown or untrusted repos.
      groups:
        - config
        - repo
        - security
        - blob

    default:
      description: Normal local development checks.
      groups:
        - config
        - repo
        - lint
        - security
        - performance

    strict:
      description: CI-quality validation.
      groups:
        - config
        - repo
        - lint
        - security
        - performance
        - integration

    release:
      description: Full release-readiness validation.
      groups:
        - config
        - repo
        - architecture
        - module
        - plugin
        - integration
        - lint
        - security
        - performance
        - custom
        - release

  lint:
    enabled: true
    autofix: false
    adapters:
      python:
        enabled: true
        tool: ruff
        required: false
      node:
        enabled: true
        detect_only: true
      go:
        enabled: true
        detect_only: true

  security:
    enabled: true
    secret_scan: true
    dependency_scan: detect-only
    adapters:
      python:
        enabled: true
        tool: bandit
        required: false
      github_actions:
        enabled: true
      docker:
        enabled: true
      kubernetes:
        enabled: true
        detect_only: true

  performance:
    enabled: true
    mode: readiness
    run_benchmarks: false
    max_large_file_mb: 50

  custom:
    enabled: true
    run_by_default: false
    scripts:
      - name: unit-tests
        command: pytest
        args:
          - tests
        timeout_seconds: 180
        required: true
        working_dir: .

      - name: build-package
        command: python
        args:
          - -m
          - build
        timeout_seconds: 120
        required: true
        working_dir: .

integrations:
  pulumi:
    enabled: true
    mode: detect-only

  ansible:
    enabled: true
    mode: detect-only

  cncf:
    enabled: true
    mode: detect-only
    tools:
      - kubernetes
      - helm
      - argocd
      - prometheus
      - opentelemetry
      - opa

  pytorch:
    enabled: true
    mode: inspect-only
    gpu_required: false

outputs:
  directory: .kiste
  doctor:
    directory: .kiste/doctor
    report: .kiste/doctor/doctor-report.json
```

---

## 5. Standard `kiste.lock`

For v0.9.2 keep the lockfile simple.

```json
{
  "kiste": "0.9.2",
  "schema": "kiste.lock/v0.9.2",
  "plugin_api": "0.2",
  "doctor_contract": "0.9.2"
}
```

Do not turn `kiste.lock` into a Python package dependency lock. That is the job of `uv.lock`, `poetry.lock`, `requirements.lock`, `package-lock.json`, `go.sum`, or similar ecosystem lockfiles.

`kiste.lock` locks Kiste contract state, not all third-party package versions.

---

## 6. Standard CLI contract

### 6.1 Canonical command

```bash
kiste doctor <path>
```

If `<path>` is omitted, it means:

```bash
kiste doctor .
```

### 6.2 Scope

```bash
kiste doctor <path> --scope repo
kiste doctor <path> --scope workspace
```

Auto-detection rule:

```text
If kiste.workspace.yaml exists -> workspace
Else if kiste.yaml exists and kind is Workspace -> workspace
Else -> repo
```

### 6.3 Profiles

```bash
kiste doctor <path> --profile safe
kiste doctor <path> --profile default
kiste doctor <path> --profile strict
kiste doctor <path> --profile release
```

Profile meaning:

```text
safe     = safest read-only checks for unknown repos
default  = normal local development checks
strict   = CI-quality checks
release  = release-readiness checks
```

### 6.4 Groups

```bash
kiste doctor <path> --group lint
kiste doctor <path> --group security
kiste doctor <path> --group performance
kiste doctor <path> --group integration
kiste doctor <path> --group custom
```

Multiple groups are allowed:

```bash
kiste doctor . --group security --group performance
```

### 6.5 Custom script execution

Custom scripts must not execute by accident.

Use:

```bash
kiste doctor . --profile release --execute-custom
```

or:

```bash
kiste doctor . --group custom --execute-custom
```

Do not use `--include-custom`. The correct flag is `--execute-custom` because it makes execution risk explicit.

### 6.6 JSON output

```bash
kiste doctor . --json
kiste doctor . --profile strict --json
kiste doctor . --profile release --execute-custom --json
```

### 6.7 Deprecated alias

`kiste check` may remain temporarily, but only as a deprecated compatibility alias:

```bash
kiste check --repo .
```

Equivalent to:

```bash
kiste doctor .
```

Do not use `kiste check` in new docs.

---

## 7. Standard Python API

### 7.1 Public API

```python
from kiste import doctor

report = doctor.run(".")
```

### 7.2 Strict mode

```python
from kiste import doctor

report = doctor.run(".", profile="strict")
```

### 7.3 Workspace mode

```python
from kiste import doctor

report = doctor.run("./workspace", scope="workspace", profile="strict")
```

### 7.4 Group-specific mode

```python
from kiste import doctor

report = doctor.run(".", groups=["security", "performance"])
```

### 7.5 Release mode with custom scripts

```python
from kiste import doctor

report = doctor.run(".", profile="release", execute_custom=True)
```

### 7.6 Required Python signature

```python
def run(
    path: str = ".",
    *,
    scope: str | None = None,
    profile: str = "default",
    groups: list[str] | None = None,
    execute_custom: bool = False,
) -> DoctorReport:
    ...
```

Do not expose CLI rendering options such as `json_output` in the core Python API. The core returns objects. The CLI renders JSON or text.

---

## 8. Standard core API

Implementation repos should expose this lower-level function from `kiste_core`:

```python
def run_doctor(
    path: str | Path = ".",
    *,
    scope: str | None = None,
    profile: str = "default",
    groups: list[str] | None = None,
    execute_custom: bool = False,
    write: bool = True,
) -> DoctorReport:
    ...
```

`kiste-py` and `kiste-cli` must both call this core engine.

Correct dependency graph:

```text
        kiste-core
        /        \
       v          v
   kiste-py    kiste-cli
```

Incorrect dependency graph:

```text
kiste-cli -> kiste-py -> kiste-core
```

---

## 9. Standard report model

### 9.1 `DoctorFinding`

```python
@dataclass
class DoctorFinding:
    id: str
    group: str
    status: str
    message: str
    severity: str | None = None
    tool: str | None = None
    suggested_command: str | None = None
    path: str | None = None
    duration_seconds: float | None = None
```

### 9.2 `DoctorReport`

```python
@dataclass
class DoctorReport:
    schema: str = "kiste.doctor/v0.9.2"
    status: str = "pass"
    scope: str = "repo"
    path: str = "."
    profile: str = "default"
    groups: list[str] = field(default_factory=list)
    checks: list[DoctorFinding] = field(default_factory=list)
```

### 9.3 JSON output

```json
{
  "schema": "kiste.doctor/v0.9.2",
  "status": "warn",
  "scope": "repo",
  "path": ".",
  "profile": "default",
  "groups": [
    "config",
    "repo",
    "lint",
    "security",
    "performance"
  ],
  "checks": [
    {
      "id": "config.kiste_yaml.exists",
      "group": "config",
      "status": "pass",
      "message": "kiste.yaml found"
    },
    {
      "id": "security.env_file.present",
      "group": "security",
      "status": "warn",
      "message": ".env exists; verify it is not committed"
    },
    {
      "id": "performance.docker.no_healthcheck",
      "group": "performance",
      "status": "warn",
      "message": "Dockerfile has no HEALTHCHECK"
    }
  ],
  "summary": {
    "pass": 1,
    "warn": 2,
    "fail": 0,
    "skip": 0,
    "info": 0
  }
}
```

### 9.4 Status rules

Finding statuses:

```text
pass
warn
fail
skip
info
```

Report statuses:

```text
pass
warn
fail
error
```

Top-level status calculation:

```text
If any check is fail -> fail
Else if any check is warn -> warn
Else pass
Internal exception -> error
```

---

## 10. Standard output layout

```text
.kiste/
  doctor/
    doctor-report.json
    lint-report.json
    security-report.json
    performance-report.json
    integration-report.json
    custom-report.json
    release-report.json

  cache/
  tmp/
  logs/
```

Commit policy:

```text
Commit:
  kiste.yaml
  kiste.lock
  contracts/schemas/
  docs/
  examples/
  tests/fixtures/

Usually do not commit:
  .kiste/cache/
  .kiste/tmp/
  .kiste/logs/
  .kiste/doctor/*.json
```

---

## 11. Standard profiles

```python
PROFILE_GROUPS = {
    "safe": [
        "config",
        "repo",
        "security",
        "blob",
    ],
    "default": [
        "config",
        "repo",
        "lint",
        "security",
        "performance",
    ],
    "strict": [
        "config",
        "repo",
        "lint",
        "security",
        "performance",
        "integration",
    ],
    "release": [
        "config",
        "repo",
        "architecture",
        "module",
        "plugin",
        "integration",
        "lint",
        "security",
        "performance",
        "custom",
        "release",
    ],
}
```

In v0.9.2, `architecture`, `module`, and `plugin` may be partially implemented, but their names are reserved.

---

## 12. Standard check groups

### 12.1 Config

Checks:

```text
kiste.yaml exists
schema is kiste.config/v0.9.2
kiste version is 0.9.2
kind is valid
project.name exists
unknown top-level sections are rejected unless they start with x-
```

### 12.2 Repo

Checks:

```text
repo path exists
Git repo detected when expected
README exists
LICENSE exists when release profile is used
pyproject/package/go files detected depending on runtime
```

### 12.3 Workspace

Checks:

```text
workspace config exists
workspace repo list exists
workspace graph can be built
repo references are valid
```

### 12.4 Lint

Checks:

```text
Python: Ruff detection / optional run
Node: ESLint/Prettier detection
Go: gofmt/go vet/staticcheck detection
YAML/JSON: config parse validation
```

Do not auto-fix by default.

### 12.5 Security

Checks:

```text
plaintext secret patterns
.env risk
raw token risk
GitHub Actions broad permissions
Dockerfile latest tag risk
Dockerfile root user risk
Kubernetes privileged/securityContext risk
SOPS/secret mapping existence
```

Do not claim full security audit.

Use wording like:

```text
No obvious plaintext secret pattern found.
```

Do not say:

```text
Repo is secure.
```

### 12.6 Performance

Checks:

```text
large files
large Docker build context
missing .dockerignore
missing Docker HEALTHCHECK
missing Kubernetes resource limits
PyTorch runtime mismatch
large model/blob handling risk
```

Do not benchmark by default.

### 12.7 Integration

Checks:

```text
Pulumi project detection
Ansible inventory/playbook detection
CNCF toolchain detection
PyTorch runtime detection
```

Do not execute:

```text
pulumi up
ansible-playbook
kubectl apply
argocd sync
large blob download
```

### 12.8 Custom

Custom scripts execute only when:

```text
doctor.custom.enabled = true
and --execute-custom is passed
```

Each script requires:

```text
name
command
args
timeout_seconds
required
working_dir
```

### 12.9 Release

Checks:

```text
version consistency
schema consistency
package build readiness
docs readiness
release notes readiness
DoctorReport generated
```

---

## 13. Standard CLI parser requirements

The `doctor` parser must accept:

```python
doctor.add_argument("path", nargs="?", default=".")
doctor.add_argument("--scope", choices=("repo", "workspace"))
doctor.add_argument("--profile", choices=("safe", "default", "strict", "release"), default="default")
doctor.add_argument("--group", action="append", dest="groups")
doctor.add_argument("--execute-custom", action="store_true")
doctor.add_argument("--json", action="store_true")
```

Handler:

```python
def cmd_doctor(args: argparse.Namespace) -> int:
    report = run_doctor(
        Path(args.path),
        scope=args.scope,
        profile=args.profile,
        groups=args.groups,
        execute_custom=args.execute_custom,
    )
    if args.json:
        print(json.dumps(report.to_dict(), indent=2))
    else:
        print(render_doctor_report(report))
    return doctor_exit_code(report, profile=args.profile)
```

Exit code:

```text
0 = pass or advisory warn
1 = fail, or strict/release warn if configured to fail on warn later
2 = internal error
```

For v0.9.2:

```text
strict/release fail on fail
warnings do not fail unless a later --fail-on-warn flag is added
```

---

## 14. Standard config validation updates

`ALLOWED_TOP_LEVEL` must include:

```python
"doctor",
"integrations",
"outputs",
"architecture",
```

`CONFIG_SCHEMA_JSON` must update to:

```text
$id: https://kiste.dev/schemas/kiste-config-v0.9.2.schema.json
kiste const: 0.9.2
schema const: kiste.config/v0.9.2
```

The config validator must accept doctor profiles and known groups.

Unknown top-level sections remain invalid unless prefixed with:

```text
x-
```

---

## 15. Implementation migration tasks for `kiste-py`

1. Update `pyproject.toml` version from `0.9.1` to `0.9.2`.
2. Update `KISTE_VERSION` to `0.9.2`.
3. Update `CONFIG_SCHEMA` to `kiste.config/v0.9.2`.
4. Add `doctor`, `integrations`, `outputs`, and `architecture` to config schema if missing.
5. Replace old `run_doctor(repo_root, write=True) -> dict` with `run_doctor(...) -> DoctorReport`.
6. Move doctor output from `.kiste/doctor-report.json` to `.kiste/doctor/doctor-report.json`.
7. Add `DoctorFinding` and `DoctorReport` models.
8. Add profile resolution.
9. Add group resolution.
10. Add repo/workspace scope detection.
11. Add lint group.
12. Add security group.
13. Add performance-readiness group.
14. Add integration detect-only group.
15. Add custom script group with `execute_custom=False` by default.
16. Update CLI `kiste doctor` parser with path/profile/scope/group/execute-custom/json.
17. Change `kiste check` into deprecated alias only.
18. Update README to document `kiste doctor`, not `kiste check`, as the quality command.
19. Add tests for profile/group resolution.
20. Add tests for `.kiste/doctor/doctor-report.json` path.

---

## 16. Final standard

The final v0.9.2 implementation target is:

```bash
kiste doctor .
kiste doctor . --profile strict
kiste doctor . --profile release --execute-custom
kiste doctor . --scope workspace --profile strict
kiste doctor . --group security --json
```

Python standard:

```python
from kiste import doctor

report = doctor.run(".")
report = doctor.run(".", profile="strict")
report = doctor.run(".", profile="release", execute_custom=True)
report = doctor.run(".", groups=["security", "performance"])
```

YAML standard:

```yaml
kiste: "0.9.2"
schema: "kiste.config/v0.9.2"
kind: Project

doctor:
  default_scope: repo
  profiles:
    default:
      groups: [config, repo, lint, security, performance]
```

Safety standard:

```text
Doctor is read-only by default.
Custom scripts require --execute-custom.
Security is a readiness scan, not an audit guarantee.
Performance is readiness, not benchmarking by default.
External tools are adapters and must not crash doctor if missing.
```

This contract should be implemented in `kiste-py` and referenced by all future Kiste repos.
