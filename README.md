# Terraform Upgrade Assessment with AI

A single [`prompt.md`](./prompt.md) you can hand to any LLM — Claude, ChatGPT, Gemini, Llama, or any model behind an OpenAI-compatible API — together with details of your Terraform setup, and get back a rigorous, evidence-based assessment of whether your Terraform CLI upgrade is safe to perform.

---

## What this is

Upgrading Terraform is **forward-only** for the state file. Once a newer CLI writes the state, no older CLI can read it. Multiply that by every developer's laptop, every CI runner, every drift detector, every policy engine, and every consumer of `terraform show -json` — and a "small bump from 1.5 to 1.9" stops being small.

This repo gives you one comprehensive prompt that asks an LLM to act as a senior Terraform engineer and walk your specific setup through a 16-step checklist before you change a single binary:

- Version path reconnaissance (every intermediate minor counts)
- Distribution choice — HashiCorp Terraform (BSL from v1.6) vs [OpenTofu](https://opentofu.org/) (MPL)
- `required_version`, provider, and module compatibility
- HCL language feature audit (`moved`, `import`, `check`, `removed`, `optional()`, provider-defined functions, ephemeral values…)
- Deprecated / removed syntax scan
- CLI command and JSON-output schema changes (the silent CI killer)
- Backend, lock file, state format, workspaces, concurrent runners
- Plan / drift simulation and an **explicit** rollback plan

The output is a risk matrix, a 0–100 readiness score, a confidence score that must not exceed the evidence, and an APPROVED / CONDITIONAL / NOT RECOMMENDED decision.

**Scope:** Terraform 1.0 (June 2021) and later, including all 1.x releases. Also applicable to OpenTofu. Pre-1.0 (the 0.11→0.12 HCL2 rewrite era) is intentionally out of scope.

---

## How to use

### 1. Gather your inputs

You will get a much better assessment if you give the model evidence rather than asking it to guess. Collect:

| Input | How to get it |
|---|---|
| Current CLI version | `terraform version` |
| Target CLI version | The release you want to move to |
| `terraform` blocks | `grep -r "terraform {" --include="*.tf"` across root and modules |
| Provider inventory | `terraform providers` and `.terraform.lock.hcl` |
| Module inventory | Every `module "x" { source = … version = … }` block |
| Backend config | The `backend "…" { }` block in your root config |
| State summary | The `terraform_version`, `serial`, `lineage`, resource count, and workspace list inside the state file. The first ~30 lines of `terraform state pull` are usually enough. |
| Lock file | The contents of `.terraform.lock.hcl` (small, safe to share if you redact any private mirror URLs) |
| CI/CD | How Terraform is installed and pinned in your pipeline |
| Workspaces | `terraform workspace list` |
| Surrounding tooling | Terragrunt version, OPA/Sentinel policies, drift detectors, anything that parses plan/state JSON |

### 2. Send it to your LLM

Paste the contents of [`prompt.md`](./prompt.md), then below it paste your inputs as a clearly delimited block, for example:

```text
=== USER INPUTS ===

Current CLI: 1.5.7
Target CLI:  1.9.5
Distribution: HashiCorp Terraform (open to OpenTofu)

required_providers:
  aws    = { source = "hashicorp/aws",    version = "~> 5.40" }
  random = { source = "hashicorp/random", version = "~> 3.5"  }

Backend: s3, with DynamoDB locking, KMS-encrypted

State summary:
  terraform_version: 1.5.7
  serial: 8421
  resources: 612 across 4 workspaces (dev, staging, prod, sandbox)

… etc …
```

### 3. Read the report

The model will produce a risk matrix, readiness score, confidence score, and an executive decision. Review the **Confidence** field first — if it is low, your inputs were thin and the report is correspondingly weaker.

### 4. Act

Use the report as input to your change-management process. The prompt explicitly forbids hiding uncertainty, so anything marked `Unknown` is a homework item, not a green light.

---

## Why a prompt and not a tool

- Every Terraform estate is different. A tool that hard-codes rules ages badly; a prompt re-evaluates against the **current** upgrade guide and **your** specific inputs each time you run it.
- LLMs are good at cross-referencing changelogs, upgrade guides, provider docs, and configuration evidence — exactly the boring, error-prone part of an upgrade review.
- You stay in control of which model, which API, and what data leaves your environment.

---

## Authoritative references

The prompt itself includes these links and instructs the model to consult them, but they are useful to have handy:

- Terraform upgrade guides (one per minor release): <https://developer.hashicorp.com/terraform/language/upgrade-guides>
- Terraform CHANGELOG: <https://github.com/hashicorp/terraform/blob/main/CHANGELOG.md>
- Terraform releases: <https://github.com/hashicorp/terraform/releases>
- BSL license announcement (affects v1.6 and later): <https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license>
- OpenTofu: <https://opentofu.org/> · <https://github.com/opentofu/opentofu>
- Lock file: <https://developer.hashicorp.com/terraform/language/files/dependency-lock>
- Backend configuration: <https://developer.hashicorp.com/terraform/language/backend>
- `required_version`: <https://developer.hashicorp.com/terraform/language/terraform#terraform-required_version>

---

## Contributing

Issues and PRs welcome — especially:

- New language features or deprecations introduced in later Terraform / OpenTofu releases.
- Real-world incident write-ups whose lessons should be encoded in the prompt.
- Wording fixes that make the model's output more concrete and less hand-wavy.

Keep the prompt **general** — version-specific micro-rules belong in the official upgrade guides the prompt already links to. The goal is to teach the model how to reason about any v1.x → v1.y upgrade, not to maintain a parallel changelog.

---

## License

Apache License 2.0 — see [`LICENSE`](./LICENSE).
