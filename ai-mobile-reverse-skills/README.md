# AI Mobile Reverse Skills

AI Mobile Reverse Skills is a set of staged analysis product specifications for mobile security scenarios. It is used to organize APK static reconnaissance, packet capture linkage, SO/JNI deep mining, encryption and vulnerability comprehensive analysis, verification design, and report delivery into a continuously advancing execution chain.

If you just want to get started quickly, it is recommended to read:

- [USER-README.md](USER-README.md)

Input materials it targets include:

- Android APK
- Directory after unpacking and decompilation
- Jadx decompilation results
- Burp / Yakit packet capture results
- so/JNI related materials
- IDA/Ghidra Engineering
- Structured JSON and reports generated during the pre-processing phase

Its design focus is not to provide fragmented prompts, but to provide:

- Orchestrator entry specification that can be reused by upper-layer AI/Skill
- Fixed 6-stage execution protocol
- Phased MCP usage specifications
- Native indexing scripts and Native automation auxiliary scripts
- Structured deliverables ready for precipitation

## Product capabilities

It helps you do the following 6 things in order:

1. [agent-01-sample-recon.md](agents/agent-01-sample-recon.md)
2. [agent-02-protocol-mapper.md](agents/agent-02-protocol-mapper.md)
3. [agent-03-crypto-native-analyzer.md](agents/agent-03-crypto-native-analyzer.md)
4. [agent-04-crypto-vuln-analyzer.md](agents/agent-04-crypto-vuln-analyzer.md)
5. [agent-05-validation-designer.md](agents/agent-05-validation-designer.md)
6. [agent-06-reporter.md](agents/agent-06-reporter.md)

The overall structure is as follows:

- Root [SKILL.md](SKILL.md) is responsible for overall control and routing
- `agents/agent-01` to `agent-06` are responsible for stage execution
- [MCP-INTEGRATION.md](docs/MCP-INTEGRATION.md) Responsible for phase access specifications
- `tools/scripts/*.py` is responsible for structured index enhancement and Native automation assistance

## Run mode

This set of Skills supports two broad categories of running modes and allows the mode to be selected in advance at the beginning, rather than left to the system to guess on the spot. No matter which mode is selected, the process starts from the first stage, the only difference is which subsequent stage automatically takes over.

### 1. Stage by stage step-by-step mode

Suitable for scenarios where you want to manually control the rhythm.

How to choose:

```text
run_mode: step_by_step
```

Features:

- Testers issue instructions step by step
- Suspend by default after each stage is completed
- Manually review the JSON/Markdown product at the current stage before deciding whether to continue

Applicable methods:

- `Start the first step`
- `Start the second step`
- `Start step three`

### 2. Automatic chain

How to choose:

```text
run_mode: auto_chain
auto_chain_mode: A/B/C
```

The automatic chain is subdivided into 3 links.

#### Link A: Analysis Closed Loop

Suitable for the following scenarios:

- Execution starts from the first stage
- After the first phase is completed, preparatory work such as detection bypass, proxy configuration, packet capture preparation, etc. need to be completed manually.
- `burp-mcp` / `yakit-mcp` and `ghidra-mcp` / `ida-mcp` are connected

Operating range:

- Phase 1 manual confirmation
- Phase 2 -> Phase 3 -> Phase 4 -> Phase 5 -> Phase 6 automatic advancement

illustrate:

- The first phase is still executed normally
- After the first phase, wait for manual completion of packet capture and other pre-preparations
- Even if humans have made pre-preparations, the system will still do pre-checks first instead of directly skipping detection conclusions, packet capture materials and Native clue judgments
- If the key conditions are met, it will automatically advance from the second stage to the sixth stage.

#### Link B: Verification closed loop

Suitable for the following scenarios:

- Execution starts from the first stage
- The first 1-3 stages are gradually confirmed manually
- It has been manually confirmed that it needs to automatically enter the vulnerability verification and report delivery stage after the fourth stage.

