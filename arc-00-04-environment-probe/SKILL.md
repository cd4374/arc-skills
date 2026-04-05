---
name: arc-00-04-environment-probe
description: Stage 0 — Detect ALL execution environment capabilities BEFORE any pipeline execution. MUST verify: execution backend (local GPU/local CPU/SSH remote), LOCAL LaTeX compilation, and network connectivity for literature verification. BLOCKING on any missing critical component.
metadata:
  category: orchestrator
  trigger-keywords: "environment probe,detect gpu,execution capability,latex,stage 0"
  applicable-stages: "0"
  priority: "0"
  version: "3.1"
  author: researchclaw
---

## Purpose

Detect ALL execution environment capabilities BEFORE any pipeline execution. This stage verifies:

1. **Execution Backend** — Required for Stages 10-13 (experiment execution)
2. **Local LaTeX Compilation** — Required for Stages 22, 28 (pdflatex + bibtex MUST be installed locally)
3. **Network Connectivity** — Required for Stages 3-4 (literature search and DOI verification)

**This stage is CRITICAL and BLOCKING** — any missing component blocks the entire pipeline.

**No LaTeX fallbacks**: LaTeX must compile locally. Execution may use a local backend or a reachable SSH remote backend, but no cloud/Overleaf LaTeX alternative is allowed.

---

## Quality Contract

`environment.json` MUST accurately reflect ALL required capabilities:

| Capability | Required By | Blocking? |
|------------|-------------|-----------|
| Execution backend (GPU/CPU/SSH) | Stages 10-13 | **YES** — no fake experiments |
| Local LaTeX (pdflatex + bibtex) | Stages 22, 28 | **YES** — no fake PDF |
| Network (doi.org/arxiv.org) | Stages 3-4 | **YES** — no fake citations |

And MUST satisfy:
- `pipeline_capable: true` only if ALL critical components are available
- `execution_capable: true` → at least one execution backend available
- `latex_capable: true` → pdflatex and bibtex can compile documents locally
- `network_capable: true` → can reach doi.org and arxiv.org

If ANY critical capability is missing:
- `environment.json` must have `pipeline_capable: false`
- `environment_error.md` must explain how to fix
- Pipeline MUST NOT proceed past Stage 0

---

## Inputs

Configuration from (in priority order):
1. **User explicit config** (`config.arc.yaml`)
2. **Auto-detected environment** (probed)

### `config.arc.yaml` Schema (Optional)
```yaml
execution:
  # Explicit backend selection (optional - if set, skips auto-detection)
  backend: "local_gpu" | "local_cpu" | "ssh_remote"

  # Conda environment configuration
  conda:
    env_name: "arc-research"  # preferred env name
    python_version: "3.10"    # Python version for new env
    auto_create: true         # create env if not exists

  # SSH remote configuration (only used if backend = "ssh_remote")
  ssh_remote:
    host: "gpu-server.example.com"
    user: "username"
    key_path: "~/.ssh/id_rsa"
    gpu_ids: [0, 1]
    conda_env: "arc-research"

  # Working directory
  working_dir: "."  # default: current directory
```

**Note**: LaTeX must be installed locally. Remote or cloud execution does not satisfy the Stage-22/28 compile requirement.

---

## Outputs

### `environment.json`
```json
{
  "pipeline_capable": true,
  "execution_capable": true,
  "latex_capable": true,
  "network_capable": true,
  "user_configured": false,
  "backend": {
    "type": "local_gpu",
    "priority": 1,
    "source": "auto_detected"
  },
  "gpu": {
    "available": true,
    "type": "NVIDIA A100",
    "memory_gb": 80,
    "count": 1,
    "backend": "cuda"
  },
  "compute": {
    "device": "cuda:0",
    "cpu_cores": 8,
    "cpu_available": true
  },
  "conda": {
    "available": true,
    "env_name": "arc-research",
    "env_path": "/home/user/miniconda3/envs/arc-research",
    "python_version": "3.10.12",
    "is_base": false,
    "packages": {
      "torch": "2.1.0",
      "numpy": "1.24.3"
    }
  },
  "latex": {
    "available": true,
    "pdflatex_version": "3.141592653-2.6-1.40.25",
    "bibtex_version": "0.99d",
    "texlive_version": "2023"
  },
  "network": {
    "doi_org_reachable": true,
    "arxiv_org_reachable": true,
    "openalex_api_reachable": true,
    "semantic_scholar_api_reachable": true,
    "last_check_timestamp": "2026-04-05T12:00:00Z"
  },
  "ssh_remote": {
    "configured": false,
    "reachable": false
  },
  "working_dir": "/Users/abc/Desktop/00-project/arc",
  "probe_timestamp": "2026-04-05T12:00:00Z",
  "blocking_issues": []
}
```

