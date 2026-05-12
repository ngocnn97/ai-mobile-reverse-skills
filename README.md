# AI Mobile Reverse Skills

A 6-stage orchestrator skill for mobile security analysis scenarios. Used for unified scheduling of APK static reconnaissance, traffic and code alignment, SO/JNI in-depth analysis, encryption and vulnerability comprehensive analysis, verification design and report delivery process. Supports JADX MCP, Burp/Yakit MCP, IDA/Ghidra MCP.

## 1. Applicable scenarios

- Android APK static reverse engineering and security portrait
- Linkage analysis between decompiled code, packet capture results, and API fields
- JNI / SO / native encryption, signature, risk-control logic locating
- Close out weak encryption, authentication and authorization, component security, JSBridge, sensitive information, and related risks
- Minimum verification scheme and POC template design in authorized test environment
- Mobile penetration testing reports and structured Findings delivery

## 2. Architecture design
It is mainly composed of the following core modules:
- Root orchestrator SKILL.md: As the dispatch center of the entire process, it is responsible for intent interception, standard input template return, task routing distribution, and stage execution rule constraints.
- 6-stage Agent: A rule set customized for the entire life cycle of mobile security analysis, covering the first stage of APK static reconnaissance to the sixth stage of security report summary.
- MCP phased access specification: clarifies the access and calling standards of Jadx-MCP (static analysis), Burp/Yakit-MCP (analysis) and Ghidra/IDA-MCP (native deep analysis) at different stages.
- Local index script (Python probe): A tool set responsible for high-coverage blind scanning, including API extraction (endpoint_extractor.py), hard-coded scanning (secret_scanner.py), JNI bridge indexing, and target SO automatic resolution and loading tools.
- Unified structured output design: Through standardized JSON and Markdown products, it is ensured that the analysis leads in the previous stages can be automatically inherited and deeply linked by subsequent agents.
```text
ai-mobile-reverse-skills/
├── SKILL.md # Orchestrator entry point: stage routing, input template, execution rules
├── README.md # User manual: process description, complete examples, interactive methods
├── agents/ # Six-stage Agent rule set
│ ├── agent-01-sample-recon.md # Phase 1: APK static reconnaissance
│ ├── agent-02-protocol-mapper.md # Phase 2: Traffic and code alignment
│ ├── agent-03-crypto-native-analyzer.md # Phase 3: SO/JNI in-depth analysis
│ ├── agent-04-crypto-vuln-analyzer.md # Phase 4: Weak encryption and high-risk vulnerability screening
│ ├── agent-05-validation-designer.md # Phase 5: Minimum verification POC design
│ └── agent-06-reporter.md # Phase 6: Security report summary
├── docs/ # Stage access and supplementary documentation
│ └── MCP-INTEGRATION.md # MCP phased access specification
├── templates/ # Report and reproduction templates
│ ├── mobile-reverse-report-template.md # Mobile security report template
│ └── repro-steps-template.md # Repro steps template
├── tools/ # Supporting tools and template resources
│ ├── frida/ # Frida related templates
│ │ ├── README.md # Frida template description
│ │ └── android_phase1_bypass.js # Phase 1 runtime preparation/observation template
│ ├── poc_templates/ # POC / verification template
│ │ ├── README.md # POC template description
│ │ ├── CASE_README.md.tmpl # Single vulnerability verification description template
│ │ ├── frida_runtime_observe.js.tmpl # Frida runtime observation template
│ │ └── python_http_validation.py.tmpl # HTTP validation script template
│ └── scripts/ # Local index script
│ ├── README.md # Script description and sample schema
│ ├── endpoint_extractor.py # Interface/URL/field clue extraction
│ ├── env_guard_indexer.py # Root / Proxy / Frida / SSL Pinning clue extraction
│ ├── native_bridge_indexer.py # JNI / JSBridge / native crypto clue extraction
│ ├── secret_scanner.py # Hardcoded key / Token / Certificate / Cloud Credential Scan
│ ├── resolve_native_target.py # Resolve the priority SO target for Phase 3 analysis
│ └── ghidra_target_loader.py # Automatically import the target SO into the Ghidra project
```