Operating range:

- Phase 1 -> Phase 2 -> Phase 3 manual confirmation
- Phase 4 -> Phase 5 -> Phase 6 automatic advancement

illustrate:

- The first three stages are still executed normally and allow manual step-by-step confirmation
- Starting from the fourth stage, the system will automatically connect to the fifth and sixth stages
- Suitable for the scenario of "manual analysis first and automatic closing later"

#### Link C: Through train

Suitable for the following scenarios:

- All manual preparations have been completed from the beginning
- `jadx-mcp`, `burp-mcp` / `yakit-mcp`, `ghidra-mcp` / `ida-mcp` all connected
- The packet capture results are ready

Operating range:

- Phase 1 -> Phase 2 -> Phase 3 -> Phase 4 -> Phase 5 -> Phase 6

illustrate:

- The through train mode does not skip the first stage, but after the first stage is completed, it continues to automatically advance to the subsequent stages.
- Even if the user has manually confirmed that "the current App can capture packets and be debugged", the system will still perform the first stage of sample reconnaissance and detection analysis before deciding whether to automatically enter the subsequent link.

### Mode selection principles

- If you want to manually confirm the results stage by stage, select explicitly:
  - `run_mode: step_by_step`
- If you want the system to continue automatically when the conditions are met, select this explicitly:
  - `run_mode: auto_chain`
- If you want to automate after the first stage and from the second stage, you usually choose:
  - `auto_chain_mode: A`
- If you want manual confirmation in the first 1-3 stages and automation from the fourth stage onwards, usually choose:
  - `auto_chain_mode: B`
- If the entire process is automatically advanced from the first stage, usually choose:
  - `auto_chain_mode: C`

## Standard startup template

In order to avoid organizing input on the spot every time, it is recommended to start by "two-stage interaction".

### Step 1: Declare the pattern first

#### Template 1: Stage-by-stage stepping mode

```text
run_mode: step_by_step
```

#### Template 2: Automatic chain A (analytical closed loop)

```text
run_mode: auto_chain
auto_chain_mode: A
```

#### Template 3: Automatic chain B (verification closed loop)

```text
run_mode: auto_chain
auto_chain_mode: B
```

#### Template 4: Automatic chain C (through train)

```text
run_mode: auto_chain
auto_chain_mode: C
```

### Step 2: Enter the first stage again

No matter which mode you choose, you will enter the first stage template first:

```text
step: 1
analysis_mode: local_source/jadx_mcp_session
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
jadx_mcp: yes/no
```

Additional instructions:

- `step_by_step`: Starting from the first step, suspending by default after the end of each stage
- `auto_chain + A`: Start from the first step, complete the pre-preparation manually after the first stage, and automate it from the second stage
- `auto_chain + B`: Starting from the first step, the first 1-3 stages are manually confirmed, and the fourth stage is automated.
- `auto_chain + C`: Starting from the first step, continue to advance according to the high automation link, but the first phase must still be executed first

## Design principles

### 1. Strictly only go through 6 stages

This skill set will not add new main stages without authorization, nor will it change the order of your stages.

### 2. AI-led analysis

In most cases, priority goes directly to:

- MCP
- Stage documentation
- Current context material

Let AI do the analysis directly.

### 3. Stage script default rules

The current version adopts the approach of "AI-led analysis + stage script combination":

- When the first stage uses `local_source` to analyze the local code directory, the following 4 index scripts are executed by default:
  - `endpoint_extractor.py`
  - `secret_scanner.py`
  - `native_bridge_indexer.py`
  - `env_guard_indexer.py`
- When using `jadx_mcp_session` in the first stage, the above four scripts are not executed by default, and priority is given to consuming the `jadx-mcp` context directly.
- `resolve_native_target.py` and `ghidra_target_loader.py` continue to maintain the original automation logic and are not affected by the first stage input method