### `environment_error.md` (only if pipeline_capable: false)
```markdown
# Environment Error: Critical Capabilities Missing

## Summary
Pipeline cannot proceed because the following critical capabilities are missing:

| Capability | Status | Required For |
|------------|--------|--------------|
| Execution Backend | ❌ MISSING | Stages 10-13 (experiments) |
| Local LaTeX | ❌ MISSING | Stages 22, 28 (paper export) |
| Network Connectivity | ❌ MISSING | Stages 3-4 (literature verification) |

## How to Fix

### Install LaTeX (REQUIRED)
```bash
# macOS
brew install --cask mactex-no-gui

# Ubuntu/Debian
sudo apt-get install texlive-full texlive-science texlive-publishers

# Verify
pdflatex --version
bibtex --version
```

### Configure Execution Environment (REQUIRED for experiments)
```bash
conda create -n arc-research python=3.10 -y
conda activate arc-research
conda install pytorch torchvision torchaudio -c pytorch
```

### Check Network Connectivity (REQUIRED)
```bash
curl -I https://doi.org/10.1000/182
curl -I https://arxiv.org/abs/2301.00001
```

## Pipeline Status
Pipeline CANNOT proceed without ALL critical capabilities.
**No bypass path is available** — install or restore the missing critical components before re-running.
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| `environment.json` created | File exists and is valid JSON |
| `pipeline_capable: true` | ALL critical capabilities available |
| Execution backend available | At least one execution backend if `execution_capable` |
| GPU info matches reality | If `gpu.available: true`, verified via nvidia-smi or torch.MPS |
| Conda configured | `conda.available: true` and `is_base: false` |
| **LaTeX available** | `latex.available: true` AND pdflatex/bibtex versions recorded |
| **Network reachable** | `doi.org` and `arxiv.org` both reachable |
| Working dir valid | `working_dir` exists and is writable |

**BLOCKING Failure**: If ANY critical capability missing → `E00_CRITICAL_MISSING` (non-retryable)

---

## Error Codes

| Code | Trigger | Retryable |
|------|---------|-----------|
| `E00_CRITICAL_MISSING` | Any critical capability (execution/LaTeX/network) missing | **No** (blocking) |
| `E00_NO_EXECUTION_ENV` | No execution backend available | No |
| `E00_NO_LATEX` | pdflatex or bibtex not found | No |
| `E00_NETWORK_UNREACHABLE` | doi.org or arxiv.org unreachable | Yes (1 retry) |
| `E00_PROBE_FAILED` | Probe commands failed unexpectedly | Yes (1 retry) |
| `E00_CONDA_NOT_FOUND` | Conda not installed | No |
| `E00_CONDA_ENV_FAILED` | Failed to create/activate conda environment | Yes (1 retry) |
| `E00_SSH_UNREACHABLE` | SSH remote configured but unreachable | No |

---

## Retry Policy
Max retries: **1** — only on transient network/probe failures.

**DO NOT retry** on:
- `E00_CRITICAL_MISSING` — missing critical component
- `E00_NO_EXECUTION_ENV` — no execution backend
- `E00_NO_LATEX` — no LaTeX installation

These are **blocking** failures that require user intervention.

---

## State Transition
`pending` → `running` → `done` | `blocked`

---

## Procedure

### Step 1 — Read User Configuration

Read `config.arc.yaml` if exists:
- If `execution.backend` is explicitly set → use it (skip auto-detection)
- If `execution.conda.env_name` is set → use as preferred env name
- If `execution.working_dir` is set → use it; else default to current directory (`.`)
- Ignore any legacy LaTeX backend settings — local LaTeX is the only supported path

Set `user_configured: true` if any explicit backend is specified.

### Step 2 — Check Network Connectivity (BLOCKING)

**This check runs first because literature verification is fundamental.**

```bash
# Test DOI resolution (required for Stage 4 literature verification)
curl -s -o /dev/null -w "%{http_code}" -L --connect-timeout 5 "https://doi.org/10.1000/182"

