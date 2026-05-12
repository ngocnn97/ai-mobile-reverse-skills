# MCP phase access specification

This document is not a general "instruction", but the MCP access specification for the current 6-stage main process.
It only answers 4 questions:

1. Which MCP should be connected first in each stage:
2. What are the minimum inputs required for each stage:
3. What do you hope to get from MCP at each stage:
4. How to cooperate with MCP and local scripts and stage documents

This specification is intended for users who already have MCP access conditions. It focuses on explaining "how to use MCP at the current stage" rather than explaining the installation steps of the MCP service itself.

## General principles

### Principle 1: MCP provides tool capabilities and does not replace stage conclusions.

The role of MCP is to bring tool context to AI, for example:

- Jadx class/method/string retrieval
- Burp / Yakit historical traffic
- IDA/Ghidra pseudocode and cross-reference

However, the analysis conclusions at each stage are still bound by the current stage document.

### Principle 2: AI-led analysis by default

if:

- MCP connected
- There are enough materials at the current stage
- Directory size is controllable

Then the priority is to directly let AI complete the stage analysis based on MCP.
It is not mandatory to run the local script first.

### Principle 3: Scripts are on-demand enhancements

It is recommended to run the script only under the following circumstances:

- The directory is huge
- Requires batch exhaustive hits
- Need to generate `raw_*.json` first
- Need to provide a stable structured input to subsequent stages

### Principle 4: Do not introduce new main stages

This specification strictly follows the current 6 phases and does not add additional main process concepts such as `Phase 0 / 1.5 / 2.5`.

### Principle 5: `output_dir` is created by the stage side

If the user has provided `{output_dir}` but the directory does not yet exist, the directory should be created at the current stage before writing the standard product.

## Stage access matrix

| Phase | Main Task | Recommended MCP | Required | Minimum Input | Recommended Output |
|---|---|---|---|---|---|
| Phase 1 | APK Static Reconnaissance | `jadx-mcp` or no MCP | No | `jadx_mcp=yes` or `{target_dir}` | `file_inventory.json` `tech_stack.json` `entrypoints.json` |
| Phase 2 | Traffic and code alignment | Burp MCP / Yakit MCP | Recommended to use | `{target_dir}` + packet capture MCP or `{traffic_source}` | `api_endpoints.json` `protocol_map.json` `traffic_alignment.json` |
| Phase 3 | In-depth analysis of SO and JNI | `ida-mcp` / `ghidra-mcp` | It is recommended to use | `{native_analysis_source}` or so / pseudocode material | `crypto_native_analysis.json` `jni_analysis.json` |
| Phase 4 | Risk and Vulnerability Screening | No MCP Enforcement | No | First Phase 1-3 Results | `vuln_analysis.json` `risk_matrix.json` `secrets_report.json` `jsbridge_analysis.json` |
| Phase 5 | Minimal Validation POC Design | Burp MCP / Yakit MCP Optional | No | Phase 4 Results | `validation_cases.json` `test_plan.md` `repro_steps.md` |
| Phase 6 | Penetration Report Summary | No MCP Enforcement | No | First Phase 1-5 Results | `security_report.md` `findings.json` |

## Phase 1: APK static reconnaissance access specification

### Target

Complete the first round of static portraits around the following tasks:

- Manifest, permissions, components
- Third-party SDK
- Hard coded information
- Environmental confrontation logic
- sign / token / encrypt / JNI / WebView entry

### Path A: Use `jadx-mcp` to analyze the currently opened sample of Jadx

Applicable scenarios:

- The target sample is already open in Jadx
- Want to analyze manifests, classes, methods and resources directly through Jadx views
- Need to quickly do class/method/keyword locating
- Need to quickly locate URL, encryption, request wrapper, native call

Recommended minimum input:

- `{output_dir}`
- `jadx_mcp = yes`

Additional instructions:

- In this mode, AI obtains the Manifest, class, method, string, resource and call clue of the currently opened sample of Jadx through `jadx-mcp`
- If `{output_dir}` does not exist, it should be created first and then write the result.

What to expect from MCP:

- Class name/method name/package name
- Keyword hits
- call chain fragment
- Suspicious strings and protocol paths

Recommended user trigger methods:

```text
I'm now connected to jadx-mcp and ready to use.
The target sample has been opened in Jadx
Help me with the first step.
```