In other words, the default execution condition of the first four scripts is "take the local code analysis path", not "unconditional execution of all scenarios".

### 4. Phase 1 is not responsible for unpacking

Phase 1 only analyzes the unpacked and decompiled materials that have been obtained.
This set of Skills does not consider "unshelling" as its responsibility.

## Current form

The current repository shape is:

- Skill orchestrator specification
- 6 stages Agent documentation
- Native indexing scripts and Native automation auxiliary scripts

It is not currently a packaged standalone CLI, TUI or web console.

The mention in the document "You said that the first step is to start, the orchestrator returns the template and routes", which means that the upper layer AI / Skill is executed according to the rules of [SKILL.md](SKILL.md), which does not mean that the automatic dialogue routing program is already included in the repository.

## Standard usage

The recommended interaction methods are as follows:

1. You first say: `Start the first step` / `Start the second step` / `Start the third step`...
2. The orchestrator first returns the standard input template of this stage.
3. You complete the materials according to the template
4. AI then officially executes this stage

The standard principles are as follows:

- The template only retains the most basic input items in the current stage
- Does not require manual filling of `focus`
- Each Agent completes the complete analysis scope of the corresponding stage by default
- If `output_dir` is provided, the stage results should be placed as standard products as much as possible
- If `output_dir` does not exist, it should be created before writing the results.

The default product principles are as follows:

- As long as the current stage has obtained enough input materials and the user provides `output_dir`, the stage results should be written to this directory as much as possible
- Especially the first stage, after providing `jadx_mcp=yes + output_dir` or `target_dir + output_dir`, should generate by default:
  - `file_inventory.json`
  - `tech_stack.json`
  - `entrypoints.json`
  - `env_guard_report.json`
- Unless the user explicitly states "only oral analysis, no documentation required", the first stage should not only give a summary of the conversation

Regarding `env_guard_report.json`, the default requirements are as follows:

- This file must be output even if no explicit Root/Frida/Proxy/SSL Pinning blocking logic is found
- A clear distinction must be made in the document:
  - `confirmed`
  - `suspected`
  - `sdk_signal_only`
  - `not_confirmed_yet`
  - `not_observed`
- It is not allowed to omit the environment verification results because "I haven't seen them yet"
- It is not allowed to write "unconfirmed" as "no problem"

## Unified output root directory

Path convention:

- You can fill in the real path for the user's own sample directory, packet capture directory, and output directory.
- The technical files, rule files, and template files within this repository are all referenced using relative paths using `ai-mobile-reverse-skills/` as the root directory.
- The absolute directory of any personal computer should not be fixed in the README and the rules of each stage.

Starting from the first step, once you identify the `output_dir`, it should be considered the unified output root directory for the entire process.

The recommended directory structure is as follows:

```text
analysis_runs/current_run/
├── step1/
├── step2/
├── step3/
├── step4/
├── step5/
└── step6/
```

The default convention is as follows:

- The result of the first step is written into `step1/`
- The results of the second step are written to `step2/`
- The third step result is written to `step3/`
- The fourth step result is written to `step4/`
- The result of the fifth step is written into `step5/`
- The result of step 6 is written into `step6/`

That is, when you first provide:

```text
output_dir: analysis_runs/current_run
```

Subsequent placement should follow the following path by default:

- `analysis_runs/current_run/step1/`
- `analysis_runs/current_run/step2/`
- `analysis_runs/current_run/step3/`
- `analysis_runs/current_run/step4/`
- `analysis_runs/current_run/step5/`
- `analysis_runs/current_run/step6/`

The benefits of doing this are:

- The product boundary of each step is clearer
- The subsequent stage knows which stage directory to go to to read the previous-stage results
- The output directory is more suitable for automatic connection and delivery
- Avoid all JSON/Markdown tiles in the same directory

It is recommended to also keep in the root directory:

- `analysis_state.json`