# Test arXiv access (required for Stage 4 arXiv verification)
curl -s -o /dev/null -w "%{http_code}" --connect-timeout 5 "https://arxiv.org/abs/2301.00001"

# Test OpenAlex API (optional but recommended)
curl -s -o /dev/null -w "%{http_code}" --connect-timeout 5 "https://api.openalex.org/works?per-page=1"

# Test Semantic Scholar API (optional)
curl -s -o /dev/null -w "%{http_code}" --connect-timeout 5 "https://api.semanticscholar.org/graph/v1/paper/search?query=test&limit=1"
```

Record:
- `network.doi_org_reachable`: true if HTTP 200/303
- `network.arxiv_org_reachable`: true if HTTP 200
- `network.openalex_api_reachable`: true if HTTP 200
- `network.semantic_scholar_api_reachable`: true if HTTP 200

**If doi.org OR arxiv.org unreachable**:
- Set `network_capable: false`
- Set `pipeline_capable: false`
- Add to `blocking_issues`: "Network unreachable - cannot verify literature DOI/arXiv"
- Fail with `E00_NETWORK_UNREACHABLE` (retry once, then block with `E00_CRITICAL_MISSING`)

### Step 3 — Check LaTeX Installation (BLOCKING)

**LaTeX is REQUIRED for Stages 22 and 28. Must be installed locally — no fallbacks.**

```bash
# Check pdflatex
pdflatex --version 2>/dev/null | head -1

# Check bibtex
bibtex --version 2>/dev/null
```

| Check | Pass Condition |
|-------|---------------|
| `pdflatex --version` exits 0 | pdflatex installed locally |
| `bibtex --version` exits 0 | bibtex installed locally |

Record:
- `latex.available`: true if both pdflatex AND bibtex available
- `latex.pdflatex_version`: version string
- `latex.bibtex_version`: version string

**If LaTeX NOT installed locally**:
- Set `latex_capable: false`
- Set `pipeline_capable: false`
- Add to `blocking_issues`: "LaTeX not installed locally — install pdflatex and bibtex"
- Fail with `E00_NO_LATEX` (blocking, non-retryable)

### Step 4 — Check Conda Installation

```bash
conda --version 2>/dev/null
which conda 2>/dev/null
```

- If exit code 0: Conda available, proceed to Step 3
- If exit code non-zero: **Fail with `E00_CONDA_NOT_FOUND`** — write instructions for installing Miniconda

### Step 5 — Detect or Create Conda Environment

**A. List existing environments:**
```bash
conda env list --json 2>/dev/null
```

**B. Check for suitable environments (priorities):**
1. User-specified `env_name` from config
2. Environments with PyTorch installed: `conda list -n {env} torch 2>/dev/null`
3. Non-base environments with numpy/scipy

**C. Environment selection criteria:**
- **NEVER use base environment** (`is_base: false` is required)
- Prefer environments with PyTorch already installed
- Prefer Python 3.9-3.11

**D. If no suitable environment exists and `auto_create: true`:**
```bash
# Create new environment
conda create -n {env_name} python={python_version} -y

# Activate and install core packages
conda activate {env_name}
conda install numpy scipy matplotlib pandas -y

# Try to install PyTorch (auto-detect CPU/GPU)
if nvidia-smi exists:
  conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
else:
  conda install pytorch torchvision torchaudio cpuonly -c pytorch
