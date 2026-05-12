# Scripts Layer

This directory places local script tools that serve the main process of the current six phases and does not constitute a new phase separately.

The responsibilities of these scripts and accompanying templates are:
- Batch scan of decompiled directories
- Generate structured raw hit results
- Provide auxiliary input for Phase 1, Phase 2, Phase 3, Phase 4
- Optional reading of `file_inventory.json`, giving priority to scanning by asset inventory

Current default rules:
- When Phase 1 uses `local_source`, the first 4 index scripts are executed by default
- When Phase 1 uses `jadx_mcp_session`, the first 4 index scripts are not executed by default
- `resolve_native_target.py` and `ghidra_target_loader.py` maintain the original automation logic

The Python script is only responsible for "scanning and indexing" and does not directly output the final vulnerability conclusion.
Frida templates are responsible for providing basic assets for runtime bypass in authorized environments and do not replace targeted analysis of specific targets.
For Phase 3, the script layer is only responsible for filling in JNI / bridge / loadLibrary clues; so you should use `ida-mcp` or `ghidra-mcp` for reverse analysis of the main path.

## Current script

### 1. endpoint_extractor.py

effect:
- Extract URL, Path, BaseURL, Retrofit annotation, request wrapper, WebSocket, upload and download related clues
- Extract traces of field names such as `sign`, `data`, `encryptData`, `timestamp`, `nonce`, `salt`, `iv`, `hmac` and other fields at the same time
- Also mark cryptographic wrapper clues such as `Cipher`, `MessageDigest`, `Mac`, `Signature`, `CryptoJS`, `JSEncrypt`, `SM2/3/4` etc.

Main service stages:
- Phase 1: APK static reconnaissance
- Phase 2: Traffic and code alignment

Output:
- `raw_endpoints.json`

### 2. secret_scanner.py

effect:
- Extract hardcoded keys, API Keys, Tokens, certificate materials, test environment URLs, intranet IPs, and debugging marks
- Supplementary identification of encryption materials such as HMAC key, AES/DES/SM4 key, IV, Salt, nonce, AndroidKeyStore alias, public and private key named variables, etc.

Main service stages:
- Phase 1: APK static reconnaissance
- Phase 4: Weak encryption and high-risk vulnerability screening

Output:
- `raw_secrets.json`

### 3. native_bridge_indexer.py

effect:
- Extract `System.loadLibrary`
- Extract `native` method declaration
- Extract JNI symbols, `RegisterNatives`
- Extract `addJavascriptInterface`, `evaluateJavascript`, `loadUrl("javascript:")`
- Additional identification of OpenSSL/EVP/AES/RSA/HMAC/SHA/MD5/SM2/SM3/SM4 etc. native encryption symbols and library traces

Main service stages:
- Phase 1: APK static reconnaissance
- Phase 3: Supplementary clues for in-depth analysis of SO and JNI

Output:
- `raw_native_bridges.json`

### 4. env_guard_indexer.py

effect:
- Extract environmental confrontation clues such as Root detection, emulator detection, proxy/VPN detection, SSL Pinning, Frida/debugging detection, signature verification, multi-open detection, etc.

Main service stages:
- Phase 1: APK static reconnaissance
- Phase 2: Traffic and code alignment (pre-preparation reference)

Output:
- `raw_env_guards.json`
- `env_guard_report.json`
- `frida_bypass_plan.json`
- `frida/android_phase1_bypass.js`

### 5. ghidra_target_loader.py

effect:
- Locate the target .so based on `lib/<abi>/*.so` and `selected_native_target.json` in the APK unpacking source directory
- Automatically parse the so that should be analyzed first
- Import the so into the specified Ghidra project through `analyzeHeadless`
- Try to pull up or switch to Ghidra GUI and open the corresponding project
- Generate `ghidra_loader_result.json` for further consumption by Phase 3

Main service stages:
- Phase 3: In-depth analysis of SO and JNI

