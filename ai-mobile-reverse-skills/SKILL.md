---
name: ai-mobile-reverse-skills
description: A 6-stage orchestrator skill for mobile security analysis scenarios. Used for unified scheduling of APK static reconnaissance, traffic and code alignment, SO/JNI in-depth analysis, encryption and vulnerability comprehensive analysis, verification design and report delivery process. Supports JADX MCP, Burp/Yakit MCP, IDA/Ghidra MCP, using a combination of AI-led analysis and stage scripts.
---

# Skill: AI Mobile Reverse Skills

## position

This Skill is the root entrance of the entire mobile security analysis system and the only stage scheduling entrance.

It targets the following typical scenarios:

- Android App static reverse analysis
- Linked analysis of packet capture results and source code
- SO / JNI / native communication and encryption logic analysis
- Screening for high-risk issues such as weak encryption, authentication and authorization, component security, and business logic
- Minimum verification scheme design in authorized environment
- Penetration testing report and deliverable generation

It does not replace detailed analysis at specific stages, but is responsible for:

1. Identify the stage the user currently wants to perform.
2. Identify currently connected MCPs, provided input materials, and existing intermediates.
3. Check whether the minimum startup conditions of the current stage are met.
4. Route the task to the correct stage Agent.
5. Constrain each stage to proceed according to the unified input and output contract.

After entering a certain step, the corresponding stage file must be used as the direct execution standard.

## Operation mode

This Skill adopts a fixed 6-stage execution mode. All modes start from Phase 1, the only differences are:

- Whether to suspend by default after each stage, waiting for manual confirmation
- From which stage does automation take over the subsequent process:

The running portals are unified into two categories, and users are allowed to explicitly select them at the beginning:

- Phase 1: APK static reconnaissance
- Phase 2: Traffic and code alignment
- Phase 3: In-depth analysis of SO and JNI
- Phase 4: Weak encryption and high-risk vulnerability screening
- Phase 5: Minimal verification POC design
- Phase 6: Penetration report summary

### Mode selection rules

The user can explicitly declare at the beginning:

- `run_mode: step_by_step`
- `run_mode: auto_chain`

If `run_mode = auto_chain`, you can continue to declare:

- `auto_chain_mode: A`
- `auto_chain_mode: B`
- `auto_chain_mode: C`

Overall control execution principles:

1. If the user has explicitly declared the mode, the orchestrator must first execute according to this mode and no longer guess on its own.
2. If the user does not declare the mode:
- Enter `step_by_step` by default
- Enter `auto_chain` only when the user clearly expresses the meaning of "automatic advancement", "running out of one prompt word", "automatically continuing later", etc.
3. If the user declares `auto_chain` but does not declare `auto_chain_mode`, the orchestrator should infer based on the current earliest feasible entry point:
- If the user emphasizes "manually handle packet capture and pre-preparation after the first stage, and then automatically advance the follow-up", prefer `A`
- If the user emphasizes "manual control in the first 1-3 stages, automatic closing in the last 4-6", prefer `B`
- If the user emphasizes "automatically advance the entire process from the first stage", prefer `C`
4. No matter which mode is selected, the orchestrator must first execute Phase 1. It is not allowed to skip the first phase because the automatic chain is selected.

### Category 1: Stage-by-stage step-by-step mode

Corresponding value:

- `run_mode: step_by_step`

Applicable scenarios:

- Testers want to manually control the pace of each step
- Manually review the JSON/Markdown products after each stage
- Only after confirming the conclusion of the current stage can you enter the next stage

Operating mechanism:

1. The user first declares the operating mode.
2. The orchestrator still returns to the standard input template starting from Phase 1.
3. The user supplements the materials required for the current stage.
4. The orchestrator is routed to the corresponding Agent for execution.
5. After the current phase is completed, it will hang by default and wait for manual confirmation whether to continue to the next phase.

Manual intervention point:

- After each stage, you can manually stop the review
- You can manually adjust whether to continue the next stage, which step to continue to, and whether to replace input materials.

### Category 2: Automatic chain

Corresponding value:

- `run_mode: auto_chain`

The automatic link is further subdivided into the following three links. All three links start from Phase 1, but the starting point for automation to take effect is different.

#### Automatic chain A: Analysis loop

Corresponding value:

- `run_mode: auto_chain`
- `auto_chain_mode: A`

Applicable scenarios:

- Hope the whole process starts from Phase 1
- However, after the first stage, pre-operations such as detection bypass, proxy configuration, and packet capture preparation need to be completed manually.
- The user clearly stated that subsequent `burp-mcp` / `yakit-mcp` and `ghidra-mcp` / `ida-mcp` have been connected
- It is hoped that after these pre-actions are completed, the system will automatically advance to the subsequent stages.

Operating range:

- Phase 1 manual confirmation
- Phase 2 -> Phase 3 -> Phase 4 -> Phase 5 -> Phase 6 automatic advancement

Operating mechanism:

1. The orchestrator executes Phase 1 first.
2. After Phase 1 is completed, it will be suspended by default, waiting for manual completion of packet capture and other pre-preparations.
3. After the user confirms that "preparation has been completed", the general control checks:
- Does Phase 1 result exist:
- Whether the packet capture material is actually available or whether the packet capture MCP is available
- Native analyzes whether MCP or local native materials are available
4. Even if the user has manually completed the pre-operation, this Skill must still do pre-check first, instead of directly skipping the detection conclusion, packet capture material and JNI/so clue judgment.
5. If the conditions are met, it will automatically advance from Phase 2 to Phase 6.
6. If the key condition does not hold, pause at the missing point and clearly indicate the blocking item.

Manual intervention point:

- Manually responsible for packet capture and pre-environment preparation after Phase 1
- Once the user confirms that "preparation has been completed", the subsequent analysis chain will automatically advance by default

#### Automatic chain B: Verification closed loop

Corresponding value:

- `run_mode: auto_chain`
- `auto_chain_mode: B`

Applicable scenarios:

- Hope the whole process starts from Phase 1
- The first 1-3 stages are manually led and gradually confirmed
- No longer stops starting from Phase 4, but automatically advances to the verification and reporting phase

Operating range:

- Phase 1 -> Phase 2 -> Phase 3 manual confirmation
- Phase 4 -> Phase 5 -> Phase 6 automatic advancement

Operating mechanism:

1. The overall control still starts from Phase 1.
2. Phase 1, 2, and 3 will hang by default after the end of each phase, waiting for manual confirmation whether to continue.
3. After entering Phase 4, the orchestrator checks whether `vuln_analysis.json`, `risk_matrix.json` and related previous-stage materials exist.
4. If the conditions are met, it will automatically enter Phase 5 from Phase 4, and then automatically enter Phase 6.
5. If key materials required for design verification or reporting are missing, the system will pause and prompt for missing items.

Manual intervention point:

- Manually responsible for completing and confirming the first 1-3 stages
- Manually can cancel the automatic chain before entering Phase 4 and execute it step by step throughout the process.

#### Automatic chain C: through train

Corresponding value:

- `run_mode: auto_chain`
- `auto_chain_mode: C`

Applicable scenarios:

- Testers have completed all manual preparations from the beginning
- `jadx-mcp`, `burp-mcp` / `yakit-mcp`, `ghidra-mcp` / `ida-mcp` are all connected
- The packet capture results are ready
- Want to automatically advance from Phase 1 to Phase 6

Operating range:

- Phase 1 -> Phase 2 -> Phase 3 -> Phase 4 -> Phase 5 -> Phase 6

Operating mechanism:

1. The orchestrator first executes Phase 1 to generate APK static reconnaissance results.
2. Even if the user has manually confirmed that "packet capture is feasible", this Skill must still perform Phase 1 detection and entrance identification, and cannot skip sample portraits and environmental detection checks.
3. After Phase 1 is completed, the orchestrator checks the following conditions:
- Whether the packet capture material already exists or the packet capture MCP is available
- Native analyzes whether the MCP is connected
- Whether there is minimum input to enter Phase 2 and Phase 3
4. If the conditions are met, it will automatically enter `auto_chain` and continuously advance from Phase 2 to Phase 6.
5. If the condition is not met, it will pause at the earliest blocking stage and prompt that the condition is missing.

Manual intervention point:

- Manually responsible for completing MCP access, packet capture preparation and other pre-environment operations before startup
- Once the through-train mode is started, subsequent stages will advance automatically by default unless the user actively interrupts

### General principles of automatic connection

When the user clearly expresses "only enter once, subsequent automatic connections", "try to advance to the report as continuously as possible", and "run from beginning to end", the orchestrator should give priority to enter the `auto_chain` mode.

In `auto_chain` mode, the master must:

1. Collect the minimum required input only once at the beginning.
2. Automatically identify existing intermediate products under `{output_dir}` to avoid repeated inquiries.
3. After each stage, it is automatically determined whether the minimum conditions for the next stage are met.
4. If the conditions are met, proceed directly to the next stage.
5. If conditions are not met, only pause when critical input is truly missing and clearly indicate what is missing.
6. The default goal is to advance to Phase 6 from the earliest feasible phase currently available.

The default principles are as follows:

- Only the most basic input items are required by default
- Does not require the user to manually specify `focus`
- Each Agent completes the standard analysis scope of this stage by default
- When `output_dir` is provided, priority should be given to placing standard products instead of just outputting the conversation summary
- If `output_dir` does not exist, it should be created first and then write the results.

Additional instructions:

- The current repository provides Skill orchestrator specifications, stage Agent documents and local index scripts
- There is currently no built-in standalone CLI, conversation router, or "one-click complete 6-stage" program entrance
- The "orchestrator returns the template and routes" mentioned in the document refers to the execution of the upper-level AI/Skill according to this file, rather than the separate command line program already in the repository.

## Pattern startup template

The following template is used to explicitly declare the run mode at the start of a task.

### Step 1: Declare the pattern first

#### Template 1: Stage-by-stage stepping mode

```text
run_mode: step_by_step
```

illustrate:

- The whole process starts from the first step
- Suspend by default after each stage
- Wait for manual review of the current stage of the product before continuing

#### Template 2: Automatic chain A (analytical closed loop)

```text
run_mode: auto_chain
auto_chain_mode: A
```

illustrate:

- The whole process starts from the first step
- After the first phase, wait for manual completion of packet capture preparation, MCP connection and other pre-processing actions
- Automatically advance from the second stage to the sixth stage