### Path B: Directly analyze the local decompiled source code or APK unpacking directory

Applicable scenarios:

- Already obtained the unpacked and decompiled directory
- Or you have obtained the unpacked source code/resource directory of the APK
- Prefer to analyze directly using local directories and editors
- No need to rely on Jadx online context

Recommended method:

- VS Code global search
- Local text search
- When Phase 1 uses `local_source`, it is executed by default:
  - `endpoint_extractor.py`
  - `secret_scanner.py`
  - `native_bridge_indexer.py`
  - `env_guard_indexer.py`
- When Phase 1 runs `jadx_mcp_session`, the above 4 scripts are not executed by default

illustrate:

- Phase 1 is not mandatory to connect `jadx-mcp`
- "Shedding the shell" is not considered a responsibility at this stage

### Phase 1 output requirements

At least try to produce:

- `file_inventory.json`
- `tech_stack.json`
- `entrypoints.json`

Optional additions:

- `raw_endpoints.json`
- `raw_secrets.json`
- `raw_native_bridges.json`
- `raw_env_guards.json`

## Phase 2: Traffic and code alignment access specifications

### Target

In the packet capture data:

- URL / Path
- Header/Query/Body fields
- sign / token / data / timestamp / encryptData

Maps back to code:

- BaseURL
- Retrofit / OkHttp
- Request wrapper
- Parameter assembly logic
- Signature and encryption points

### Recommended MCP

- Burp MCP
- Yakit MCP

### Minimum input

- `{target_dir}`
- And meet one of the following:
- Burp MCP connected
- Yakit MCP connected
- Provide `{traffic_source}`

### What to expect from MCP

- Historical request list
- Request headers, parameters, response summary
- Login, payment, data, upload and other scene traffic
- Fields related to authentication and signature

### Recommended user trigger method

```text
I am now connected to the Burp MCP.
The packet capture results are in analysis_runs/current_run/traffic/traffic.json
The target directory is sample_target/decompiled
Help me with step two.
```

### Optional script enhancements

- `endpoint_extractor.py`
- Used to fill in static URL, Path, deeplink, provider clues
- `env_guard_indexer.py`
- Used to look back at agents, certificate verification, and packet capture detection clues

### Phase 2 output requirements

At least try to produce:

- `api_endpoints.json`
- `protocol_map.json`
- `traffic_alignment.json`

It is also recommended to complete the following field-level output for direct consumption in the subsequent comprehensive analysis stage:

- `field_role`
- `location`
- `builder_path`
- `crypto_entry_candidate`
- `related_endpoint_group`
- `value_shape`
- `related_native_candidate`
- `replay_relevant`
- `matched_field_flows`

## Phase 3: SO and JNI in-depth analysis access specifications

### Target

Do native digging around the following issues:

- Where is the JNI entrance:
- How to call native in Java layer
- Whether sign / encrypt / data / token related logic sinks to so
- What is the order of algorithm, Key, IV, Salt and parameters in so:
- Whether there is native layer counter logic

### Recommended MCP

- `ida-mcp`
- `ghidra-mcp`

### Minimum input

Just satisfy one of the following:

- Connected `ida-mcp`
- `ghidra-mcp` is connected
- Provide `{native_analysis_source}`
- Provides so/JNI pseudocode that can be directly analyzed

### What to expect from MCP

- JNI symbols and entries
- Cross-reference
- Pseudocode of key functions
- `RegisterNatives`, `System.loadLibrary` relationship
- Key strings and constants

### Recommended user trigger method

```text
I'm now connected to ida-mcp and ready to use.
The IDA project is in sample_target/native/sample.i64
Help me with step three.
```

or:

```text
I now have ghidra-mcp connected and ready to use.
The Ghidra project is in sample_target/native/sample.gpr
Help me with step three.
```

### Optional script enhancements

- `native_bridge_indexer.py`
- Used to supplement JNI, WebView, JSBridge, and bridge API clues
- It is only used as entry supplementary evidence and does not replace the so analysis of `ida-mcp` / `ghidra-mcp`

### Phase 3 output requirements

At least try to produce:

- `crypto_native_analysis.json`
- `jni_analysis.json`

It is also recommended to complete the following restoration fields for direct consumption in Phase 4:

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

