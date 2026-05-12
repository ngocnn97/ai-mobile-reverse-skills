# POC Templates

This directory provides Phase 5 reusable minimal POC script templates.

Design principles:
- Only for authorized testing environments
- By default, placeholder URL, placeholder token, and placeholder business parameters are used
- Does not embed real production targets or real sensitive values
- Aim to “verify existence” rather than expand influence

Recommended usage:
- `python_http_validation.py.tmpl`
- Minimal verification based on HTTP requests such as unauthorized access, unauthorized access, parameter tampering, signature bypass, etc.
- `frida_runtime_observe.js.tmpl`
- Used for observation scripts that still need to confirm the order of input, output, Key, IV, and parameters while running
- `CASE_README.md.tmpl`
- Used to supplement each vulnerability catalog with running instructions, stop loss points and manual patching items

Recommended output directory:
- `pocs/{vuln_id}/validate_request.py`
- `pocs/{vuln_id}/runtime_observe.js`
- `pocs/{vuln_id}/README.md`
