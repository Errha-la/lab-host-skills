---
name: gpu-experiment-runner
description: Schedule and run bounded GPU experiments from the scheduler-managed shared GPU pool with daily GPU selection, one-line placement advice, allocation provenance, checkpoints, telemetry, and receipts. Use for training, benchmarking, or evaluation on a shared lab host.
---

# GPU Experiment Runner

提供 `GPU Placement Gate` 與 `Run Control Plane`；僅允許 scheduler-managed shared GPU pool；不管理 driver、MIG 或 scheduler configuration。

## Trigger conditions

GPU training、benchmark、evaluation、inference batch 或長時間 CUDA workload。

## Inputs

- `$ARGUMENTS`: `entrypoint`, `args`, `env_prefix`, `output_dir`, `checkpoint_dir`, `resource_envelope`。
- Valid [Preflight Manifest](../lab-host-preflight/SKILL.md)、[Environment Receipt](../project-conda-environment/SKILL.md)、[Execution Gate](../shared-host-safe-execution/SKILL.md)。

## Procedure

1. Refresh `GPU Inventory`: shared-pool candidates、device/class、free VRAM、utilization、active allocation、queue/partition state。
2. 執行 `Shared Pool Gate`：排除 `exclusive`、`private`、`reserved`、ownership unknown 與 direct-allocation device。
3. 執行 `Daily GPU Selection Gate`：向使用者詢問「今天 GPU 要用哪張？」並列出剩餘的 scheduler-visible shared-pool candidates。
4. 同一訊息提供恰好一句 `Placement Recommendation`，依 workload VRAM、GPU utilization、queue latency 與 allocation provenance 排序；資料不足時建議暫停。
5. 等待使用者選擇；selection 僅對當日當次 run 有效，跨日或 inventory 變更須重問。
6. 將選擇轉成 scheduler GPU class/count/resource request，由 scheduler 指派 physical index；禁止 direct allocation 與硬綁 device index。
7. 建立 `Resource Envelope`: walltime、CPU、RAM、GPU count/class、VRAM ceiling、checkpoint cadence、retry budget、telemetry interval。
8. 建立 unique `Run ID` 與 `Run Manifest`; output/checkpoint 必須 user-writable 且 collision-free。
9. 執行 `Smoke Test`: environment、framework import、assigned device、minimal forward/dry batch。
10. 通過後 launch；監測 own job/PID、GPU telemetry、metrics、checkpoint。Boundary breach 時停止 own job 並保存 recoverable state。
11. 驗證 artifacts、checkpoint、metrics、exit code、resource compliance；輸出 receipt 至 [lab-setup-log](../lab-setup-log/SKILL.md)。

## Outputs

`Run Receipt`: `gpu_choice`, `placement_recommendation`, `allocation_provenance`, `job_id/run_id`, `resource_envelope`, `telemetry`, `checkpoints`, `artifacts`, `exit_code`, `status`。Status: `PLANNED | QUEUED | PASS | PARTIAL | BLOCKED`。

## Prohibitions

禁止 hardcoded GPU index/account/partition/path、direct allocation、exclusive/private/reserved device、unallocated device、boundary overrun、other-user job control、shared-output cleanup、driver/MIG/scheduler mutation、sudo、infinite retry、uncheckpointed long run。

## Stop conditions

shared-pool policy、scheduler allocation、quota、VRAM、resource envelope 或 checkpoint strategy 為 `UNKNOWN`；使用者尚未完成每日選卡；無 scheduler；Smoke Test 失敗；或偵測 boundary breach。

## Verification

- GPU choice 與一句 placement recommendation 已記錄。
- Physical device 來源可追溯至 scheduler allocation 且 pool label 為 shared。
- Runtime 使用 project prefix，Smoke Test 與 full run dependency graph 一致。
- Checkpoint、metrics、logs、exit code 與 resource compliance 可驗證。

## Composition

Domain skill（含 Isaac）提供 workload logic；本 skill 擁有 placement、allocation、run boundaries 與 recovery gates。