![Insert picture description here](https://i-blog.csdnimg.cn/direct/7c754161e2d2472c88d6cf4a08d196d6.png#pic_center)


## 3. How to use this repository

The core entrance to this repository is:

- `ai-mobile-reverse-skills/SKILL.md`

When used as a Skill package, place `ai-mobile-reverse-skills/` in the Codex / AI Skill search directory that supports `SKILL.md`, or let Codex directly read `SKILL.md` in this directory in the current workspace. The stage documents, scripts and templates inside the repository are referenced with `ai-mobile-reverse-skills/` as the relative root directory.

If you just want to understand the process, read `ai-mobile-reverse-skills/USER-README.md` first; if you want to execute the complete stage rules, refer to the 6 Agent documents under `ai-mobile-reverse-skills/SKILL.md` and `agents/`.

## 4. Stage process description

| Phase | Agent | Goal | Main Output |
|---|---|---|---|
| Phase 1 | SampleRecon | APK static reconnaissance, technology stack identification, environment detection, sensitive entrance and SO clue preliminary screening | `file_inventory.json`, `tech_stack.json`, `entrypoints.json`, `env_guard_report.json` |
| Phase 2 | ProtocolMapper | Align packet capture requests, API fields, signature parameters and code implementations | `api_endpoints.json`, `protocol_map.json`, `traffic_alignment.json` |
| Phase 3 | CryptoNativeAnalyzer | Analysis of JNI/SO/native encryption and signing logic around Phase 2 clues | `crypto_native_analysis.json`, `jni_analysis.json` |
| Phase 4 | CryptoVulnAnalyzer | Consolidate previous-stage evidence into weak-encryption and high-risk vulnerability findings | `vuln_analysis.json`, `risk_matrix.json`, `secrets_report.json`, `jsbridge_analysis.json` |
| Phase 5 | ValidationDesigner | Design minimal validation scenarios and POC templates in an authorized environment | `validation_cases.json`, `test_plan.md`, `repro_steps.md` |
| Phase 6 | Reporter | Summarize Phase 1-5, generate delivery reports and Findings | `security_report.md`, `findings.json` |

All modes start with Phase 1. Automatic chaining also does not skip the first stage.
![Insert image description here](https://i-blog.csdnimg.cn/direct/625a635a779845ea8461726b75abae9e.png)

## 5. MCP access instructions

MCP is the tool-context entry point and does not replace stage judgment. This skill uses the following MCPs.

| MCP | Main uses | Typical stages |
|---|---|---|
| `jadx-mcp` | Read the classes, methods, resources, strings and call references of the current Jadx sample | Phase 1, Phase 4 |
| Burp MCP / Yakit MCP | Read packet capture request, header, body, response summary and API context | Phase 2, Phase 5 |
| `ida-mcp` / `ghidra-mcp` | Analysis of SO, JNI, pseudocode, cross-references and native encryption logic | Phase 3 |

![Insert image description here](https://i-blog.csdnimg.cn/direct/2d222bef41924711b782fbac97bd80cf.png)

For complete specifications see:

- `ai-mobile-reverse-skills/docs/MCP-INTEGRATION.md`
## 6. Operation mode

### 5.1. Progress step by step

It is suitable for scenarios where each step requires manual review and the focus of analysis can be adjusted at any time.

```text
run_mode: step_by_step
```

Features:

- Suspend by default after each stage
- Manually confirm the results of the current stage before entering the next stage
- Suitable for projects with complex samples, unstable preconditions for packet capture, and projects that require step-by-step judgment

### 5.2. Automatic chain

It is suitable for scenarios where the prerequisite materials are relatively complete and the system is expected to be continuously advanced to the report as much as possible.

```text
run_mode: auto_chain
auto_chain_mode: A/B/C
```

| Mode | Automation scope | Suitable situation |
|---|---|---|
| A | Phase 1 manual confirmation, Phase 2-6 automatic advancement | After the first phase, preparations such as proxy, packet capture, MCP connection, etc. need to be completed manually |
| B | Phase 1-3 manual confirmation, Phase 4-6 automatic advancement | Manual digging in the front, vulnerability closing, verification and reporting automation in the back |
| C | Phase 1-6 should be promoted automatically as much as possible | The decompilation directory, packet capture, MCP and native analysis materials have been prepared before starting |

The automatic chain pauses at the earliest blocking stage when a key condition is missing, such as packet capture results, `ghidra_root`, or previous-stage artifacts.

## 6. Quick start

It is recommended to start according to the "two-stage" method: first select the mode, and then enter the first stage.

### 6.1. Select mode

Manual step-by-step execution:

```text
run_mode: step_by_step
```

Or choose automatic chaining:

```text
run_mode: auto_chain
auto_chain_mode: B
```

### 6.2. Provide first stage input

Analyze the local decompilation directory:

```text
step: 1
analysis_mode: local_source
target_dir: sample_target/decompiled
output_dir: analysis_runs/current_run
jadx_mcp: no
```

Using the current Jadx MCP session:

```text
step: 1
analysis_mode: jadx_mcp_session
output_dir: analysis_runs/current_run
jadx_mcp: yes
```

Field description:

- `analysis_mode`: `local_source` means analyzing the local decompilation/unpacking directory, `jadx_mcp_session` means using the opened Jadx MCP session
- `target_dir`: the main analysis directory after decompilation; it can be left blank when using Jadx MCP
- `output_dir`: unified output directory, inherited by default in subsequent stages
- `jadx_mcp`: Whether `jadx-mcp` is currently connected
 
## 7. Update instructions

### 2026/4/23
First edition released