illustrate:
- This script is responsible for "finding the target .so + importing the Ghidra project + trying to pull up the GUI"
- You should confirm `ghidra_root` before using it for the first time. This is the root directory of the local Ghidra installation. It is not recommended to rely on automatic guessing.
- It does not guarantee that the Ghidra GUI will automatically switch focus to the corresponding program tab
- Final focus behavior of the GUI depends on the native Ghidra version, operating system and current workspace state

Output:
- `ghidra_loader_result.json`

### 6. resolve_native_target.py

effect:
- Read Phase 1 / Phase 2 products
- Automatically converge the current .so files that are most worthy of entering Phase 3 analysis
- Generate candidate list and final selection results
- Provide stable input to `ghidra_target_loader.py`

Main service stages:
- Phase 2: Native target convergence after traffic and code alignment
- Phase 3: Preparation for in-depth analysis of SO and JNI

Output:
- `native_target_candidates.json`
- `selected_native_target.json`

## Matching Frida template

Table of contents:

- `../frida/android_phase1_bypass.js`

effect:

- Provide minimal adaptable bypass templates for Root/Emulator/Proxy/SSL Pinning detection identified in Phase 1
- Provide basic hook assets for phase 2 packet capture pre-preparation

illustrate:

- The template is oriented to the authorization environment by default
- `raw_env_guards.json` and `entrypoints.json` should be combined for tailoring instead of copying without distinction.

## Unified usage

By default, the unified output root directory is passed in, and the script will write to the corresponding stage directory by itself:

- Phase 1 script is written to `{output_dir}/step1/`
- `resolve_native_target.py` is written to `{output_dir}/step2/`
- `ghidra_target_loader.py` writes to `{output_dir}/step3/`
- Reading the old version of `{output_dir}/<artifact>` is only used as a compatibility guide when tiling the results.

Example:

```bash
python3 endpoint_extractor.py --target-dir sample_target/decompiled --output-dir analysis_runs/current_run
python3 secret_scanner.py --target-dir sample_target/decompiled --output-dir analysis_runs/current_run
python3 native_bridge_indexer.py --target-dir sample_target/decompiled --output-dir analysis_runs/current_run
python3 env_guard_indexer.py --target-dir sample_target/decompiled --output-dir analysis_runs/current_run
python3 resolve_native_target.py --output-dir analysis_runs/current_run --target-dir sample_target/decompiled
python3 ghidra_target_loader.py --output-dir analysis_runs/current_run --target-dir sample_target/apk_unpacked --project-dir analysis_runs/current_run/ghidra_projects --project-name sample_project --ghidra-root sample_tools/Ghidra/ghidra_x.y.z_PUBLIC
```

If you already have `file_inventory.json`, you can also use it like this:

```bash
python3 endpoint_extractor.py --target-dir sample_target/decompiled --output-dir analysis_runs/current_run --inventory analysis_runs/current_run/step1/file_inventory.json
python3 secret_scanner.py --target-dir sample_target/decompiled --output-dir analysis_runs/current_run --inventory analysis_runs/current_run/step1/file_inventory.json
python3 native_bridge_indexer.py --target-dir sample_target/decompiled --output-dir analysis_runs/current_run --inventory analysis_runs/current_run/step1/file_inventory.json
python3 env_guard_indexer.py --target-dir sample_target/decompiled --output-dir analysis_runs/current_run --inventory analysis_runs/current_run/step1/file_inventory.json
```

## Design principles

- Purely local execution
- Prioritize the use of standard libraries
- Ensure coverage first, then hand it over to Agent for semantic analysis
- Output unified JSON to facilitate consumption in subsequent stages

## Sample Output Schema

The following structure is not a complete field table, but the minimum common fields that should be relied upon for subsequent Agent consumption.

### raw_endpoints.json

