---
name: "rl-security-review"
description: "Use when reviewing ML or RL research code for security problems: unsafe pickle or torch.load deserialization of model checkpoints, subprocess command injection from config values, exposed Ray cluster and Ray Tune ports, and unauthenticated FastAPI control endpoints. Scoped to research and ML stacks, not general web application security."
---

# RL / ML Security Review Skill — Research Stack Security Audit

## Core Rule
> Unsafe pickle deserialization (`pickle.load()`) and unauthenticated Ray cluster ports are severe security vulnerabilities.

Scope: research and ML stacks. For general web application security, use a general-purpose security review instead.

---

## 1. Vulnerability Checklist for ML/RL Stacks

1. **Unsafe Model Loading**: Avoid untrusted `pickle.load()` or `torch.load()` without `weights_only`:
   - **Unsafe**: `torch.load("checkpoint.pt")`
   - **Safe**: `torch.load("checkpoint.pt", weights_only=True)` or Safetensors.
   - A checkpoint downloaded from a model hub is untrusted input. Treat it as such.
2. **Subprocess Injection**: Never pass raw user or config input to `subprocess.Popen(shell=True)`. Config values interpolated into a shell command are an injection vector — pass an argument list, not a string.
3. **Ray Tune / FastAPI Port Binding**: Bind Ray dashboard and FastAPI endpoints to `127.0.0.1` instead of `0.0.0.0` in non-isolated environments. An open Ray dashboard port allows arbitrary remote job submission.
4. **Cluster Credentials**: No API keys, WandB tokens, or SSH keys in config files that get committed. Read them from the environment.

---

## 2. Quick Audit Commands

```bash
# Unsafe deserialization
grep -rnE "torch\.load\(|pickle\.load\(|yaml\.load\(" --include="*.py" .

# Shell injection surface
grep -rnE "shell\s*=\s*True|os\.system\(" --include="*.py" .

# Wide-open binds
grep -rnE "0\.0\.0\.0|--host[= ]0\.0\.0\.0" --include="*.py" --include="*.yaml" .

# Committed secrets
git log --all -S "api_key" --oneline
```

