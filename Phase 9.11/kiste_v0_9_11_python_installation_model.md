# Kiste v0.9.11 — Python Installation Model for Separate CLI and Python SDK Repositories

Status: Packaging clarification  
Release: `0.9.11`  
Theme: `kiste-cli` and `kiste-py` are separate source repositories, but normal `pip install` should install both.

---

## 1. Purpose

v0.9.13 will split Kiste into five repositories:

```text
1. kiste-cli
2. kiste-py
3. kiste-unit-sdk
4. kiste-core
5. kiste-spec
```

Even though `kiste-cli` and `kiste-py` live in different repositories, the user installation experience should stay simple.

Final rule:

```text
pip install kiste should install both the Python SDK and the CLI.
```

Repo split is a development and ownership boundary.

Install command is a user experience boundary.

They do not need to be the same.

---

## 2. Recommended PyPI Package Model

Recommended distributions:

```text
kiste
kiste-py
kiste-cli
kiste-core
kiste-unit-sdk
```

Meaning:

```text
kiste:
  umbrella package / default user install
  depends on kiste-py and kiste-cli

kiste-py:
  Python SDK package
  imports and exposes programmatic API

kiste-cli:
  CLI executable package
  depends on kiste-py or kiste-core as needed
  exposes console command: kiste

kiste-core:
  Core engine package
  lifecycle, capability graph, hook runtime, policy, plan/review/monitor models

kiste-unit-sdk:
  optional developer package for building KisteUnits
```

---

## 3. Default Install Behavior

User command:

```bash
pip install kiste
```

Expected result:

```text
Installs:
  kiste
  kiste-py
  kiste-cli
  kiste-core

Provides:
  Python import API
  CLI command: kiste
```

Example:

```python
import kiste

workspace = kiste.Workspace.load(".")
```

CLI:

```bash
kiste read
kiste inspect
kiste plan
kiste review
```

---

## 4. Advanced Install Options

SDK-only install:

```bash
pip install kiste-py
```

CLI-only install:

```bash
pip install kiste-cli
```

Unit developer install:

```bash
pip install kiste-unit-sdk
```

Full developer install:

```bash
pip install "kiste[dev]"
```

Standard units install, if separated later:

```bash
pip install "kiste[standard-units]"
```

---

## 5. Dependency Direction

Recommended dependency direction:

```text
kiste -> kiste-py + kiste-cli
kiste-cli -> kiste-py
kiste-py -> kiste-core
kiste-unit-sdk -> kiste-core
kiste-core -> no runtime dependency on kiste-cli, kiste-py, or kiste-unit-sdk
kiste-spec -> no runtime Python dependency
```

Forbidden dependency direction:

```text
kiste-core -> kiste-cli
kiste-core -> kiste-py
kiste-core -> kiste-unit-sdk
kiste-spec -> runtime packages
```

---

## 6. Console Script Rule

`kiste-cli` owns the console entrypoint.

Example package config:

```toml
[project.scripts]
kiste = "kiste_cli.main:main"
```

The umbrella `kiste` package installs `kiste-cli`, so users still get the command:

```bash
kiste
```

---

## 7. Import Rule

The canonical Python import should remain:

```python
import kiste
```

Even if the source repository is named:

```text
kiste-py
```

Reason:

```text
Repository name and import name do not need to be identical.
```

Recommended:

```text
repository: kiste-py
PyPI distribution: kiste-py
Python import package: kiste
```

The umbrella `kiste` distribution must not break the canonical import.

---

## 8. Release Coordination

Because `pip install kiste` installs both CLI and SDK, releases must coordinate versions.

Recommended version lock:

```text
kiste==0.9.13
kiste-py==0.9.13
kiste-cli==0.9.13
kiste-core==0.9.13
```

The umbrella package should pin compatible ranges:

```toml
[project]
name = "kiste"
dependencies = [
  "kiste-py==0.9.13",
  "kiste-cli==0.9.13"
]
```

For development versions:

```toml
dependencies = [
  "kiste-py>=0.9.13,<0.10.0",
  "kiste-cli>=0.9.13,<0.10.0"
]
```

---

## 9. Why This Matters

The repo split should help maintainability.

It should not make the user install experience harder.

Correct mental model:

```text
Developers see five repositories.
Users see one install command.
```

---

## 10. Acceptance Criteria

This clarification is accepted only if:

```text
1. kiste-cli and kiste-py may live in separate repositories.
2. pip install kiste installs both CLI and Python SDK.
3. kiste-cli owns the console script.
4. kiste-py owns the canonical Python import surface.
5. kiste-core has no runtime dependency on CLI or Python SDK wrappers.
6. kiste-spec has no runtime package dependency.
7. Advanced users may install kiste-py or kiste-cli separately.
8. Version coordination is defined across packages.
```

---

## 11. Final Rule

```text
kiste-cli and kiste-py are separate source repositories.

pip install kiste installs both.

The five-repo split is for development ownership.
The umbrella package is for user experience.
```
