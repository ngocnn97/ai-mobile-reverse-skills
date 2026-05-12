#AI Mobile Reverse Skills User Quick Instructions
This is a 6-stage workflow for mobile security analysis, which is used to organize APK static reconnaissance, traffic and code alignment, SO/JNI in-depth analysis, vulnerability closure, verification design and report delivery into a process that can be executed step by step or automatically.

## Remember two things first
1. All modes start from the first step and Phase 1 will not be skipped.
2. There are only two types of running modes: `step_by_step` and `auto_chain`.

## What should you at least prepare before starting:
- Decompile directory, or an open `jadx-mcp` session
- a unified `output_dir`
- If you want to do traffic alignment:
- Packet capture results have been prepared, or `burp-mcp` / `yakit-mcp` has been connected
- If you want to automatically import Ghidra:
- Confirmed in advance `ghidra_root`
- Already have the APK unpacking source code directory, and `lib/<abi>/*.so` exists in the directory

## Step 1: Select the mode first
Stage-by-stage stepping mode:
```text
run_mode: step_by_step
```
It is suitable for scenarios where you want to manually review every step and want to stop and adjust the focus of analysis at any time.

Automatic chain mode:
```text
run_mode: auto_chain
auto_chain_mode: A/B/C
```
meaning:
- `A`: Manually prepare packet capture and MCP after the first stage, and automatically advance to the sixth stage from the second stage.
- `B`: Manual confirmation in the first 1-3 stages, automatically advanced to the sixth stage from the fourth stage onwards
- `C`: Try to advance the entire process automatically from the first stage, suitable for scenarios where pre-preparation has been completed

## Step 2: Enter the first stage again
No matter which mode, the first step template will be sent first:
```text
step: 1
analysis_mode: local_source/jadx_mcp_session
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
jadx_mcp: yes/no
```
Parameter description:
- `analysis_mode`
- `local_source`: analyze the local decompilation directory
- `jadx_mcp_session`: Directly consume the `jadx-mcp` current session
- `target_dir`: main analysis directory after decompilation
- `output_dir`: unified output directory, inherited by default in subsequent stages
- `jadx_mcp`: Whether `jadx-mcp` is currently connected

## Typical startup sequence
The most common startup method is two-stage:

1. First-mover mode:
```text
run_mode: step_by_step
```
or:
```text
run_mode: auto_chain
auto_chain_mode: A/B/C
```

2. Send the first step template again:
```text
step: 1
analysis_mode: local_source/jadx_mcp_session
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
jadx_mcp: yes/no
```

This way the system will know:
- How do you want to run:
- Where to read material
- Where to write the result:

## What do these 6 stages do:
1. `Phase 1`: APK static reconnaissance, technology stack identification, environment detection, suspicious so preliminary screening
2. `Phase 2`: Align traffic with code, converge on key fields and goals so
3. `Phase 3`: In-depth analysis of SO / JNI, with APK unpacking source code, it can automatically pull so and import it into Ghidra
4. `Phase 4`: Comprehensive settlement of encrypted links and high-risk issues
5. `Phase 5`: Minimal verification ideas and POC template design
6. `Phase 6`: Aggregate results and generate standardized reports

## Where to write the result
By default it reads:
```text
{output_dir}/
├── step1/
├── step2/
├── step3/
├── step4/
├── step5/
└── step6/
```
There will also be:
```text
{output_dir}/analysis_state.json
```
It is not an analysis result file, but a process status file, used to record:
- Current operating mode
- What steps are you currently on:
- Which step has been completed
- Which step awaits confirmation
- which step is blocked

If there is an interruption later, it is usually necessary to look at it first and then decide which step to continue from.

## When will the script be automatically used:
Phase 1 if:
```text
analysis_mode: local_source
```
These 4 scripts are executed by default:
- `endpoint_extractor.py`
- `secret_scanner.py`
- `native_bridge_indexer.py`
- `env_guard_indexer.py`

Phase 1 if:
```text
analysis_mode: jadx_mcp_session
```
These four scripts will not be executed by default, and the `jadx-mcp` context will be used directly first.

Phase 2 / 3 will also automatically use:
- `resolve_native_target.py`
- so used to converge the third stage priority analysis after the second stage
- `ghidra_target_loader.py`
- The third stage is used to automatically import the target .so to Ghidra

in other words:
- The first 4 scripts mainly serve local code analysis
- The last two scripts mainly serve Native automatic advancement

## If you want to automatically import Ghidra
SO automation relies on two types of materials:
- Decompiled code context, used to determine why a certain so should be analyzed
- APK unpacking source code directory, used to automatically pull the target .so from `lib/<abi>/*.so`

If there is only a decompiled directory and no APK unpacked source code directory, the system can converge on candidate so names, but cannot automatically pull so. `.so` explicitly provided by the user can be used as native analysis material, but does not belong to "automated pulling so".

Before first use, you only need to confirm one parameter in advance:
```text
ghidra_root
```
It is the root directory of your local Ghidra installation, for example:
```text
sample_tools/Ghidra/ghidra_x.y.z_PUBLIC
```
Currently, Ghidra's automatic import is only adapted to macOS. The automatic pull-up, project opening and GUI behavior under Windows/Linux have not yet been adapted and verified.

It is recommended to write it in:
```text
{output_dir}/analysis_state.json
```
The rest of the Ghidra project directory, project name, so search directory and ABI priority are automatically derived by the system by default.

## When will the automatic chain stop:
Even if you select `auto_chain`, the system will not forcefully run down when key conditions are missing. Common pause points include:
- The packet capture material does not exist, and `burp-mcp` / `yakit-mcp` is not available
- The third stage requires Ghidra to be automatically imported, but `ghidra_root` is not configured
- The key results in the previous stages are not generated, for example, the target .so convergence result is missing.

At this time, the status is usually written, prompting what is missing, and then stopping at the earliest blocking stage.

## The 3 most common usages
Usage 1: Step by step
```text
run_mode: step_by_step
```
Then send the first step template.

Usage 2: Manual analysis at the front, automatic closing at the back
```text
run_mode: auto_chain
auto_chain_mode: B
```
Then send the first step template. The first 1-3 steps are manually confirmed and continue automatically after the fourth step.

Usage 3: All preparations are done, try to run automatically
```text
run_mode: auto_chain
auto_chain_mode: C
```
Then send the first step template. The system will try to advance continuously from the first step.

## If you are stuck, where should you look first:
Prioritize:
- `{output_dir}/analysis_state.json`
- JSON/MD file in the current stage directory

If you want the complete rules, look again:
- `README.md`
- `SKILL.md`
- `docs/STATE-MODEL.md`
