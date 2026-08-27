---
name: lab-setup-log
description: Append auditable manifests and receipts for lab-host preflight, project environments, command execution, and GPU runs. Use for reproducibility and evidence-state tracking; never store secrets or promote partial evidence to success.
---

# Lab Setup Log

提供 append-only `Evidence Ledger`；不執行 setup、environment mutation 或 workload。

## Trigger conditions

Preflight、environment、execution 或 GPU run 需要 reproducibility、audit trail、handoff。

## Inputs

- `$ARGUMENTS`: `project_path`, `log_path`, `event_type`, `manifest_or_receipt`。
- Upstream manifest/receipt；log target 必須位於 verified user-writable scope。

## Procedure

1. 讀取既有 ledger；保留歷史 `BLOCKED/PARTIAL/WARNING`，禁止 silent rewrite。
2. Append `Evidence Event`: timestamp、context fingerprint、project revision/data version、environment prefix、command/entrypoint、resource envelope、exit code、artifacts、evidence state、blockers。
3. `Evidence State`: `PLANNED | ATTEMPTED | VALIDATED | PARTIAL | BLOCKED | EVIDENCE_PENDING`。
4. 執行 `Secret Redaction`: token、password、private key、credential URL、sensitive env values。
5. Conda event 加入 spec/prefix/runtime/manifest checksum；GPU event 加入 GPU choice、placement recommendation、allocation provenance、job/run ID、checkpoint、metrics。
6. Read-back verify：schema、status/evidence consistency、path scope、secret scan。

## Outputs

Append-only Markdown/structured ledger 與 `Log Receipt`: `event_id`, `log_path`, `evidence_state`, `evidence_refs`, `unresolved_items`, `verification`。

## Prohibitions

禁止 secrets、other-user metadata、history deletion/overwrite、shared-document mutation、evidence inflation。Download、file presence、process start、job submission 均非 `VALIDATED`。

## Stop conditions

Log scope/ownership 不明、schema 無法安全解析、secret redaction 未完成、status 與 evidence 衝突，或 context/revision 不可識別。

## Verification

- Event 可依 context、prefix、command、inputs、resource envelope 重現。
- 每個 state 有 evidence 或 `EVIDENCE_PENDING`。
- Ledger 無 secrets 與 unauthorized paths。
- 歷史 append-only；未決項有 confirmation path/owner（若已知）。

## Composition

Evidence flow: `MATT → preflight → environment → execution → domain work → GPU run → ledger`。