```

**E. Validate environment:**
```bash
conda activate {env_name}
python -c "import sys; print(sys.executable)"
python -c "import torch; print(torch.__version__)"
```

Record:
- `conda.env_name`: selected environment name
- `conda.env_path`: full path to environment
- `conda.python_version`: Python version
- `conda.is_base`: false (always)
- `conda.packages`: key packages and versions

### Step 4 — Check SSH Remote GPU (if configured)

**Check SSH remote FIRST before local hardware (prioritize remote GPU power):**

Read `config.arc.yaml` for SSH configuration:
```yaml
execution:
  ssh_remote:
    host: "gpu-server.example.com"
    user: "username"
    key_path: "~/.ssh/id_rsa"
    conda_env: "arc-research"  # remote conda env name
```

If SSH is configured:
```bash
ssh -o ConnectTimeout=5 -o BatchMode=yes -i {key_path} {user}@{host} "
  nvidia-smi --query-gpu=name,memory.total --format=csv,noheader
" 2>/dev/null
```

- If exit code 0: SSH remote available with GPU
  - Record `ssh_remote.reachable: true`
  - Record remote GPU info
  - **Skip local GPU/CPU detection** (use SSH remote as primary)
  - Go to Step 6 (Final Validation)
- If exit code non-zero: Mark as unreachable, continue to local detection

Record:
- `ssh_remote.configured`: true/false
- `ssh_remote.reachable`: true/false
- `ssh_remote.host`: configured host
- `ssh_remote.gpu_info`: GPU details from remote (if reachable)

### Step 5 — Check Local GPU (CUDA / MPS)

**Only if SSH remote not configured or not reachable.**

**Using the selected conda environment:**

```bash
conda activate {env_name}
python3 -c "
import torch
print('CUDA available:', torch.cuda.is_available())
if torch.cuda.is_available():
    print('Device count:', torch.cuda.device_count())
    print('Device name:', torch.cuda.get_device_name(0))
    print('Memory:', torch.cuda.get_device_properties(0).total_memory / 1e9)