```json
{
  "scan_meta": {
    "tool": "endpoint_extractor.py",
    "target_dir": "sample_target/decompiled",
    "file_source": "inventory|walk"
  },
  "type_statistics": {
    "full_url": 10,
    "retrofit_annotation": 6,
    "crypto_field_key": 4,
    "crypto_wrapper_hint": 3
  },
  "base_url_candidates": [
    {
      "value": "https://api.example.com",
      "source_file": "smali/.../Api.smali",
      "source_line": 42,
      "reason": "base_url"
    }
  ],
  "by_file": {
    "smali/.../Api.smali": {
      "hit_count": 3,
      "hits": [
        {
          "type": "retrofit_annotation",
          "value": "POST /auth/login",
          "line": 42,
          "context": "..."
        },
        {
          "type": "crypto_field_key",
          "value": "sign",
          "line": 57,
          "context": "..."
        }
      ]
    }
  },
  "all_hits": []
}
```

### raw_secrets.json

```json
{
  "scan_meta": {
    "tool": "secret_scanner.py",
    "target_dir": "sample_target/decompiled",
    "scan_rules_count": 40
  },
  "severity_statistics": {
    "Critical": 3,
    "High": 8
  },
  "category_statistics": {
    "secret": 2,
    "cloud_key": 1,
    "crypto_material": 3
  },
  "by_file": {
    "assets/config.json": {
      "hit_count": 2,
      "hits": [
        {
          "category": "crypto_material",
          "sub_type": "hmac_key_assignment",
          "severity": "Critical",
          "value": "abcd1234...",
          "masked_value": "abcd...1234",
          "line": 10,
          "confidence": "high",
          "is_placeholder": false,
          "context": "..."
        }
      ]
    }
  },
  "all_hits": []
}
```

### raw_native_bridges.json

```json
{
  "scan_meta": {
    "tool": "native_bridge_indexer.py",
    "target_dir": "sample_target/decompiled"
  },
  "type_statistics": {
    "load_library": 2,
    "native_method": 5,
    "native_crypto_symbol": 4,
    "java_crypto_bridge": 3
  },
  "libraries": [
    {
      "library": "native-lib",
      "occurrences": [
        {
          "source_file": "smali/.../MainActivity.smali",
          "source_line": 18,
          "type": "load_library"
        }
      ]
    }
  ],
  "by_file": {
    "smali/.../Bridge.smali": {
      "hit_count": 4,
      "hits": [
        {
          "type": "add_javascript_API",
          "value": "appBridge",
          "line": 66,
          "context": "..."
        },
        {
          "type": "native_crypto_symbol",
          "value": "EVP_EncryptInit_ex(ctx, ...)",
          "line": 91,
          "context": "..."
        }
      ]
    }
  },
  "all_hits": []
}
```

### raw_env_guards.json

```json
{
  "scan_meta": {
    "tool": "env_guard_indexer.py",
    "target_dir": "sample_target/decompiled"
  },
  "guard_statistics": {
    "root_detection": 4,
    "ssl_pinning_or_cert_check": 3
  },
  "by_file": {
    "smali/.../SecurityCheck.smali": {
      "hit_count": 3,
      "hits": [
        {
          "guard_type": "root_detection",
          "match": "RootBeer",
          "line": 21,
          "pattern": "RootBeer",
          "bypass_hint": "...",
          "context": "..."
        }
      ]
    }
  },
  "all_hits": []
}
```

## Agent Consumption Notes

- Phase 1 priority consumption:
  - `raw_endpoints.json`
  - `raw_secrets.json`
  - `raw_native_bridges.json`
  - `raw_env_guards.json`
- Phase 2 key consumption:
  - `raw_endpoints.json`
  - `raw_env_guards.json`
- Phase 3 key consumption:
  - `raw_native_bridges.json`
- Phase 4 key consumption:
  - `raw_secrets.json`
  - `raw_endpoints.json`
  - `raw_env_guards.json`
- `raw_native_bridges.json` (when sign/data may sink to native)

When Agent reads these files, priority is given to:

- `scan_meta`
- `type_statistics / category_statistics / guard_statistics`
- `base_url_candidates / libraries`
- `by_file`
- `all_hits`
