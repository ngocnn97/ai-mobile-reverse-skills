# State Model

This document defines the minimum state model of `analysis_state.json`, which is used to support the step-by-step mode and automatic chain mode of `ai-mobile-reverse-skills`.

## File location

The status file is located by default at:

- `{output_dir}/analysis_state.json`

Among them, `{output_dir}` adopts the unified output root directory convention, for example:

- `analysis_runs/current_run`

## File Responsibilities

`analysis_state.json` does not save specific vulnerability conclusions or protocol analysis results, but is responsible for recording the process status of the current task, including:

- Current target and path context
- Current operating mode
- Current execution phase
-Status of each stage
- Manual readiness status
- Native runtime configuration
- Location of products at each stage

## Pre-configuration before first use

If the process needs to automatically converge so and import Ghidra through `ghidra_target_loader.py`, it is recommended to at least confirm before starting the task:

- `ghidra_root`

Among them, `ghidra_root` is a local path that cannot be stably and automatically guessed. It is recommended that users confirm it manually before using it for the first time. For example:

- `sample_tools/Ghidra/ghidra_x.y.z_PUBLIC`

Subsequent processes will be inherited by default and should not be required repeatedly every time you enter the third stage.

At the same time, the SO automation link also needs to have:

- Decompiled code context: used to locate Java calls, JNI entries and business field relationships
- APK unpacking source code context: the APK unpacking directory must exist and `lib/<abi>/*.so` in it must be accessible

If the APK unpacking source code directory is missing, `selected_native_target.json` can only be used as a candidate basis and cannot enter the "automated pulling so" link. `.so` provided explicitly by the user can be used as native analysis material, but is not automatically pulled.

The rest of the Native runtime parameters, such as:

- `ghidra_project_dir`
- `ghidra_project_name`
- `so_search_roots`
- `preferred_abis`

The default should be automatically derived by the system using a combination of `{output_dir}`, the sample directory structure, the target .so distribution and the default ABI policy.

## Minimal JSON template

```json
{
  "target_name": "demo_app",
  "run_mode": "auto_chain",
  "auto_chain_mode": "B",
  "analysis_mode": "local_source",
  "target_dir": "sample_target/decompiled",
  "output_dir": "analysis_runs/current_run",
  "current_phase": "phase_1",
  "overall_status": "running",
  "manual_ready": {
    "traffic_ready": false,
    "mcp_jadx_ready": false,
    "mcp_burp_ready": false,
    "mcp_native_ready": false
  },
  "native_runtime": {
    "native_mcp": "ghidra-mcp",
    "native_analysis_source": "auto",
    "ghidra_root": "sample_tools/Ghidra/ghidra_x.y.z_PUBLIC",
    "ghidra_project_dir": "",
    "ghidra_project_name": "",
    "so_search_roots": [],
    "preferred_abis": []
  },
  "phases": {
    "phase_1": {
      "status": "pending",
      "step_dir": "analysis_runs/current_run/step1",
      "required_outputs": [
        "file_inventory.json",
        "tech_stack.json",
        "entrypoints.json",
        "env_guard_report.json"
      ],
      "actual_outputs": [],
      "notes": ""
    },
    "phase_2": {
      "status": "pending",
      "step_dir": "analysis_runs/current_run/step2",
      "required_outputs": [
        "api_endpoints.json",
        "protocol_map.json",
        "traffic_alignment.json"
      ],
      "actual_outputs": [],
      "notes": ""
    },
    "phase_3": {
      "status": "pending",
      "step_dir": "analysis_runs/current_run/step3",
      "required_outputs": [
        "crypto_native_analysis.json",
        "jni_analysis.json"
      ],
      "actual_outputs": [],
      "notes": ""
    },
    "phase_4": {
      "status": "pending",
      "step_dir": "analysis_runs/current_run/step4",
      "required_outputs": [
        "vuln_analysis.json",
        "risk_matrix.json"
      ],
      "actual_outputs": [],
      "notes": ""
    },
    "phase_5": {
      "status": "pending",
      "step_dir": "analysis_runs/current_run/step5",
      "required_outputs": [
        "validation_cases.json",
        "test_plan.md",
        "repro_steps.md"
      ],
      "actual_outputs": [],
      "notes": ""
    },
    "phase_6": {
      "status": "pending",
      "step_dir": "analysis_runs/current_run/step6",
      "required_outputs": [
        "security_report.md",
        "findings.json"
      ],
      "actual_outputs": [],
      "notes": ""
    }
  }
}
```