print('MPS available:', torch.backends.mps.is_available() if hasattr(torch.backends, 'mps') else False)
"
```

Record:
- `gpu.available`: true if CUDA or MPS available
- `gpu.type`: GPU name or "Apple Silicon MPS"
- `gpu.memory_gb`: total memory in GB
- `gpu.count`: number of GPUs
- `gpu.backend`: "cuda" or "mps"
- `compute.device`: "cuda:0" or "mps" or "cpu"

### Step 6 — Check Local CPU

Check whether the local machine can serve as a valid execution backend when no higher-priority backend is selected.

```bash
conda activate {env_name}
python3 -c "
import os
import psutil
print('CPU cores:', os.cpu_count())
print('Memory GB:', psutil.virtual_memory().total / 1e9)
"
```

Record:
- `compute.cpu_cores`: number of CPU cores
- `compute.cpu_available`: true
- `compute.memory_gb`: total RAM

### Step 7 — Determine Backend (Priority Order)

**Priority: SSH Remote > Local GPU > Local CPU**

**If user explicitly configured backend:**
- Use the configured backend
- Validate it is available, fail if not (`E00_NO_EXECUTION_ENV`)

**If auto-detecting (no user config):**
Priority order:
1. **ssh_remote** — if configured AND reachable (remote GPU power preferred)
2. **local_gpu** — if CUDA or MPS available
3. **local_cpu** — if no higher-priority backend is available but local execution is still viable

```
Decision Tree:
├─ SSH configured AND reachable? → ssh_remote (priority 1)
├─ CUDA available? → local_gpu (priority 2)
├─ MPS available? → local_gpu with MPS backend (priority 2)
└─ CPU execution available? → local_cpu (priority 3)
```

### Step 8 — Validate Working Directory

```bash
pwd  # current directory
ls -la {working_dir} 2>/dev/null
touch {working_dir}/.probe_test && rm {working_dir}/.probe_test
```

- Default: current working directory (`.`)
- Verify exists and is writable
- Record absolute path in `working_dir`

### Step 9 — Determine pipeline_capable

Calculate `pipeline_capable` based on ALL critical capabilities:

```
pipeline_capable = execution_capable AND latex_capable AND network_capable
```

| Capability | Must Be |
|------------|---------|
| `execution_capable` | true |
| `latex_capable` | true |
| `network_capable` | true (doi.org AND arxiv.org reachable) |

**If ANY is false**:
- Set `pipeline_capable: false`
- Collect all blocking issues in `blocking_issues` array
- Write `environment_error.md` with all fix instructions
- Fail with `E00_CRITICAL_MISSING`

### Step 10 — Write Output

**If pipeline_capable: true:**
```json
{
  "pipeline_capable": true,
  "execution_capable": true,
  "latex_capable": true,
  "network_capable": true,
  "user_configured": false | true,
  "backend": {
    "type": "local_gpu" | "local_cpu" | "ssh_remote",
    "source": "user_config" | "auto_detected"
  },
  "gpu": { ... },
  "compute": { ... },
  "conda": {
    "available": true,
    "env_name": "arc-research",
    "env_path": "/path/to/env",
    "python_version": "3.10.12",
    "is_base": false,
    "packages": { "torch": "2.1.0", ... }
  },
  "latex": {
    "available": true,
    "pdflatex_version": "...",
    "bibtex_version": "..."
  },
  "network": {
    "doi_org_reachable": true,
    "arxiv_org_reachable": true,
    "openalex_api_reachable": true,
    "semantic_scholar_api_reachable": true
  },
  "ssh_remote": { "configured": false | true, "reachable": false | true },
  "working_dir": "/absolute/path/to/workspace",
  "blocking_issues": [],
  "probe_timestamp": "..."
}
```

**If pipeline_capable: false:**
```json
{
  "pipeline_capable": false,
  "execution_capable": false | true,
  "latex_capable": false | true,
  "network_capable": false | true,
  "user_configured": false | true,
  "backend": null | { ... },
  "gpu": { "available": false | true },
  "compute": { "cpu_available": false | true },
  "conda": { "available": false | true, "is_base": true },
  "latex": { "available": false },
  "network": { "doi_org_reachable": false, "arxiv_org_reachable": false },
  "ssh_remote": { "configured": false | true, "reachable": false },
  "working_dir": null | "...",
  "blocking_issues": ["No execution backend", "No LaTeX", "Network unreachable"],
  "probe_timestamp": "...",
  "failure_reason": "Critical capabilities missing: [list]"
}
```

Also write `environment_error.md` with specific fix instructions for EACH missing capability.

### Step 11 — Final Validation

Run all validation checks:
- `pipeline_capable: true` required
- All critical capabilities available
- Working dir writable

**If pipeline_capable: false**:
- Fail with `E00_CRITICAL_MISSING` (blocking, non-retryable)
- Do NOT proceed to Stage 1

No bypass path is allowed. If any critical component is missing, the pipeline remains blocked until the environment is fixed.

---

## Integration with Pipeline

This stage should be invoked:
1. **At pipeline start** — BEFORE Stage 1 to fail fast
2. **Blocks ALL downstream stages** if `pipeline_capable: false`

The orchestrator (`arc-00-01-research-pipeline`) must:
1. Run Stage 0 first
2. Check `pipeline_capable` before proceeding to Stage 1
3. If `pipeline_capable: false`, abort with clear instructions

All downstream stages should:
- Read `environment.json` to determine ALL capabilities
- Use `conda.env_name` for all Python operations
- Require `latex_capable: true` before any LaTeX compilation
- Use `working_dir` as the base path
- Verify network connectivity for literature stages (3-4)

---

## Blocking Summary

| Missing Capability | Blocks | Reason |
|-------------------|--------|--------|
| No execution backend | Stages 10-13 | Prevents fake experiments |
| No LaTeX | Stages 22, 28 | Prevents fake PDFs |
| No network (doi.org/arxiv.org) | Stages 3-4 | Prevents fake citations |

**Anti-Hallucination Guarantee**: The pipeline will NEVER proceed if it cannot verify the authenticity of its outputs. Missing capabilities must be fixed BEFORE execution begins.