#### Template 3: Automatic chain B (verification closed loop)

```text
run_mode: auto_chain
auto_chain_mode: B
```

illustrate:

- The whole process starts from the first step
- The first 1-3 stages are manually confirmed by default
- Automatically advance from the fourth stage to the sixth stage

#### Template 4: Automatic chain C (through train)

```text
run_mode: auto_chain
auto_chain_mode: C
```

illustrate:

- The whole process starts from the first step
- It is assumed from the beginning that all manual preparations have been completed
- Start from the first stage and continue to advance to the sixth stage according to the automatic chain
- Through-train mode must still perform Phase 1 sample reconnaissance and testing confirmation first

### Step 2: Enter the first stage

No matter which mode you choose, you should first enter the first stage template:

```text
step: 1
analysis_mode: local_source/jadx_mcp_session
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
jadx_mcp: yes/no
```

Additional instructions:

- If you select `auto_chain_mode: A` or `auto_chain_mode: C`, you can continue to add packet capture, Native, and report related parameters after the first stage template.
- If you select `auto_chain_mode: B`, you can complete the first stage first, and then add verification and reporting parameters before entering the fourth stage.

## Timing of use

This Skill should be enabled when the user wants to analyze any of the following materials:

- Android directory after unpacking and decompilation
- Jadx decompilation results
- Burp / Yakit / mitmproxy packet capture results
- .so files, JNI clues, IDA/Ghidra projects
- `raw_*.json`, `*_analysis.json`, `report.md` output in the previous stages

## The iron law of total control and execution

1. Only the current 6 stages are allowed to be executed. New main stages are not allowed, the order is not allowed to be modified, and the names are not allowed to be changed.
2. Phase 1 only analyzes the unpacked and decompiled materials that have been obtained, and cannot be expressed as "this Skill is responsible for unpacking".
3. Advance in the default order; if the user explicitly requests "Start Step N" and the materials are sufficient, you can directly enter this step.
4. "AI-led analysis + stage script combination" is adopted by default:
- If the current MCP and materials are sufficient, priority will be given to directly performing stage analysis by AI.
- When the stage takes the local code analysis path, the corresponding script is executed by default to complete batch indexing and structured raw hits.
- Scripts do not replace stage analysis conclusions.
5. All conclusions must be tied to evidence as much as possible:
- code path
- Line number or symbol name
- Packet capture field
- JNI/so pseudocode snippet
- or `raw_*.json` raw hits
6. If there is insufficient material at the current stage, you can only point out the missing items and what needs to be prepared for the next step, but cannot make up the context.
7. After the current stage is completed, we should try our best to produce standard products that can be consumed in the next stage.
8. Phase 1 When the user provides `{target_dir}` and `{output_dir}`, it should generate by default:
   - `file_inventory.json`
   - `tech_stack.json`
   - `entrypoints.json`
   - `env_guard_report.json`
Unless the user explicitly requests "verbal analysis only, no writing of documents", it is not allowed to just summarize the conversation without writing.
9. `{output_dir}`, once confirmed in the first input, shall be regarded as the unified output root directory for the entire session and shall not be changed at will in subsequent stages unless explicitly overridden by the user.
10. After the unified output of the root directory is confirmed, the orchestrator should default to a staged directory placement method instead of placing all artifacts flat in the root directory.

## Unify input variables

The orchestrator gives priority to identifying and using the following variables:

- `{target_name}`: target App name
- `{apk_path}`: APK path, optional, only used to supplement sample meta information
- `{target_dir}`: directory after unpacking and decompilation
- `{traffic_source}`: Burp / Yakit / mitmproxy export results
- `{native_analysis_source}`: IDA/Ghidra project, pseudocode or so sample path
- `{output_dir}`: output directory
- `raw_*.json`: script index results
- `*_analysis.json`: stage analysis results
- `report.md` / `security_report.md`: final report class document

## Session-level path inheritance rules

Once the user explicitly provides the following paths or identifiers in the current task, the orchestrator MUST treat them as the default context for the current session, automatically inherited by subsequent phases unless the user explicitly overrides:

- `{target_name}`
- `{apk_path}`
- `{target_dir}`
- `{traffic_source}`
- `{native_analysis_source}`
- `{output_dir}`

The inheritance principles are as follows:

1. After the user provides it for the first time, he or she shall not be asked to fill in the same path again in subsequent stages.
2. If the user only says "continue to the second step", "continue to the third step" or "continue to the fourth step", the orchestrator should give priority to reusing the recorded path context.
3. If the user activates the `4/5/6` integrated mode, he only needs to specify `{output_dir}` once at the beginning, and subsequent Phase 5 and Phase 6 will automatically inherit it.
4. If the user provides a new path, the old value will be overwritten with the latest value.
5. If a certain type of path is missing at the current stage, but can be inferred from existing products, for example, if it is known that there is a preamble file in `{output_dir}`, priority should be given to inferring from the known context instead of requiring the user to fill it in manually again.

## Unify output directory rules

When the user first provides `{output_dir}`, the orchestrator must treat it as the "unified output root directory" and create the following stage subdirectories under it by default:

- `{output_dir}/step1`
- `{output_dir}/step2`
- `{output_dir}/step3`
- `{output_dir}/step4`
- `{output_dir}/step5`
- `{output_dir}/step6`

The default output locations of each stage are as follows:

- Phase 1 -> `{output_dir}/step1/`
- Phase 2 -> `{output_dir}/step2/`
- Phase 3 -> `{output_dir}/step3/`
- Phase 4 -> `{output_dir}/step4/`
- Phase 5 -> `{output_dir}/step5/`
- Phase 6 -> `{output_dir}/step6/`

Default requirements:

1. Before starting the first stage, ensure that the unified output root directory exists.
2. When executing the first stage, at least `step1` should be created, and it is recommended to pre-create `step2` to `step6` at the same time.
3. By default, subsequent stages will only write products to their own stage directory.
4. When reading previous-stage products in subsequent stages, priority should be given to reading them from the corresponding `stepN/` directory instead of requiring the user to re-specify the file path.
5. If the repository or user already has an old version of the "flat output" result, it can be read compatible, but the new default writing method should be a staged directory.

Product path alias rules:

- The new process parses Phase 1 products into `{output_dir}/step1/<artifact>` by default.
- Phase 2 artifacts are resolved to `{output_dir}/step2/<artifact>` by default.
- Phase 3 artifacts are resolved to `{output_dir}/step3/<artifact>` by default.
- Phase 4 artifacts are resolved to `{output_dir}/step4/<artifact>` by default.
- Phase 5 artifacts are resolved to `{output_dir}/step5/<artifact>` by default.
- Phase 6 artifacts are resolved to `{output_dir}/step6/<artifact>` by default.
- The old version of root directory tiling products can be compatible when reading, but by default no longer can be tiled to the root directory when writing.

It is recommended to maintain in the root directory at the same time:

- `{output_dir}/analysis_state.json`

For logging:

- current target name
- Session level path context
- The current stage of execution
-Status of each stage
- Product paths at each stage

### `analysis_state.json` Role description

`analysis_state.json` is the status record file of the entire 1-6 stage workflow. It is not responsible for saving specific analysis conclusions, but is responsible for recording the running status of the current task.

Its main functions are:

- Record the basic context of the current task, such as `{target_name}`, `{target_dir}`, `{output_dir}`, running mode and automatic chain mode;
- Record the current stage of execution, and whether each stage is `pending`, `running`, `waiting_review`, `completed`, `blocked` or `failed`;
- Provide switching basis for automatic chain A / B / C to determine whether it is currently allowed to automatically enter the next stage;
- Provide a basis for interruption recovery so that the process can continue from the correct stage after hanging, blocking or failing;
- Index the output directory of each stage to avoid repeated inquiry of the previous product path in subsequent stages.

The responsibilities are divided as follows:

- `agents/*.md` is responsible for performing analysis at each stage;
- `analysis_state.json` is responsible for recording process status;
- The orchestrator is responsible for updating this file when a stage starts, ends, hangs, blocks, or fails.

Minimum field suggestions include:

- `target_name`
- `run_mode`
- `auto_chain_mode`
- `target_dir`
- `output_dir`
- `current_phase`
- `overall_status`
- `manual_ready`
- `phases`

### Stage status field

The following status values ​​should be used by default for orchestrator and each stage:

- `pending`: not started yet
- `running`: executing
- `waiting_review`: The current stage has been executed and is waiting for manual confirmation
- `completed`: The current stage has been completed and is available for downstream consumption
- `blocked`: missing input, unsatisfied conditions, or paused by manual request
- `failed`: Execution error or result unavailable

The default rules are as follows:

- When `run_mode = step_by_step`, after the stage execution is completed, the default value is written as `waiting_review`
- After manual confirmation is allowed to continue, the current stage is written as `completed`
- When `run_mode = auto_chain`, after the switching conditions are met, the current stage is written as `completed` and automatically enters the next stage.
- When key material is missing, write `blocked` instead of pretending to move forward.
- When an execution error is reported or the key product is invalid, it should be written as `failed`

### Automatic chain switching conditions

The automatic chain does not "jump automatically when the time is up", but must meet the prerequisites before entering the next stage.

#### Link A

Applicable logic:

- Phase 1 is executed normally and manually confirmed
- After manually completing packet capture preparation, MCP connection and other prerequisite actions
- Automatically advance from Phase 2 to Phase 6

Minimum switching conditions:

- `phase_1.status = completed`
- `manual_ready.traffic_ready = true`
- `manual_ready.mcp_burp_ready = true` or `{traffic_source}` provided
- `manual_ready.mcp_native_ready = true` or `{native_analysis_source}` provided

#### Link B

Applicable logic:

- Phase 1-3 is gradually confirmed manually
- Automatically advance from Phase 4 to Phase 6

Minimum switching conditions:

- `phase_1.status = completed`
- `phase_2.status = completed`
- `phase_3.status = completed`

#### Link C

Applicable logic:

- Execution starts from Phase 1
- All environment, packet capture and MCP preparations have been completed manually before starting
- After meeting the conditions, it will continue to automatically advance to Phase 6

Minimum switching conditions:

- `analysis_mode = local_source` or `manual_ready.mcp_jadx_ready = true`
- `manual_ready.traffic_ready = true`
- `manual_ready.mcp_burp_ready = true` or `{traffic_source}` provided
- `manual_ready.mcp_native_ready = true` or `{native_analysis_source}` provided
- `phase_1.status = completed`

Additional requirements:

- All automatic chains must first execute Phase 1, and sample reconnaissance and detection confirmation must not be skipped
- If any condition is not met, write `blocked` and wait for manual processing instead of continuing to jump.
- If the product exists but key fields are missing, it should be entered into `blocked` or `failed` first and shall not be regarded as `completed`

## Main stage file

Orchestrator can only route to the following files:

1. `agents/agent-01-sample-recon.md`
2. `agents/agent-02-protocol-mapper.md`
3. `agents/agent-03-crypto-native-analyzer.md`
4. `agents/agent-04-crypto-vuln-analyzer.md`
5. `agents/agent-05-validation-designer.md`
6. `agents/agent-06-reporter.md`

## Stage routing table

| Stage | Frequently Asked Questions | Main Tool Mode | Recommended MCP | Minimum Input Requirements | Recommended Products |
|---|---|---|---|---|---|
| Phase 1 | "Start the first step" "Do static reconnaissance first" "I have connected jadx-mcp" | `jadx-mcp` session analysis or local directory analysis | `jadx-mcp` or no MCP | `jadx_mcp=yes` or `{target_dir}` | Default generation `file_inventory.json` `tech_stack.json` `entrypoints.json` `env_guard_report.json` |
| Phase 2 | "Start the second step" "Align traffic and code" "I have connected Burp MCP" | Packet capture and code mapping | Burp MCP / Yakit MCP | `{target_dir}` + capture MCP or `{traffic_source}` | `api_endpoints.json` `protocol_map.json` `traffic_alignment.json` |
| Phase 3 | "Start the third step" "Analyze so/JNI" "I have connected ida-mcp" "I have connected ghidra-mcp" | so / JNI Deep Digging | `ida-mcp` / `ghidra-mcp` | `{native_analysis_source}` or so related materials | `crypto_native_analysis.json` `jni_analysis.json` |
| Phase 4 | "Start the fourth step" "Do vulnerability screening" | Summary risk judgment | No mandatory MCP, call back `jadx-mcp` when necessary | At least part of the results of the first 1-3 phases | `vuln_analysis.json` `risk_matrix.json`, and try to complete `secrets_report.json` `jsbridge_analysis.json` |
| Phase 5 | "Start the fifth step" "Do minimum verification" "Design POC" | Verification solution design and minimum script template generation | Burp / Yakit MCP optional | Phase 4 results | `validation_cases.json` `test_plan.md` `repro_steps.md` `poc_scripts_index.json` |
| Phase 6 | "Start the sixth step" "Summary report" "Output report" | Delivery and summary | No mandatory MCP | Results of the first 1-5 phases | `security_report.md` `findings.json`, etc. |

illustrate:

- The "recommended products" in the table should be written to the corresponding `stepN/` directory by default.
- If a unified output root directory is used, these files should no longer be written directly to the `{output_dir}` root.

## Routing rules

### Rule 1: Prioritize identification of explicit stages

If the user explicitly says "First Step/Second Step/Third Step/Fourth Step/Fifth Step/Sixth Step", priority will be given to routing according to the user-specified stage.

A typical statement is as follows:

- "Help me take the first step"
- "I am now connected to jadx-mcp, let's take the first step"
- "I am now connected to Burp MCP, continue to step 2"
- "I am now connected to ida-mcp, do the third step"
- "I am now connected to ghidra-mcp, do the third step"
- "Help me do the fourth step of vulnerability screening"
- "Help me design the fifth step minimal verification POC"
- "Help me put together the sixth step report"

If the user just says:

- "Take the first step"
- "Start step two"
- "Start step three"
- "Start Step 4"
- "Start step five"
- "Start step six"

If sufficient input materials are not provided, the general control ** must not start analysis directly **. It must first return to the standard input template of this stage and wait for the user to complete it before executing.

### Rule 2: When there is no explicit stage, the earliest feasible stage is inferred based on the material.

If the user does not specify which step to take, the judgment will be in the following order:

1. If `{target_dir}` is provided, priority is given to starting from Phase 1 in "local decompile/unpack source directory mode".
2. If `jadx-mcp` is connected and the target sample has been opened in Jadx, it is preferable to start from Phase 1 in "`jadx-mcp` session mode".
3. If `{target_dir}` is not provided and `jadx-mcp` is not connected, it must be clearly stated that the current repository is not responsible for unpacking or decompilation by itself, and the user is required to complete `jadx-mcp` or the unpacked and decompiled directory.
4. If only packet capture materials and code directories are provided, start from Phase 2.
5. If only IDA/Ghidra/so materials are provided, start from Phase 3.
6. If `vuln_analysis.json` and `risk_matrix.json` are provided, you can start from Phase 5 or Phase 6.
7. If there is ambiguity, give priority to an earlier stage rather than jumping ahead without permission.

### Rule 3: Stage pre-check

Before entering a certain stage, the general control should at least conduct the following checks:

#### Phase 1

Just satisfy one of the following:

- Yes `{target_dir}`
- There is `jadx_mcp = yes` and the target sample is opened in Jadx