## Stage status field

It is recommended to use the following status values ​​uniformly:

- `pending`
- `running`
- `waiting_review`
- `completed`
- `blocked`
- `failed`

### Meaning

- `pending`: The stage has not started yet
- `running`: The stage is executing
- `waiting_review`: The stage has been executed and is waiting for manual confirmation
- `completed`: The stage has been completed and is available for downstream consumption
- `blocked`: lack of materials, unsatisfied conditions or manual request suspension
- `failed`: Execution error or result unavailable

## Pattern behavior

### `step_by_step`

- After each stage is executed, it is written as `waiting_review` by default
- After manual confirmation, write it as `completed`
- The orchestrator will only enter the next stage after manual confirmation.

### `auto_chain`

- After the stage is completed and the switching conditions are met, the current stage is written as `completed`
- The orchestrator automatically updates `current_phase` and enters the next phase
- If the switching condition is not met, it is written as `blocked`

## Native runtime configuration fields

### `native_runtime.native_mcp`

- `ghidra-mcp`
- `ida-mcp`
- `none`

### `native_runtime.native_analysis_source`

- `auto`
- `selected_target_json`
- Explicit so/IDA/Ghidra project path is only used as user-specified native analysis material and does not represent automated pulling of so

### `native_runtime.ghidra_root`

Ghidra installation root directory.
This is a native path that must be confirmed in advance when automatically importing so into Ghidra. The default should not be guessed by the system.

### `native_runtime.ghidra_project_dir`

Ghidra project directory. If not specified separately, it should be placed by default:

- `analysis_runs/current_run/ghidra_projects`

### `native_runtime.ghidra_project_name`

Ghidra project name. If not specified separately, it should be automatically generated based on the current task name or output directory name by default.

### `native_runtime.so_search_roots`

List of root directories used to automatically find so. If not specified separately, it should be automatically deduced based on the sample unpacking directory first, usually pointing to `lib/` in the unpacking directory.

### `native_runtime.preferred_abis`

Used to determine priority when there are multiple copies of so with the same name. If not specified separately, the default processing is in the following order:

- `arm64-v8a`
- `armeabi-v7a`

## Automatic chain switching conditions

### Link A

Target:

- Phase 1 manual confirmation
- Phase 2-6 automatic advancement

Minimum conditions:

- `phase_1.status = completed`
- `manual_ready.traffic_ready = true`
- `manual_ready.mcp_burp_ready = true` or `{traffic_source}` provided
- `manual_ready.mcp_native_ready = true` or `{native_analysis_source}` provided

### Link B

Target:

- Phase 1-3 manual confirmation
- Phase 4-6 automatic advancement

Minimum conditions:

- `phase_1.status = completed`
- `phase_2.status = completed`
- `phase_3.status = completed`

### Link C

Target:

- Starting from Phase 1 and continuing to automatically advance to Phase 6

Minimum conditions:

- `analysis_mode = local_source` or `manual_ready.mcp_jadx_ready = true`
- `manual_ready.traffic_ready = true`
- `manual_ready.mcp_burp_ready = true` or `{traffic_source}` provided
- `manual_ready.mcp_native_ready = true` or `{native_analysis_source}` provided
- `phase_1.status = completed`

## Update Responsibilities

The responsibilities are divided as follows:

- `agents/*.md`: Responsible for analysis and execution of each stage
- `analysis_state.json`: Responsible for recording process status
- `SKILL.md` Master Control: Responsible for creating and updating status files

During normal use, testers do not need to manually maintain this file. It should be automatically updated by the process when a stage starts, ends, hangs, blocks, or fails.
