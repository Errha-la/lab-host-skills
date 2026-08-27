---
name: project-conda-environment
description: Manage one isolated Conda prefix and dependency manifest per project on a shared lab host. Use for environment create, inspect, update, export, or remove; never mutate base/shared Conda.
---

# Project Conda Environment

管理 project-scoped `Environment Lifecycle`；一個 project 對應一個 user-writable prefix。

## Trigger conditions

Conda environment create、inspect、update、export、lock 或 remove。

## Inputs

- `$ARGUMENTS`: `project_path`, `operation`, `env_prefix`, `environment_spec`, `runtime_constraints`。
- Valid [Preflight Manifest](../lab-host-preflight/SKILL.md)。
- Mutation operations 需 explicit authorization。

## Procedure

1. 驗證 account、project、Conda binary、base prefix 與 `Scope Boundary`。
2. 解析 spec/lockfile；缺少 Python/CUDA/framework constraints 時輸出 `SPEC_GAP`，不得自行選版。
3. 決定 `env_prefix`: project-local `.conda/env` 僅限 user-owned writable project；否則要求 user-writable prefix。
4. Mutation 前輸出 `Change Plan`: prefix、channels、packages、write set、rollback path；取得 explicit authorization。
5. 使用 prefix-scoped commands；執行優先採 `conda run --prefix <prefix>`，禁止 base install。
6. 執行 `Runtime Verification`: Python/pip executable、版本、imports、`conda list`、exit code。
7. 產生 `Environment Manifest`: cross-platform export；同平台精確重現另產 explicit lockfile。
8. 將 receipt 送至 [lab-setup-log](../lab-setup-log/SKILL.md)。

## Outputs

`Environment Receipt`: `operation`, `prefix`, `source_spec`, `resolved_packages`, `runtime_verification`, `manifest`, `status`。Status: `PLANNED | PASS | PARTIAL | BLOCKED`。

## Prohibitions

禁止 base/shared/global mutation、system Python/Node mutation、sudo、PATH/profile mutation、global pip/npm、secret export。Remove/cache cleanup 需 exact target 與獨立 authorization。

## Stop conditions

Prefix scope/ownership 不明、runtime constraints 衝突、shared/base 為唯一 writable target、需要 privilege escalation，或 verification 落到 base executable。

## Verification

- Prefix 與 project 一對一，且位於 user-writable scope。
- Python/pip executable 位於該 prefix。
- Manifest 與 resolved packages 可對照且不含 secrets。
- Mutation 有 authorization、exit code 與 receipt。

## Composition

Command boundary 由 [shared-host-safe-execution](../shared-host-safe-execution/SKILL.md) 管理；GPU allocation 由 [gpu-experiment-runner](../gpu-experiment-runner/SKILL.md) 管理。