Phase 1 must explicitly distinguish between two types of analysis:

- `analysis_mode = jadx_mcp_session`: Connect the current Jadx session through `jadx-mcp` to analyze the opened sample
- `analysis_mode = local_source`: directly analyze the decompiled source code or the directory after unpacking the APK

If `{output_dir}` is provided at the same time, this stage will write out in this directory by default:

- `file_inventory.json`
- `tech_stack.json`
- `entrypoints.json`

If `{output_dir}` does not exist, the directory should be created before writing the result file.

#### Phase 2

Must have:

- `{target_dir}`

And meet one of the following:

- Burp MCP connected
- Yakit MCP connected
- Provide `{traffic_source}`

#### Phase 3

Just satisfy one of the following:

- Connected `ida-mcp`
- `ghidra-mcp` is connected
- Provide `{native_analysis_source}`
- Provide so/JNI pseudocode materials that can be directly analyzed

If you need to enable SO automation links, including `resolve_native_target.py` to converge the target .so and `ghidra_target_loader.py` to automatically import Ghidra, you must have both:

- Decompile code context: `{target_dir}` or `jadx_mcp_session`
- APK unpacking source code context: the APK unpacking directory must exist and `lib/<abi>/*.so` in it must be accessible

In the absence of decompiled code context, we can only perform single-file native analysis, and cannot claim to have completed Java -> JNI -> business field link analysis.
When the APK unpacking source code directory is missing, you can converge on candidate so names, or analyze `.so` explicitly provided by the user, but you cannot claim to have automatically pulled so. Explicit `.so` only belongs to user-specified native analysis material and does not belong to the "automated pull" link.

#### Phase 4

You should try to have some of the following materials:

- Phase 1 entrance and environmental reconnaissance results
- Phase 2 API and protocol mapping results
- JNI/native analysis results for Phase 3

#### Phase 5

One of the following should already be present:

- `vuln_analysis.json`
- `risk_matrix.json`
- Confirmed or pending vulnerability entries

#### Phase 6

You should already have at least some of the results from the previous stages 1-5.

If the prerequisites are not met, the orchestrator must point out what is missing rather than push forward reluctantly.

### Rule 4: Give the template first, then execute it

When a user enters a certain stage but does not have all the materials, the general control must take the following actions first:

1. Identify the current stage number
2. Return the standard input template of this stage
3. Make it clear which fields are required and which are optional
4. Wait for the user to complete the request and then route to the corresponding Agent.

The general control officer shall not directly conduct "guess-making analysis" when the material is obviously missing.

## MCP usage specifications

### Phase 1: APK static reconnaissance

- Preferred: `jadx-mcp`
- Alternative: No MCP, use VS Code/local directory search directly

This stage supports two analysis methods:

- `jadx_mcp_session`: Directly read the Manifest, class, method, string, resource and call thread of the currently opened Jadx sample through `jadx-mcp`
- `local_source`: directly analyze the local directory that has been unpacked, decompiled or unpacked

By default at this stage, at least the following products should be placed:

- `file_inventory.json`
- `tech_stack.json`
- `entrypoints.json`
- `env_guard_report.json`

Among them, `env_guard_report.json` cannot be omitted because "no explicit detection logic is found". If only indirect signals such as shells, risk-control SDKs, and native security libraries are confirmed at the current stage, the following should also be clearly written:

- `confirmed`
- `suspected`
- `sdk_signal_only`
- `not_confirmed_yet`
- `not_observed`

One of the other states is used for the second step of packet capture preparation and the fourth step of risk closing for direct consumption.

The core of this stage is not "executing the decompilation tool", but analyzing the materials that have been obtained:

- Manifest
- Permissions and components
- Third-party SDK
- Environmental confrontation logic
- Signature/Encryption/Token/JNI/WebView entry

Under the premise that `{output_dir}` has been provided, the key results should be structured in this stage by default instead of just staying in the dialogue:

- `file_inventory.json`
- `tech_stack.json`
- `entrypoints.json`

If `{output_dir}` does not currently exist, it should be created first and then placed.

### Phase 2: Traffic and code alignment

- Preferred: Burp MCP / Yakit MCP

At this stage, MCP is mainly used to obtain:

- Request list
- Header/Query/Body fields
- Response characteristics
- Classified traffic such as login, payment, information, upload, etc.

Then with the code directory:

- BaseURL
- Retrofit / OkHttp
- request wrapper
- Sign / token / data / timestamp and other construction logic

Do the mapping.

The most important thing at this stage is not to simply get the API list, but to try to spit out field-level evidence that can be directly consumed by Phase 4, for example:

- `field_role`
- `location`
- `builder_path`
- `crypto_entry_candidate`
- `related_endpoint_group`
- `value_shape`
- `related_native_candidate`
- `replay_relevant`
- `matched_field_flows`

### Phase 3: In-depth analysis of SO and JNI

- Preferred: `ida-mcp` / `ghidra-mcp`
- Main path: directly drive IDA/Ghidra analysis so and JNI logic through `ida-mcp` or `ghidra-mcp`
- The local script is only responsible for filling in the JNI / bridge / loadLibrary clues and is not used as the main means of so reverse analysis.

At this stage, MCP is mainly used to obtain:

- JNI portal
- Cross-reference
- so pseudocode
- Key function context
- Algorithm and native protection logic clues

The output of this stage should try to complete the restoration fields that can be directly consumed by Phase 4, for example:

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

### Phase 4-6

- No mandatory MCP
- Main consumption preorder results
- If you need to supplement evidence, you can call back the previous MCP, but you cannot take the opportunity to rewrite the current 6-stage main process.

in:

- Phase 4 In addition to `vuln_analysis.json` and `risk_matrix.json`, if `raw_secrets.json` or WebView / JSBridge related clues already exist, try to complete `secrets_report.json` and `jsbridge_analysis.json`
- Phase 6 should give priority to consuming the above-mentioned completed structured products instead of merging them from scratch in the reporting stage.

## Script calling specification

The default rules are as follows:

- When Phase 1 adopts the `local_source` path, the following 4 local index scripts are executed by default:
  - `tools/scripts/endpoint_extractor.py`
  - `tools/scripts/secret_scanner.py`
  - `tools/scripts/native_bridge_indexer.py`
  - `tools/scripts/env_guard_indexer.py`
- When Phase 1 adopts the `jadx_mcp_session` path, the above four scripts are not executed by default, and priority is given to consuming the `jadx-mcp` context directly.
- `tools/scripts/resolve_native_target.py` and `tools/scripts/ghidra_target_loader.py` continue to execute according to the original automation logic and are not affected by the above rules

Corresponding functions:

- `endpoint_extractor.py`: extract URL, API path, BaseURL, deeplink, provider, network security config
- `secret_scanner.py`: Extract keys, tokens, certificates, cloud credentials, debugging traces, and intranet clues
- `native_bridge_indexer.py`: Extract JNI, WebView, JSBridge, Native loading and bridging clues, only as a pre-clue supplement for Phase 3
- `env_guard_indexer.py`: Extract Root, simulator, proxy, SSL Pinning, Frida, signature verification, integrity verification clues
- `tools/frida/android_phase1_bypass.js`: Provides Root / emulator / proxy / SSL Pinning basic bypass template in an authorized environment

## Stage promotion standards

After each stage of general control, it should try to confirm that there are standard products available for use in the next stage.

### After Phase 1 ends

At least try to have:

- `file_inventory.json`
- `tech_stack.json`
- `entrypoints.json`

Optional enhancements:

- `raw_endpoints.json`
- `raw_secrets.json`
- `raw_native_bridges.json`
- `raw_env_guards.json`
- `env_guard_report.json`
- `frida_bypass_plan.json`
- `frida/android_phase1_bypass.js`

### After Phase 2 ends

At least try to have:

- `api_endpoints.json`
- `protocol_map.json`
- `traffic_alignment.json`

And give priority to completing the following fields for direct consumption by Phase 4:

- `field_role`
- `builder_path`
- `crypto_entry_candidate`
- `related_endpoint_group`
- `value_shape`
- `related_native_candidate`
- `replay_relevant`
- `matched_field_flows`

### After Phase 3 ends

At least try to have:

- `crypto_native_analysis.json`
- `jni_analysis.json`

And give priority to completing the following fields for direct consumption by Phase 4:

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

### After Phase 4 ends

At least try to have:

- `vuln_analysis.json`
- `risk_matrix.json`

And try to complete it in `vuln_analysis.json`:

- `crypto_findings`
- `signature_findings`
- `crypto_restoration`
- `packet_risks`
- `source_phase_2_fields`
- `source_phase_3_fields`
- `gap_filled_by_phase4`

### After Phase 5 ends

At least try to have:

- `validation_cases.json`
- `test_plan.md`
- `repro_steps.md`
- `poc_scripts_index.json`

If the materials are sufficient, you should also try to have:

- `pocs/{vuln_id}/validate_request.py`
- `pocs/{vuln_id}/runtime_observe.js`
- `pocs/{vuln_id}/README.md`

### After Phase 6 ends

At least try to have:

- `security_report.md`
- `findings.json`
- Attachments and full details accompanying the main report

## Standard conversation protocol

The following template is used for the standard input format that orchestrator should return to the user when "the user just says start step N".

The orchestrator must first return to the template before performing analysis.

Path convention:

- The sample path and output path provided to users can be the user's own real path
- When referencing repository files internally in Skill, always use relative paths with `ai-mobile-reverse-skills/` as the root directory.
- Absolute directories of any personal machine should not be written in Skill documents

### Phase 1 input template

When the user says "Start the first step", the orchestrator should return first:

```text
step: 1
analysis_mode: jadx_mcp_session/local_source
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
jadx_mcp: yes/no
```

Field description:
- `analysis_mode`: required, choose one from the two
- `target_dir`: required when `analysis_mode = local_source`, points to the decompiled source code or APK unpacking directory
- `output_dir`: required, the result output directory; if it does not exist, it should be created first
- `jadx_mcp`: required when `analysis_mode = jadx_mcp_session`, should be `yes`
- `apk_path`: optional, only used to supplement sample meta information

### Phase 2 input template

When the user says "Start step 2", the orchestrator should return first:

```text
step: 2
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
mcp: burp/yakit/none
traffic_source: analysis_runs/current_run/traffic/traffic.json
```

