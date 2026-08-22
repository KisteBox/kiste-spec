# Kiste Main Specification Archive

This repository is the main specification source of truth that all Kiste projects must follow. It contains the versioned product and architecture specifications for the Kiste ecosystem. Kiste is a deployment-oriented developer tool that starts with Docker repository inspection and grows into Kubernetes generation, GitOps, library/CLI workflows, self-hosting, modular cloud architecture, static plugins, and expanded diagnostics.

## How to use this repository

- Use this repository as the canonical reference before implementing or changing any Kiste project.
- Start with [Phase 1](Phase%201/spec.md) for the initial repo scanner and token inspector scope.
- Use the phase table below to find the specification that matches the capability or release version you need.
- Treat the PDF files as the canonical specs for later phases unless a Markdown source is present beside them.
- Keep future specifications in phase-numbered folders using descriptive filenames.
- Use the Phase 9.12 schema and examples as the canonical format for new Kiste manifests.

## Specification index

| Phase | Focus | Spec files |
| --- | --- | --- |
| Phase 1 | Repo scanner, token inspector, readiness checks, and command reference. | [`spec.md`](Phase%201/spec.md), [`cmd.md`](Phase%201/cmd.md) |
| Phase 2 | Next-phase Kiste specification. | [`kiste_phase2_spec.pdf`](Phase%202/kiste_phase2_spec.pdf) |
| Phase 3 | Phase 3 Kiste specification. | [`kiste_phase3_spec.pdf`](Phase%203/kiste_phase3_spec.pdf) |
| Phase 4 | GitOps and SOPS specification. | [`kiste_phase4_gitops_sops_spec.pdf`](Phase%204/kiste_phase4_gitops_sops_spec.pdf) |
| Phase 4.5 | Review specification. | [`kiste_phase4_5_review_spec.pdf`](Phase%204.5/kiste_phase4_5_review_spec.pdf) |
| Phase 5 | Library and CLI specification. | [`kiste_phase5_library_cli_spec.pdf`](Phase%205/kiste_phase5_library_cli_spec.pdf) |
| Phase 6 | Repository, serverless, and Docker Hub specification. | [`kiste_phase6_repo_serverless_dockerhub_spec.pdf`](Phase%206/kiste_phase6_repo_serverless_dockerhub_spec.pdf) |
| Phase 7 | Data repository specification. | [`kiste_phase7_data_repo_spec.pdf`](Phase%207/kiste_phase7_data_repo_spec.pdf) |
| Phase 8 | Self-hosting, modular cloud, and Argo CD specification. | [`kiste_phase8_self_hosting_modular_cloud_argocd_spec.pdf`](Phase%208/kiste_phase8_self_hosting_modular_cloud_argocd_spec.pdf) |
| Phase 8.5 | Release stabilization and data/blob storage revision. | [`kiste_phase8_5_release_stabilization_spec.pdf`](Phase%208.5/kiste_phase8_5_release_stabilization_spec.pdf), [`kiste_phase8_5_1_data_blob_storage_revision_spec.pdf`](Phase%208.5/kiste_phase8_5_1_data_blob_storage_revision_spec.pdf) |
| Phase 9 / v0.9.0 | Static plugins and split-ready architecture. | [`kiste_phase9_v0_9_0_static_plugins_split_ready_architecture_spec.pdf`](Phase%209/kiste_phase9_v0_9_0_static_plugins_split_ready_architecture_spec.pdf) |
| Phase 9.1 / v0.9.1 | Standard config and contract updates. | [`kiste_v0_9_1_standard_config_contract_spec.pdf`](Phase%209.1/kiste_v0_9_1_standard_config_contract_spec.pdf), [`kiste_v0_9_1_standard_contract_updated_spec.pdf`](Phase%209.1/kiste_v0_9_1_standard_contract_updated_spec.pdf) |
| Phase 9.2 / v0.9.2 | Expanded doctor diagnostics. | [`kiste_v0_9_2_expanded_doctor_spec.pdf`](Phase%209.2/kiste_v0_9_2_expanded_doctor_spec.pdf) |
| Phase 9.10 / v0.9.10 | Capability-first tool/unit/workspace model and global capability dependency graph. | [`tool capability model`](Phase%209.10/kiste_v0_9_10_tool_capability_kisteunit_workspace_model.md), [`global dependency graph`](Phase%209.10/kiste_v0_9_10_global_capability_dependency_graph.md) |
| Phase 9.11 / v0.9.11 | Capability preference/fit and standard-unit/Go-hook boilerplate. | [`preference and fit model`](Phase%209.11/kiste_v0_9_11_capability_preference_fit_model.md), [`standard unit boilerplate`](Phase%209.11/kiste_v0_9_11_boilerplate_for_standard_units_and_four_repo_split.md) |
| Phase 9.12 / v0.9.12 | Canonical Kiste YAML, intent-aware capability matchmaking, deterministic resolution, lock format, schema, and examples. | [`YAML and capability standard`](Phase%209.12/kiste_v0_9_12_yaml_capability_standard.md), [`JSON Schema`](Phase%209.12/schemas/kiste-v1alpha1.schema.json), [`examples`](Phase%209.12/examples/) |

## Phase progression

1. **Read** GitHub, GitLab, Hugging Face, Docker registries, cloud APIs, Kubernetes APIs, docs, specs
2. **Inspect** Kiste doctor, security scan, performance scan, cost analysis, compliance check, drift detection
3. **Plan** Kubernetes plan, Pulumi plan, Ansible playbook plan, GitOps plan, data/blob plan, rollback plan
4. **Review** Human approval, partial approval, change request, risk acceptance
5. **Deploy** Argo CD, Pulumi, Ansible, CI/CD, Kubernetes, cloud APIs
6. **Monitor** Continuously validate health, tests, scans, compliance, cost, drift, and outcomes.

Back to Inspect when drift/risk/failure appears

## Repository maintenance

When adding or updating specs:

1. Keep this repository aligned as the source of truth for every Kiste implementation repository.
2. Add new files to the appropriate phase folder.
3. Use filenames that include the Kiste phase or release version.
4. Update the specification index in this README.
5. Prefer Markdown for editable source specs and PDF for published artifacts.
6. Validate new canonical manifests against the current machine-readable schema.