Used to record which step is currently executed, the status of each step, and the product path of each stage.

### What is `analysis_state.json`

`analysis_state.json` is the process state file of the current task.
It is not responsible for saving vulnerability conclusions or protocol restoration results, but is responsible for recording:

- Who is on the current task;
- What is the current operating mode;
- Which stage is currently being executed;
- Which stages have completed, hung, blocked or failed;
- Where are the products at each stage written:

Its main uses are:

- Let the system know where the current process should stop and where it should continue;
- Provide switching basis for automatic chain A / B / C;
- Help the process recover from the correct stage after interruption, failure or human intervention;
- Avoid repeatedly asking about the location of the product in the previous stage in subsequent stages.

During normal use, testers do not need to manually maintain `analysis_state.json`.
It should be automatically generated and updated by the orchestrator during the process running, as a status record file for the entire workflow.

To view the complete field definitions, stage status values ​​and automatic chain switching conditions, please refer to:

- [docs/STATE-MODEL.md](docs/STATE-MODEL.md)

### Recommended parameters to configure before first use

If you need to automatically converge so in the future, and automatically pull the target .so from the APK unpacking source code and import it into Ghidra, it is recommended to confirm it at the beginning of the task:

- `ghidra_root`

At the same time, you need to ensure that the current task has the APK unpacking source code directory, and `lib/<abi>/*.so` can be accessed in the directory.
The decompiled code is used to determine "why this so was analyzed", and the APK unpacking source code directory is used to actually automatically pull the target .so from `lib/`. When either side is missing, analysis can only be downgraded and the full SO automation link cannot be declared complete. If the user explicitly provides `.so`, it can be used as native analysis material, but it cannot be called "automated pulling of so".

Among them, `ghidra_root` is the local Ghidra installation root directory, which usually cannot be automatically guessed by the system stability. Therefore, it is recommended that the user manually confirm it before using it for the first time, and then inherit it uniformly.

Currently, Ghidra's automatic import is only adapted to macOS; the automatic pull-up, project opening and GUI behavior under Windows/Linux have not yet been adapted and verified.

The remaining content related to Ghidra automatic import, such as:

- `ghidra_project_dir`
- `ghidra_project_name`
- `so_search_roots`
- `preferred_abis`

The default should be automatically derived by the system based on `{output_dir}`, the sample directory structure and the actual distribution of so, and the user is not required to fill it in manually.

## The path only needs to be specified once

In order to reduce repeated typing, this set of skills now uses the "session-level path inheritance" rule by default.

In other words, as long as you have explicitly provided:

- `target_dir`
- `output_dir`
- `traffic_source`
- `native_analysis_source`
- `target_name`

It will be automatically used by default in subsequent stages, so you don’t need to write it again every step.

For example:

### First statement

```text
step: 1
analysis_mode: local_source
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
jadx_mcp: no
```

### Follow up to continue

You can directly say later:

```text
Start the second step
```

If there is no explicit replacement path, the orchestrator should continue to use it by default:

- `target_dir: sample_target/decompiled`
- `output_dir: analysis_runs/current_run`

Same reason:

- Inherit `target_dir` and `output_dir` by default when starting the third step
- When entering the fourth step, `output_dir` will be inherited by default
- Step 6 inherits `target_name`, `report_type`, `include_appendix` by default (if given before)
- The `step1` to `step6` subdirectories in the unified output root directory will also be used by default, without the need to re-specify each step.

##Standard input template for each step

These templates are used to enter specific stages.
If you have declared the pattern earlier, then subsequent stages only need to fill in the missing materials of the current stage; the path and preorder context are automatically inherited by default.

### The first step is to enter the template

```text
step: 1
analysis_mode: jadx_mcp_session/local_source
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
jadx_mcp: yes/no
```

### The second step is to enter the template

```text
step: 2
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
mcp: burp/yakit/none
traffic_source: analysis_runs/current_run/traffic/traffic.json
```