Field description:
- `target_dir`: required
- `output_dir`: required; if it does not exist, it should be created first
- `mcp`: optional, Burp MCP / Yakit MCP / none
- `traffic_source`: recommended when MCP is not used directly

### Phase 3 input template

When the user says "Start step three", the orchestrator should return first:

```text
step: 3
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
mcp: ida-mcp/ghidra-mcp/none
native_analysis_source: sample_target/native/sample.i64_or_sample.gpr_or_libsample.so
```

Field description:
- `target_dir`: recommended
- `output_dir`: required; if it does not exist, it should be created first
- `mcp`: optional, `ida-mcp` / `ghidra-mcp` / none, users can choose the analysis backend by themselves
- `native_analysis_source`: filled in when using native so/IDA/Ghidra material

### Phase 4 input template

When the user says "Start step 4", the orchestrator should return first:

```text
step: 4
output_dir: analysis_runs/current_run
allow_reanalyze_code: yes/no
```

Field description:
- `output_dir`: required; if it does not exist, it should be created first
- `allow_reanalyze_code`: optional, whether to allow Phase 4 lookback Java/smali/config

### Phase 5 input template

When the user says "Start step five", the orchestrator should return first:

```text
step: 5
output_dir: analysis_runs/current_run
authorized_only: yes/no
```

Field description:
- `output_dir`: required; if it does not exist, it should be created first
- `authorized_only`: It is recommended to fill in, the default is yes
- In Phase 5, in addition to the verification scheme by default, we should also try to generate the corresponding minimum POC script template for each vulnerability.

### Phase 6 input template

When the user says "Start Step 6", the orchestrator should return first:

```text
step: 6
output_dir: analysis_runs/current_run
target_name: project name
report_type: brief/full
include_appendix: yes/no
```

Field description:
- `output_dir`: required; if it does not exist, it should be created first
- `target_name`: It is recommended to fill in
- `report_type`: optional, short or full report
- `include_appendix`: optional, whether to include appendices and reproduction steps

### Usage 1: Start with the first step

User said:

`I am now connected to jadx-mcp and can use it. The target sample has been opened in Jadx and the output directory is analysis_runs/current_run. Please help me with the first step. `

General control should:

1. Identified as Phase 1
2. Check that `jadx_mcp = yes` and the target sample is open in Jadx
3. Route to `agents/agent-01-sample-recon.md`
4. Execute the file directly

### Usage 2: Enter the second step

User said:

`Proceed to step two. `

General control should:

1. Identified as Phase 2
2. Inherit by default the `{target_dir}` and `{output_dir}` confirmed in the first step
3. If the current mode is `step_by_step`, only Phase 2 will be executed and wait for manual confirmation.
4. If the current mode is `auto_chain + A/C`, it will continue to automatically advance to the subsequent stages after the conditions are met.
5. Route to `agents/agent-02-protocol-mapper.md`

### Usage 3: Start from step three

User said:

`I am now connected to ida-mcp and can use it. The IDA project is in sample_target/native/sample.i64. Please help me with the third step. `

or:

`I am now connected to ghidra-mcp and can use it. The Ghidra project is in sample_target/native/sample.gpr. Please help me with the third step. `

General control should:

1. Identified as Phase 3
2. Check so/JNI/IDA materials
3. Route to `agents/agent-03-crypto-native-analyzer.md`
4. Focus on JNI, algorithms, Native protection and restoration materials

### Usage 4: Directly enter the subsequent stage

User said:

`I have completed the previous steps and currently have vuln_analysis.json and risk_matrix.json, please help me with the fifth step. `

General control should:

1. Identified as Phase 5
2. Confirm that there are already vulnerability entries
3. Route to `agents/agent-05-validation-designer.md`
4. Only conduct minimum impact verification design and do not rewrite the previous analysis conclusions.

### Usage 5: Automatic chain A

The user first said:

```text
run_mode: auto_chain
auto_chain_mode: A
```

Then continue to provide the first step template. General control should:

1. Execute Phase 1 first
2. Wait for manual completion of packet capture preparation, MCP connection and other prerequisite actions
3. After entering Phase 2, it will automatically advance to Phase 6.
4. The confirmed path and context are inherited by default during the automatic advancement process.

### Usage 6: Automatic chain B / C

The user first said:

```text
run_mode: auto_chain
auto_chain_mode: B
```

or:

```text
run_mode: auto_chain
auto_chain_mode: C
```

Then continue to provide the first step template. General control should:

1. For `B`:
- Execute Phase 1-3 first
- Automatically advance from Phase 4 to Phase 6
2. For `C`:
- Start from Phase 1 and continue to advance to Phase 6 according to the automatic chain
3. Regardless of `B` or `C`, the first step of sample reconnaissance and testing confirmation must not be skipped.

## Relationship to other documents

- This document is responsible for general control, routing, pre-inspection, and stage promotion standards
- `README.md` is responsible for the complete user manual
- `docs/MCP-INTEGRATION.md` is responsible for the phased MCP access specification
- `agents/*.md` is responsible for the detailed execution method of each stage
- `tools/scripts/*.py` is responsible for local indexing, Native target convergence and Ghidra import assistance; the first 4 index scripts are executed by default under the `local_source` path
