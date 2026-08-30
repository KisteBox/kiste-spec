# Kiste v0.9.13B — Trusted Init and Control Unit Runtime

**Status:** Draft implementation architecture  
**Depends on:** Kiste v0.9.13A  
**Theme:** Establish the six canonical Control Units through one trusted `kiste init` transaction and expose a stable workspace/status API.

## 1. Purpose

Phase 9.13A defines the canonical lifecycle:

```text
Read → Inspect → Plan → Review → Deploy → Monitor
```

Phase 9.13B defines the minimum runtime and initialization contract needed to make those Control Units usable and auditable.

The public initialization contract is:

```python
workspace, status = kiste.init()
```

Success:

```python
workspace is not None
status.ready is True
```

Normal validation or security failure:

```python
workspace is None
status.ready is False
```

`status` SHOULD be produced on both successful and failed initialization whenever Kiste can safely construct it.

---

## 2. Control Unit Runtime Model

The six Control Units are system-class KisteUnits with stable identities:

```text
kiste-system/read
kiste-system/inspect
kiste-system/plan
kiste-system/review
kiste-system/deploy
kiste-system/monitor
```

They MAY initially run inside one Kiste process.

Logical separation is required; process or network separation is not.

Each Control Unit MUST support the semantic equivalent of:

```text
bootstrap
run
status
```

`bootstrap` prepares the Control Unit.

`run` performs its lifecycle responsibility.

`status` reports readiness without executing the lifecycle stage.

Future adapters MAY use native, local gRPC, remote gRPC, or WASM execution without changing Control Unit semantics.

---

## 3. `kiste init` Is the Bootstrap

The complete local bootstrap process is exposed to users as:

```bash
kiste init
```

The normal user MUST NOT need to manually execute bootstrap plan/apply/validate steps.

Internally, `kiste init` MUST still behave as a plan-before-apply transaction:

```text
validate environment
      ↓
generate immutable InitPlan
      ↓
calculate plan digest
      ↓
validate built-in init safety policy
      ↓
auto-approve safe local plan
      ↓
apply that exact plan
      ↓
post-validate
      ↓
return workspace + status
```

Apply MUST NOT silently regenerate a different plan.

---

## 4. Auto-Approval

`kiste init` auto-approves by default only because its authority is tightly bounded.

Auto-approval is allowed only when all blocking checks pass and the plan:

- writes only within the canonical workspace boundary;
- performs no external infrastructure mutation;
- executes no workspace code;
- executes no custom hooks;
- downloads no arbitrary Units or plugins;
- reads no secret values;
- requires no network access by default;
- resolves only trusted built-in Control Units;
- preserves existing evidence.

Init auto-approval MUST NOT grant approval or mutation authority to later lifecycle stages.

```text
init auto-approval ≠ Review approval ≠ Deploy authority
```

Deploy MUST begin with no ambient mutation authority and MUST require an approved execution context.

---

## 5. Hostile Workspace Assumption

`kiste init` MUST treat workspace content as untrusted.

Initialization MUST NOT execute:

- repository scripts;
- package install hooks;
- shell commands from workspace files;
- Docker builds;
- Terraform;
- Kubernetes operations;
- cloud mutation APIs;
- custom KisteUnit code.

Workload analysis begins later in Read and Inspect.

---

## 6. OS-Level Initialization Security

Implementations SHOULD use operating-system protections where available.

At minimum, initialization MUST defend against path and state races by requiring:

- canonical workspace path validation;
- path-containment checks for every bootstrap-owned target;
- rejection of unsafe symlink/reparse-point traversal;
- ownership checks where the OS exposes reliable ownership metadata;
- exclusive initialization locking;
- atomic replacement of bootstrap-owned files;
- restrictive permissions for bootstrap state where supported;
- revalidation immediately before apply.

POSIX implementations SHOULD use non-following file opens and advisory/exclusive file locks where available.

Windows implementations SHOULD use equivalent exclusive file locking and reparse-point checks where available.

A suspicious path or ownership conflict MUST fail closed rather than being auto-repaired.

---

## 7. WorkspaceControl

A successful `kiste.init()` returns a usable `WorkspaceControl` object.

It represents the trusted local control-plane handle and SHOULD expose or reference:

```text
workspace identity
canonical root
workspace configuration
configuration digest
lock information
policy references
Control Unit registry
evidence index
resolved six Control Units
```

A returned non-`None` workspace means Kiste has established the minimum trust boundary required to use that control plane.

A failed trust decision MUST return `workspace = None`.

---

## 8. WorkspaceStatus

`WorkspaceStatus` is the structured result of initialization.

It SHOULD contain:

```text
ready
phase
message
checks
init ID
auto-approval state
applied state
InitPlan digest
Control Unit health
evidence reference
log reference
```

Suggested phases:

```text
Ready
Degraded
SafeMode
Failed
```

A status object is evidence about Kiste's own control-plane readiness, not workload readiness.

---

## 9. Status and Failure Contract

Normal validation and security problems SHOULD be returned as data rather than process termination.

Example:

```python
workspace, status = kiste.init()

if workspace is None:
    print(status.message)
    for check in status.checks:
        print(check.status, check.message)
```

Library code SHOULD NOT use `SystemExit` for normal init validation failures.

Exceptions SHOULD be reserved for unexpected internal failures where a reliable `WorkspaceStatus` cannot be returned.

---

## 10. Init Checks

`kiste init` MUST check everything necessary to trust Kiste itself, including:

```text
Kiste bootstrap seed/runtime availability
workspace path containment
filesystem type and accessibility
ownership where supported
permissions where supported
symlink/reparse-point safety
existing Kiste state integrity
configuration validity
lock consistency
Control Unit registry integrity
evidence-store availability
InitPlan safety
post-initialization consistency
```

These checks MUST NOT be confused with workload inspection.

```text
kiste init    → trusts Kiste control plane
kiste read    → establishes static workload evidence
kiste inspect → establishes dynamic workload evidence
```

---

## 11. Control Unit Registry

Initialization MUST register exactly one active implementation for each canonical Control Unit.

Suggested generated location:

```text
.kiste/control/registry.json
```

Initial implementation MAY map directly to the existing embedded lifecycle functions.

Example conceptual registry:

```yaml
read:
  unitRef: kiste-system/read
  implementation: embedded

inspect:
  unitRef: kiste-system/inspect
  implementation: embedded

plan:
  unitRef: kiste-system/plan
  implementation: embedded

review:
  unitRef: kiste-system/review
  implementation: embedded

deploy:
  unitRef: kiste-system/deploy
  implementation: embedded
  mutationAuthority: approved-plan-only

monitor:
  unitRef: kiste-system/monitor
  implementation: embedded
```

Initialization registers these Control Units but MUST NOT execute them.

---

## 12. Initialization Evidence and Audit Trail

Every `kiste init` invocation SHOULD produce auditable structured output, including successful runs.

Suggested generated files:

```text
.kiste/init/init-result.json
.kiste/init/init-evidence.json
.kiste/evidence/index.json
```

Initialization evidence SHOULD include:

```text
init ID
workspace identity
Kiste version
InitPlan digest
auto-approval result
checks
Control Unit registry digest
security validation result
evidence-store status
final workspace readiness
```

The audit rule is:

> successful initialization is evidence, not silence.

Existing lifecycle evidence MUST never be erased by re-running `kiste init`.

---

## 13. CLI Contract

Default:

```bash
kiste init
```

The CLI MUST display a concise summary and final check/log location even on success.

Structured output SHOULD be available through:

```bash
kiste init --json
```

The CLI and Python API MUST use the same underlying initialization implementation and status schema.

---

## 14. Idempotence

Re-running:

```bash
kiste init
```

MUST be safe.

It MUST NOT:

- duplicate Control Units;
- erase evidence;
- silently replace suspicious state;
- grant additional authority;
- reset approvals;
- overwrite conflicting trusted metadata without detection.

Security-sensitive conflicts MUST produce `workspace = None` and a failed status.

---

## 15. Migration Requirements for `kiste-py`

Implementation SHOULD consolidate the current init/bootstrap/workspace paths into one core initialization path.

Required migration direction:

```text
1. Add WorkspaceControl and WorkspaceStatus.
2. Add public kiste.init() -> (WorkspaceControl | None, WorkspaceStatus).
3. Make CLI `kiste init` call that same API.
4. Register six embedded Control Units during init.
5. Add InitPlan digest and built-in auto-approval policy.
6. Add OS-level path/locking/atomic-write hardening.
7. Add init result/evidence logs.
8. Remove silent config-load fallback for trusted workspace loading.
9. Remove library-level SystemExit from normal init failure paths.
10. Keep legacy bootstrap commands only as compatibility/diagnostic surfaces.
```

---

## 16. Test Requirements

`kiste_core` MUST include tests covering at least:

```text
successful init returns workspace + ready status
successful init writes result/evidence logs
successful init registers exactly six Control Units
successful init leaves Deploy without ambient authority
re-running init is idempotent
existing evidence is preserved
invalid existing config returns None + failed status
unsafe .kiste symlink/reparse-point returns None + failed status
unsafe kiste.yaml symlink returns None + failed status
Control Unit registry tampering returns None + failed status
invalid evidence index returns None + failed status
InitPlan never requests network/workspace execution/secret values/external mutation
CLI and Python API expose the same status semantics
```

OS-specific security tests SHOULD be skipped only when the host OS cannot provide the required primitive.

---

## 17. Non-Goals

Phase 9.13B does not require:

- six microservices;
- external scanner integrations;
- KubeVela deployment integration;
- SkyPilot integration;
- automatic infrastructure mutation;
- remote evidence storage;
- TOSCA;
- Puccini.

---

## 18. Acceptance Criteria

Phase 9.13B is accepted when:

1. `workspace, status = kiste.init()` is the canonical public init API.
2. Success returns a non-`None` workspace and `status.ready == True`.
3. Normal validation/security failure returns `None` and a readable failed status.
4. Status includes checks/log references on success and failure when possible.
5. CLI and Python use the same init implementation.
6. The six canonical Control Units are registered during init but not run.
7. Exactly one implementation resolves for each canonical stage.
8. Deploy has no ambient mutation authority.
9. Init auto-approval is limited to safe local initialization.
10. Init performs no external mutation or workspace code execution.
11. Init applies the exact immutable plan it validated.
12. Init uses OS-level locking/path protections where available.
13. Init is idempotent and preserves evidence.
14. Suspicious state fails closed instead of being silently repaired.
15. `kiste_core` tests enforce the init success, failure, audit, and filesystem-security contracts.

---

## 19. Final Rule

```text
kiste init
    establishes whether Kiste itself can be trusted.

If trusted:
    return WorkspaceControl + successful WorkspaceStatus.

If not trusted:
    return None + explanatory WorkspaceStatus.

Only after that may Read, Inspect, Plan, Review, Deploy, and Monitor operate.
```