### The third step is to enter the template

```text
step: 3
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
mcp: ida-mcp/ghidra-mcp/none
native_analysis_source: sample_target/native/sample.i64_or_sample.gpr_or_libsample.so
```

Additional instructions:

- If `mcp = ghidra-mcp` and you want to automatically import the target .so, you should have confirmed `ghidra_root` before starting the task
- It is recommended to write such native-related parameters into `analysis_state.json` instead of temporarily adding them in the third stage.

### The fourth step is to enter the template

```text
step: 4
output_dir: analysis_runs/current_run
allow_reanalyze_code: yes/no
```

### The fifth step is to enter the template

```text
step: 5
output_dir: analysis_runs/current_run
authorized_only: yes/no
```

### Step 6 Enter template

```text
step: 6
output_dir: analysis_runs/current_run
target_name: project name
report_type: brief/full
include_appendix: yes/no
```

## Sample dialogue

### Usage 1: Select the mode first, then enter the first step

For example, you first select:

```text
run_mode: auto_chain
auto_chain_mode: B
```

Then provide the next one:

```text
step: 1
analysis_mode: local_source
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
jadx_mcp: no
```

At this time the system should understand:

- The whole process still starts from the first step
- Manual confirmation is allowed in the first 1-3 stages
- Automatically advance from the fourth stage to the sixth stage

### Usage two: step by step mode

If you want to manually confirm each step, you can first provide:

```text
run_mode: step_by_step
```

Then enter the first step template. Just continue to say:

- `Proceed to step 2`
- `Proceed to step 3`
- `Proceed to step 4`

The system should suspend by default after each stage, waiting for manual review.

### Usage 3: Automatic chain A

If you want to automate the first phase and then from the second phase onwards, you can first provide:

```text
run_mode: auto_chain
auto_chain_mode: A
```

Then enter the first step template. At this point the system should:

- Complete the first stage first
- Wait for manual completion of packet capture preparation, MCP connection and other prerequisite actions
- Automatically advance from the second stage to the sixth stage

### Usage 4: Automatic chain C

If you have completed all manual preparations from the beginning, you can first provide:

```text
run_mode: auto_chain
auto_chain_mode: C
```

Then enter the first step template. At this point the system should:

- Execution starts from the first stage
- No skipping detection analysis
- Continue to automatically advance to subsequent stages when conditions are met

## Complete process example

A complete example from Phase 1 to Phase 6 is given below to make it clear to users:

- How to start each step
- What materials are needed for each step:
- What results will be precipitated at each step:
- How to consume the product of the previous step in the next step

### Sample background

Assume that the following materials currently exist:

- Decompiled directory: `sample_target/decompiled`
- Output directory: `analysis_runs/demo_app_run`
- Packet capture export file: `analysis_runs/demo_app_run/traffic/traffic.json`
- IDA project: `sample_target/native/demo_app.i64`

### Step 1: Start APK static reconnaissance

The user first said:

```text
Take the first step
```

The orchestrator first returns the template:

```text
step: 1
analysis_mode: jadx_mcp_session/local_source
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
jadx_mcp: yes/no
```

After the user completes it, it will be sent:

```text
step: 1
analysis_mode: local_source
target_dir: sample_target/decompiled
output_dir: analysis_runs/demo_app_run
jadx_mcp: no
```

By default this stage should be routed to:

- [agent-01-sample-recon.md](agents/agent-01-sample-recon.md)

At the end of this stage, at least:

- `file_inventory.json`
- `tech_stack.json`
- `entrypoints.json`
- `env_guard_report.json`

If you recognize the logic of environmental confrontation, you should also try to complete:

- `frida_bypass_plan.json`
- `frida/android_phase1_bypass.js`

### Step 2: Start aligning traffic with code

The user first said:

```text
Start the second step
```

The orchestrator first returns the template:

```text
step: 2
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
mcp: burp/yakit/none
traffic_source: analysis_runs/current_run/traffic/traffic.json
```

After the user completes it, it will be sent:

```text
step: 2
target_dir: sample_target/decompiled
output_dir: analysis_runs/demo_app_run
mcp: none
traffic_source: analysis_runs/demo_app_run/traffic/traffic.json
```

By default this stage should be routed to:

- [agent-02-protocol-mapper.md](agents/agent-02-protocol-mapper.md)

At the end of this stage, at least:

- `api_endpoints.json`
- `protocol_map.json`
- `traffic_alignment.json`

At the same time, we should try our best to complete the field-level results for direct consumption in subsequent stages, for example:

- `field_role`
- `builder_path`
- `crypto_entry_candidate`
- `related_endpoint_group`
- `matched_field_flows`

### Step 3: Start SO/JNI in-depth analysis

The user first said:

```text
Start step three
```

The orchestrator first returns the template:

```text
step: 3
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
mcp: ida-mcp/ghidra-mcp/none
native_analysis_source: sample_target/native/sample.i64_or_sample.gpr_or_libsample.so
```

After the user completes it, it will be sent:

```text
step: 3
target_dir: sample_target/decompiled
output_dir: analysis_runs/demo_app_run
mcp: none
native_analysis_source: sample_target/native/demo_app.i64
```

By default this stage should be routed to:

- [agent-03-crypto-native-analyzer.md](agents/agent-03-crypto-native-analyzer.md)

At the end of this stage, at least:

- `crypto_native_analysis.json`
- `jni_analysis.json`

At the same time, try to complete:

- `java_entry`
- `native_entry`
- `crypto_algorithm_candidate`
- `key_derivation`
- `iv_derivation`
- `salt_derivation`
- `input_order`
- `output_encoding`
- `restoration_confidence`

### Step 4: Start screening for weak encryption and high-risk vulnerabilities

The user first said:

```text
Start step four
```

The orchestrator first returns the template:

```text
step: 4
output_dir: analysis_runs/current_run
allow_reanalyze_code: yes/no
```

After the user completes it, it will be sent:

```text
step: 4
output_dir: analysis_runs/demo_app_run
allow_reanalyze_code: yes
```

By default this stage should be routed to:

- [agent-04-crypto-vuln-analyzer.md](agents/agent-04-crypto-vuln-analyzer.md)

At the end of this stage, at least:

- `vuln_analysis.json`
- `risk_matrix.json`

If the materials are sufficient, you should also try to complete:

- `secrets_report.json`
- `jsbridge_analysis.json`

This phase will prioritize consuming the field results from Phase 2 and Phase 3 instead of re-guessing from zero.

### Step 5: Start minimal verification POC design

The user first said:

```text
Start step five
```

The orchestrator first returns the template:

```text
step: 5
output_dir: analysis_runs/current_run
authorized_only: yes/no
```

After the user completes it, it will be sent:

```text
step: 5
output_dir: analysis_runs/demo_app_run
authorized_only: yes
```

By default this stage should be routed to:

- [agent-05-validation-designer.md](agents/agent-05-validation-designer.md)

At the end of this stage, at least:

- `validation_cases.json`
- `test_plan.md`
- `repro_steps.md`
- `poc_scripts_index.json`

If the materials are sufficient, you should also try to complete:

- `pocs/{vuln_id}/validate_request.py`
- `pocs/{vuln_id}/runtime_observe.js`
- `pocs/{vuln_id}/README.md`

### Step 6: Start penetrating report summary

The user first said:

```text
Start step six
```

The orchestrator first returns the template:

```text
step: 6
output_dir: analysis_runs/current_run
target_name: project name
report_type: brief/full
include_appendix: yes/no
```

After the user completes it, it will be sent:

```text
step: 6
output_dir: analysis_runs/demo_app_run
target_name: Demo App
report_type: full
include_appendix: yes
```

By default this stage should be routed to:

- [agent-06-reporter.md](agents/agent-06-reporter.md)

At the end of this stage, at least:

- `security_report.md`
- `findings.json`

And try to complete as many attachments as possible according to the material conditions:

- `api_endpoints_full.md`
- `secrets_full.md`
- `native_findings_full.md`

### What does this complete link explain:

The promotion method of this set of Skills is not "the user lets the AI ​​play freely with just one sentence", but:

1. The orchestrator returns to the stage template first.
2. The user completes the minimum input at this stage
3. The current stage Agent generates structured products
4. Continue to consume these products in the next stage

Therefore, this repository is more suitable for:

- Advancement of people-driven stage
- AI performs analysis according to procedures
- Standard results for each step of placing orders
-Finally form a complete delivery chain

## Standard workflow

### Step 1: Do APK static reconnaissance first

enter:

- `{target_dir}`
- or `jadx-mcp` is connected and the target sample is open in Jadx
- Optional: `jadx-mcp`

Target:

- Get `file_inventory.json`
- Get `tech_stack.json`
- Get `entrypoints.json`
- Get `env_guard_report.json`

### Step 2: Align traffic and code

enter:

- `{target_dir}`
- `{traffic_source}` or Burp / Yakit MCP

Target:

- Get `api_endpoints.json`
- Get `protocol_map.json`
- Get `traffic_alignment.json`

This stage will now try to complete a set of field-level evidence for direct consumption in step 4, including:

- `field_role`
- `location`
- `builder_path`
- `crypto_entry_candidate`
- `related_endpoint_group`
- `value_shape`
- `related_native_candidate`
- `replay_relevant`
- `matched_field_flows`

### Step 3: Do deep digging with SO and JNI

enter:

- `{native_analysis_source}` or so material
- Optional: `ida-mcp` / `ghidra-mcp`

Target:

- Get `crypto_native_analysis.json`
- Get `jni_analysis.json`

illustrate:

- The main path of Phase 3 is `ida-mcp` / `ghidra-mcp` to directly analyze so and JNI
- The local script only fills in the entry and bridging clues, and does not undertake to reverse the main process.

At this stage, we will try to complete a set of native restoration fields for direct consumption in step 4, including:

- `java_entry`
- `native_entry`
- `related_fields`
- `related_endpoints`
- `crypto_algorithm_candidate`
- `key_derivation`
- `iv_derivation`
- `salt_derivation`
- `input_order`
- `output_encoding`
- `restoration_confidence`

### Step 4: Screen for weak encryption and high-risk vulnerabilities

enter:

- Results of first 1-3 steps

Target:

- Get `vuln_analysis.json`
- Get `risk_matrix.json`
- Try to complete `secrets_report.json` as much as possible
- Try to complete `jsbridge_analysis.json` as much as possible

Step 4 is not a simple summary, but a priority to consume the fields that have been settled in steps 2 and 3, and then complete:

- Comprehensive restoration judgment of `sign/data/encryptData`
- Weak encryption and signature security assessment
- Top 10 risk audit on source-code side
- Packet-side risk linkage analysis

Therefore, in `vuln_analysis.json`, in addition to vulnerability entries, we will also try our best to fill in:

- `crypto_findings`
- `signature_findings`
- `crypto_restoration`
- `packet_risks`
- `source_phase_2_fields`
- `source_phase_3_fields`
- `gap_filled_by_phase4`

### Step 5: Make minimum verification POC design

enter:

- Vulnerability entries and risk matrix for Phase 4

Target:

- Get `validation_cases.json`
- Get `test_plan.md`
- Get `repro_steps.md`
- Get `poc_scripts_index.json`

If there are enough materials, we will try to get:

- `pocs/{vuln_id}/validate_request.py`
- `pocs/{vuln_id}/runtime_observe.js`
- `pocs/{vuln_id}/README.md`

### Step 6: Compile penetration report summary

enter:

- Results of first 1-5 steps

Target:

- Get `security_report.md`
- Get `findings.json`
- Get all the matching accessories

## How to use MCP

It is recommended to use it in the following ways:

- Phase 1: `jadx-mcp` or local directory analysis
- Phase 2:Burp MCP / Yakit MCP
- Phase 3:`ida-mcp` / `ghidra-mcp`
- Phase 4-6: Based on the previous results, call back the previous MCP if necessary

For detailed instructions, see:

- [MCP-INTEGRATION.md](docs/MCP-INTEGRATION.md)

## How to use the script

Scripts are used as an optional enhancement layer.

Currently available scripts:

- [endpoint_extractor.py](tools/scripts/endpoint_extractor.py)
- [secret_scanner.py](tools/scripts/secret_scanner.py)
- [native_bridge_indexer.py](tools/scripts/native_bridge_indexer.py)
- [env_guard_indexer.py](tools/scripts/env_guard_indexer.py)

use:

- `endpoint_extractor.py`: Supplement API, URL, deeplink, provider and other clues
- `secret_scanner.py`: Supplement hardcoded keys, tokens, certificates, cloud credentials and other clues
- `native_bridge_indexer.py`: Supplements JNI, WebView, JSBridge, Native loading clues, does not replace the so analysis of `ida-mcp` / `ghidra-mcp`
- `env_guard_indexer.py`: add Root, simulator, proxy, SSL Pinning, Frida, signature verification clues
- `tools/frida/android_phase1_bypass.js`: add Phase 1 runtime bypass template assets

For detailed instructions, see:

- [tools/scripts/README.md](tools/scripts/README.md)

## Directory structure

```text
ai-mobile-reverse-skills/
├── SKILL.md
├── README.md
├── docs/
│   └── MCP-INTEGRATION.md
├── agents/
│   ├── agent-01-sample-recon.md
│   ├── agent-02-protocol-mapper.md
│   ├── agent-03-crypto-native-analyzer.md
│   ├── agent-04-crypto-vuln-analyzer.md
│   ├── agent-05-validation-designer.md
│   └── agent-06-reporter.md
└── tools/
    ├── scripts/
    │   ├── README.md
    │   ├── endpoint_extractor.py
    │   ├── secret_scanner.py
    │   ├── native_bridge_indexer.py
    │   └── env_guard_indexer.py
    └── frida/
        ├── README.md
        └── android_phase1_bypass.js
```

## The connection relationship between Phase 2 / 3 / 4

The current main connection relationship is as follows:

- Phase 2 is responsible for aligning the capture fields with the code construction logic, focusing on spitting out `field_role`, `builder_path`, `crypto_entry_candidate`, `matched_field_flows`
- Phase 3 is responsible for opening up the link between Java -> JNI -> so, focusing on spitting out `java_entry`, `native_entry`, `crypto_algorithm_candidate`, `key_derivation`, `input_order`
- Phase 4 is responsible for consuming these fields first, rather than re-guessing them from scratch, and then synthesizing them into:
  - `crypto_restoration`
  - `packet_risks`
  - `vulnerabilities`

Therefore, step 4 assumes comprehensive closure responsibilities rather than pure risk list generation responsibilities.

## Deliver the product

When advanced according to the standard process, this set of Skills will gradually accumulate the following deliverable products:

- Phase 1:`file_inventory.json`, `tech_stack.json`, `entrypoints.json`
- Phase 2:`api_endpoints.json`, `protocol_map.json`, `traffic_alignment.json`
- Phase 3:`crypto_native_analysis.json`, `jni_analysis.json`
- Phase 4:`vuln_analysis.json`, `risk_matrix.json`
- Phase 5: `validation_cases.json`, `test_plan.md`, `repro_steps.md`, `poc_scripts_index.json` and `pocs/` split by vulnerability
- Phase 6: `security_report.md`, `findings.json` and supporting attachments
