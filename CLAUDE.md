# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **prompt-only project**, not a code project. The artifact is `prompt.md` — a comprehensive instruction set that users hand to an LLM (with their own Terraform inputs) to get a Terraform CLI upgrade-feasibility assessment. There is no build, no test suite, no CI, no application code.

## Repository layout

- `prompt.md` — the actual deliverable. Everything else exists to support it.
- `README.md` — user-facing intro and "how to use" walkthrough.
- `LICENSE` — Apache 2.0.
- `.gitignore` — guards against users accidentally committing Terraform artefacts they paste in for local testing (`*.tfstate`, `.terraform/`, `*.tfvars`, etc.).

## Design principles to preserve when editing `prompt.md`

These were deliberate decisions, not defaults — keep them unless the user explicitly asks otherwise:

1. **Keep it general.** The prompt covers any `v1.x → v1.y` upgrade starting from v1.0. Version-specific micro-rules belong in the official HashiCorp upgrade guides the prompt already links to — not in this prompt. The user explicitly rejected a per-version-file structure.
2. **Scope is Terraform 1.0 and later.** Pre-1.0 (0.11/0.12 HCL2 rewrite era) is intentionally out of scope.
3. **Cover both distributions.** HashiCorp Terraform (BSL from v1.6) and OpenTofu (MPL fork from 1.5.x) are both first-class. The license-change step exists specifically to surface this to the user.
4. **Preserve the skeleton.** ROLE → INPUTS (tiered Required/Recommended/Optional) → WORKFLOW (Step 0 — Triage Inputs, then the 16 analysis steps) → REQUIRED OUTPUTS (Risk Matrix, Readiness 0–100, Confidence %, Executive Summary) → MANDATORY RULE (What/When/Impact/Severity/Remediation, split into Verified/Probable/Possible/Unknown) → STYLE. Add steps or rows where genuinely needed; don't reorder for its own sake.
5. **Conservatism is the point.** "Never hide uncertainty," confidence must not exceed evidence, findings categorised by evidence level. Do not soften this language.
6. **Link, don't restate.** When mentioning an upgrade-guide URL, link directly to `developer.hashicorp.com/terraform/language/upgrade-guides` and let the model fetch the version-specific page. Don't inline changelog content here.

## When adding new content

- New Terraform language features (e.g. additions in future minor releases) belong in **Step 6 — HCL Language Feature Audit** as a brief bullet with the version it appeared in. Keep entries one line each.
- New risk dimensions belong as a new row in the Risk Matrix and a corresponding numbered step in the workflow. Update both together.
- Keep the `README.md` "Authoritative references" list in sync with the URL list at the top of `prompt.md`.
