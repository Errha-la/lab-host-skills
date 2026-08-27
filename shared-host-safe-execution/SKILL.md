---
name: shared-host-safe-execution
description: Gate bounded commands on a multi-user AI/GPU host through scope, ownership, authorization, and resource controls. Use before any non-read-only host command; never authorize system or other-user mutation.
---

# Shared Host Safe Execution

提供 `Execution Gate`；只允許 read-only 或 user-owned project mutation。

## Trigger conditions

Shared host 上會寫檔、啟動 process、下載 dependency 或消耗 compute 的 command/script。

## Inputs

- `$ARGUMENTS`: `command`, `cwd`, `inputs`, `outputs`, `timeout`, `resource_envelope`。
- Valid [Preflight Manifest](../lab-host-preflight/SKILL.md)。
- Project mutation 另需 valid [Environment Receipt](../project-conda-environment/SKILL.md)。

## Procedure

1. `Operation Class`: `READ_ONLY | PROJECT_MUTATION | DENY`。
2. `Scope Resolution`: resolve cwd/input/output/temporary/checkpoint paths；拒絕 unresolved vars、broad globs、root/home-root/shared targets。
3. `Ownership Gate`: target、process、environment 均須屬 current account。
4. `Execution Plan`: exact command、cwd、I/O set、timeout、resource envelope、checkpoint、failure action。
5. Project mutation 取得 explicit authorization；以 least privilege 執行，禁止 secret interpolation。
6. 產生 `Execution Receipt`; cleanup 僅限本次建立且 user-owned 的 temporary artifacts，非預設步驟。

## Safe defaults

`plan-only`, finite timeout, finite retry, foreground execution, scheduler preference, no cleanup。

## Outputs

`Execution Receipt`: `operation_class`, `authorization`, `command`, `environment`, `scope`, `resource_envelope`, `start/end`, `exit_code`, `artifacts`, `status`。

## Prohibitions

`DENY`: sudo、OS/driver/service/scheduler mutation、other-user process/file access、shared cleanup、global package install、unbounded retry、privilege fallback。

## Stop conditions

Identity、ownership、scope、resource envelope、authorization 或 failure action 任一為 `UNKNOWN`；或 operation class 為 `DENY`。

## Verification

- Plan 與 executed command 一致。
- Write set 全在 `Scope Boundary`。
- Timeout、exit code、artifacts、errors 均有 receipt。
- Process start 不等於 task success。

## Composition

長時間 GPU workload 交給 [gpu-experiment-runner](../gpu-experiment-runner/SKILL.md)；receipt 交給 [lab-setup-log](../lab-setup-log/SKILL.md)。