## Phase 4: Weak encryption and high-risk vulnerability screening access specifications

### Target

Form risk conclusions based on the results of the first 1-3 steps, including:

- Weak encryption
- Hardcoded keys
- Authentication and authorization issues
- Data security issues
-Business logic issues
- Components and Deeplink Risks

### MCP Policy

- No mandatory MCP
- Callback when necessary:
  - `jadx-mcp`
  - `ida-mcp`
  - `ghidra-mcp`

### Minimum input

The first 1-3 stages result at least partially.

### Optional script enhancements

- `secret_scanner.py`
- `env_guard_indexer.py`

### Phase 4 output requirements

- `vuln_analysis.json`
- `risk_matrix.json`

It is recommended to further solidify the following structure in `vuln_analysis.json`:

- `crypto_findings`
- `signature_findings`
- `crypto_restoration`
- `packet_risks`
- `source_phase_2_fields`
- `source_phase_3_fields`
- `gap_filled_by_phase4`

## Phase 5: Minimum verification POC design access specification

### Target

Minimum impact verification design based on Phase 4 results, including:

- data encryption and decryption verification
- Signature bypass verification
- Verification of unauthorized access
- Parameter tampering verification
- Unauthorized access verification

### MCP Policy

- Burp MCP / Yakit MCP optional
- Mainly used to view historical requests, play back authorization test traffic and organize verification samples

### Minimum input

- `vuln_analysis.json` or `risk_matrix.json`

### Output requirements

- `validation_cases.json`
- `test_plan.md`
- `repro_steps.md`
- `poc_scripts_index.json`

If the materials are complete, try to generate:

- `pocs/{vuln_id}/validate_request.py`
- `pocs/{vuln_id}/runtime_observe.js`
- `pocs/{vuln_id}/README.md`

## Phase 6: Penetration report summary access specifications

### Target

Summarize steps 1-5 into deliverable reports.

### MCP Policy

- No mandatory MCP
- If you need to supplement evidence, you can call back the previous-stage MCP, but it does not change the locating of this stage as "summarization and delivery".

### Minimum input

The first 1-5 stages result in at least some of them.

### Output requirements

- `security_report.md`
- `findings.json`
- Comes with full range of accessories

## Optional supplementary MCP

### Chrome MCP / Playwright MCP

It is not a mandatory access item for the 6-stage main process, but can be used for:

- Automate the login process
- Token acquisition path observation
- H5 page behavior linkage
- Browser-side auxiliary verification

### UniDbg MCP

It is not required for the main process, but can be used for:

- Static native branch that cannot be restored
- Register/memory/breakpoint auxiliary judgment

## The relationship between MCP and script layer

The relationship between the two is not a substitution, but a division of labor:

- MCP: Provide tool context and interaction capabilities
- Stage documents: stipulates methodology, steps, and output requirements
- Local script: do high coverage batch hits and structured indexing

The corresponding relationship is as follows:

- `endpoint_extractor.py`
- Suitable for supplementing APIs and path clues for Phase 1 and Phase 2
- `secret_scanner.py`
- Suitable for supplementing sensitive information clues in Phase 1 and Phase 4
- `native_bridge_indexer.py`
- Suitable for supplementing JNI/bridge clues for Phase 1 and Phase 3
- `env_guard_indexer.py`
- Suitable for supplementing Phase 1 environmental confrontation clues

## Recommended minimum closed loop

If you want to make a minimum usable version first, the suggestions are as follows:

1. Phase 1: `jadx-mcp` or local directory analysis
2. Phase 2:Burp MCP / Yakit MCP
3. Phase 3:`ida-mcp` / `ghidra-mcp`
4. Phase 4-6: Complete risk judgment, POC design and report delivery based on previous products

Among them, in addition to the main output, Phase 4 should also try to organize `raw_secrets.json` and WebView / JSBridge clues into `secrets_report.json` and `jsbridge_analysis.json` for direct consumption by Phase 6.

If the token is tight and the directory is too large, add a local script index.

## Current unified context variables

```text
{target_name} Target App name
{apk_path} APK file path, optional, only used to supplement sample meta information
{target_dir} Directory after unpacking and decompilation
{traffic_source} Burp / Yakit export results or local capture files
{native_analysis_source} IDA / Ghidra project, so sample or pseudocode material
{output_dir} output directory
```
