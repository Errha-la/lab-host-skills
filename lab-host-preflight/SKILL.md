---
name: lab-host-preflight
description: Run a read-only capability, ownership, Conda, GPU, and scheduler preflight on a shared lab host. Use before host setup, project execution, or GPU work; never mutate host state.
---

# Lab Host Preflight

產生 `Preflight Manifest`；本 skill 僅做 read-only discovery，不取代 MATT、Addy、Isaac 或 domain skill。

## Trigger conditions

Shared lab host、remote execution、Conda／Node base、GPU、scheduler 或 host readiness 檢查。

## Inputs

- `$ARGUMENTS`: `project_path`、`workload_class`、`required_runtimes`、`scheduler_expected`。
- Current shell context；缺省 `project_path` 為 `cwd`，不得推測 account/path。

## Procedure

1. 建立 `Context Fingerprint`: account、host、OS/kernel、shell、cwd、timestamp。
2. 建立 `Scope Boundary`: resolved project path、owner、read/write capability；不掃描其他 user home。
3. 建立 `Runtime Inventory`: Python、Node、package manager、Conda、NVIDIA、scheduler 的 path/version/exit code。
4. 建立 `Conda Topology`: base prefix、active prefix、env directories；base 只標記，不作 project runtime。
5. 建立 `GPU Inventory`: `nvidia-smi -L`、VRAM、utilization、process telemetry；`allocation_provenance` 未證實即為 `UNKNOWN`。
6. 建立 `Scheduler Profile`: command、queue/partition visibility、allocation model；不提交 job。
7. 輸出 manifest 與 gate status。

## Outputs

`Preflight Manifest`: `timestamp`, `context`, `scope`, `runtimes`, `conda`, `gpu`, `scheduler`, `status`, `blockers`。Status: `PASS | PASS_WITH_WARNINGS | BLOCKED`; unknown field 固定為 `UNKNOWN`。

## Prohibitions

禁止 package install、env creation、profile/config mutation、sudo、service restart、process termination、cache/shared-data cleanup、secret collection、hardcoded account/path/GPU index。

## Stop conditions

Context、scope、ownership、GPU allocation 或 scheduler policy 任一不可驗證；或檢查需要 privilege escalation。

## Verification

- Command 均為 read-only 且保存 exit code。
- Base/shared/project resources 分類正確。
- `UNKNOWN` 不升格為 `PASS`。
- Manifest 可供下游 gate 引用。

## Composition

通過後才進入 [project-conda-environment](../project-conda-environment/SKILL.md) 或 [shared-host-safe-execution](../shared-host-safe-execution/SKILL.md)。
