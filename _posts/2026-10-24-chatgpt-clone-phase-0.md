---
title: "Building a Small ChatGPT Clone: Phase 0 - Environment Setup and Cloud Configuration"
date: 2026-10-24 14:30:00 +0200
categories: [CloudComputing, Phase 0]
tags: [GCP, Python, Git, CloudShell, ChatGPT, AI, Development]
toc: true
math: true
---
# Implement my small chatGPT - Phase 0

## First step

During this phase, I am focusing on setting up all the accounts and services required to complete this project.
I will be using **Google Cloud Platform (GCP)** as the main environment to implement and deploy everything.

This includes:

- Creating and configuring my GCP project
- Setting up billing and budget alerts
- Verifying my Cloud Shell and gcloud configuration
- Preparing the repository and development workflow
- Ensuring that all required tools and permissions are correctly initialized

You can follow the progress directly on my GitHub repository (link below).
I update it regularly as I advance through the setup and implementation steps.

This phase 0 needs a script validator to check whether my development is correctly configured before I start the rest of the project. 

```python
#!/usr/bin/env python3
"""Phase 0 self-check — probes your environment and writes a submission report.

Run this from the repository root once you have finished the setup tasks::

    python phase-0-setup/verify_setup.py

It inspects your machine (Python version, gcloud config, Docker, git remote) and
writes ``submission/phase0_report.json``. Commit that file and push — your CI and the
instructor grade it.

Two small functions are left for YOU to implement (search for ``TODO``):

  * ``parse_gcloud_config`` — pull the account + project out of ``gcloud`` JSON output.
  * ``parse_repo_slug``     — turn a git remote URL into ``owner/repo``.

Everything else is provided. These are pure functions (input string in, value out),
so you can test them without touching the cloud — see ``tests/test_units.py``.
"""

from __future__ import annotations

import json
import sys
from pathlib import Path

# Make the shared `autograder` package importable when run from anywhere.
REPO_ROOT = Path(__file__).resolve().parents[1]
if str(REPO_ROOT) not in sys.path:
    sys.path.insert(0, str(REPO_ROOT))

from autograder import env_checks  # noqa: E402

REPORT_PATH = REPO_ROOT / "submission" / "phase0_report.json"

# --------------------------------------------------------------------------- #
# Functions YOU implement
# --------------------------------------------------------------------------- #
def parse_gcloud_config(raw: str) -> dict:
    """Parse the JSON emitted by ``gcloud config list --format=json``.

    Return ``{"account": <str>, "project": <str>}`` using empty strings when a
    value is absent. The relevant fields live under the ``"core"`` section, e.g.::

        {"core": {"account": "you@example.com", "project": "my-proj-123"}}

    Args:
        raw: the raw stdout string from the gcloud command (may be empty).
    """
    # TODO: parse `raw` as JSON and return {"account": ..., "project": ...}
    #       from the "core" section. Return empty strings if raw is empty or a
    #       field is missing.
    raise NotImplementedError("Phase 0: implement parse_gcloud_config()")

def parse_repo_slug(remote_url: str) -> str:
    """Turn a GitHub remote URL into an ``owner/repo`` slug (no ``.git`` suffix).

    Must handle both forms git prints:

      * HTTPS: ``https://github.com/eurecom/clouds-lab.git``  -> ``eurecom/clouds-lab``
      * SSH:   ``git@github.com:eurecom/clouds-lab.git``      -> ``eurecom/clouds-lab``

    Return ``""`` if the URL is empty or not a github.com URL.

    Args:
        remote_url: output of ``git remote get-url origin`` (may be empty).
    """
    # TODO: return "owner/repo" for both the HTTPS and SSH forms shown above.
    #       Strip any trailing ".git". Return "" if not a github.com URL.
    raise NotImplementedError("Phase 0: implement parse_repo_slug()")

# --------------------------------------------------------------------------- #
# Provided probes (do not modify)
# --------------------------------------------------------------------------- #
def check_python() -> dict:
    major, minor, micro = env_checks.python_version()
    return {
        "version": f"{major}.{minor}.{micro}",
        "ok": env_checks.python_is_supported(),
    }

def check_environment() -> dict:
    """Record where this ran. Cloud Shell is the mandated dev environment; local is allowed."""
    return {"cloud_shell": env_checks.in_cloud_shell()}

def check_gcloud() -> dict:
    code, out, err = env_checks.run(["gcloud", "config", "list", "--format=json"])
    if code != 0:
        return {"account": "", "project": "", "error": err or "gcloud not found"}
    try:
        parsed = parse_gcloud_config(out)
    except NotImplementedError as exc:
        return {"account": "", "project": "", "error": str(exc)}
    return {"account": parsed["account"], "project": parsed["project"], "error": None}

def check_docker() -> dict:
    code, out, err = env_checks.run(["docker", "--version"])
    return {"version": out if code == 0 else "", "present": code == 0}

def check_git() -> dict:
    url = env_checks.git_remote_url()
    try:
        slug = parse_repo_slug(url)
    except NotImplementedError:
        slug = ""
    return {
        "remote": url,
        "is_github": "github.com" in url,
        "slug": slug,
    }

def build_report() -> dict:
    return {
        "phase": "0",
        "environment": check_environment(),
        "python": check_python(),
        "gcloud": check_gcloud(),
        "docker": check_docker(),
        "git": check_git(),
    }

def _print_summary(report: dict) -> None:
    def mark(ok: bool) -> str:
        return "OK  " if ok else "MISS"

    py = report["python"]
    gc = report["gcloud"]
    dk = report["docker"]
    gt = report["git"]
    env = report["environment"]
    print("Phase 0 environment check")
    print("-" * 40)
    where = "Cloud Shell" if env["cloud_shell"] else "local machine"
    print(f"[OK  ] Running on: {where}")
    print(f"[{mark(py['ok'])}] Python {py['version']} (need 3.11 or 3.12)")
    print(f"[{mark(bool(gc['account']))}] gcloud account: {gc['account'] or '(unset)'}")
    print(f"[{mark(bool(gc['project']))}] gcloud project: {gc['project'] or '(unset)'}")
    print(f"[{mark(dk['present'])}] {dk['version'] or 'docker not found'}")
    print(f"[{mark(gt['is_github'])}] git remote: {gt['remote'] or '(unset)'}")
    print("-" * 40)

def main() -> int:
    report = build_report()
    REPORT_PATH.parent.mkdir(parents=True, exist_ok=True)
    REPORT_PATH.write_text(json.dumps(report, indent=2), encoding="utf-8")
    _print_summary(report)
    print(f"\nWrote {REPORT_PATH.relative_to(REPO_ROOT)} — commit it and push.")
    return 0

if __name__ == "__main__":
    raise SystemExit(main())
```

## Mission

My first mission was to implement two small functions. 

**Parse GCloud configuration**

```python
def parse_gcloud_config(raw: str) -> dict:
    """Parse the JSON emitted by ``gcloud config list --format=json``.

    Return ``{"account": <str>, "project": <str>}`` using empty strings when a
    value is absent. The relevant fields live under the ``"core"`` section, e.g.::

        {"core": {"account": "you@example.com", "project": "my-proj-123"}}

    Args:
        raw: the raw stdout string from the gcloud command (may be empty).
    """
    # TODO: parse `raw` as JSON and return {"account": ..., "project": ...}
    #       from the "core" section. Return empty strings if raw is empty or a
    #       field is missing.
    raise NotImplementedError("Phase 0: implement parse_gcloud_config()")
```

**First attempt**

```python
def parse_gcloud_config(raw: str) -> dict:

    if not raw.strip():
        return {"account": "", "project": ""}

    data = json.loads(raw)
    core = data.get("core", {})

    account = core.get("account", "")
    project = core.get("project", "")

    return {
        "account": account,
        "project": project
    }
```

**What is raw?** 

`raw` is the **exact text output** you get when you run this command:

```bash
gcloud config list --format=json
#Output:
{
  "accessibility": {
    "screen_reader": "True"
  },
  "component_manager": {
    "disable_update_check": "True"
  },
  "compute": {
    "gce_metadata_read_timeout_sec": "30"
  },
  "core": {
    "account": "gabriel.lapointe@eurecom.fr",
    "disable_usage_reporting": "False",
    "project": "project-54f8e752-5266-4cfd-9ff",
    "universe_domain": "googleapis.com"
  },
  "metrics": {
    "environment": "devshell"
  }
}
```

**Output Goal**

```json
{
    "account": "gabriel.lapointe@eurecom.fr",
    "project": "project-54f8e752-5266-4cfd-9ff",
}
```

The first thing is to ensure that JSON has been implemented which is actually the case: 

```python
import json
```

**Parse Repository slug**

```python
def parse_repo_slug(remote_url: str) -> str:
    """Turn a GitHub remote URL into an ``owner/repo`` slug (no ``.git`` suffix).

    Must handle both forms git prints:

      * HTTPS: ``https://github.com/eurecom/clouds-lab.git``  -> ``eurecom/clouds-lab``
      * SSH:   ``git@github.com:eurecom/clouds-lab.git``      -> ``eurecom/clouds-lab``

    Return ``""`` if the URL is empty or not a github.com URL.

    Args:
        remote_url: output of ``git remote get-url origin`` (may be empty).
    """
    # TODO: return "owner/repo" for both the HTTPS and SSH forms shown above.
    #       Strip any trailing ".git". Return "" if not a github.com URL.
    raise NotImplementedError("Phase 0: implement parse_repo_slug()")
```

Actually, this happens: 

```python
git remote get-url origin
#Output
https://github.com/Van-tran-19/cloud.git
```

After learning the necessary, this is my final code: 

```python
def parse_repo_slug(remote_url: str) -> str:
    if not remote_url.strip():
        return ""

    if remote_url.startswith("https://github.com"):
        parts = remote_url.rstrip(".git").split("/")
        owner = parts[-2]
        repo = parts[-1]
        return f"{owner}/{repo}"

    if remote_url.startswith("git@github.com"):
        slug = remote_url.split(":")[1]
        return slug.rstrip(".git")

    return ""
```

## Learning

**How to convert a JSON string into a python dictionary?** 

A JSON string is just : 

```json
{
  "core": {
    "account": "gabriel@example.com",
    "project": "my-proj-123"
  }
}
```

Inside Python, this arrives as **one big string**, not a dictionary.

```python
"{\n  \"core\": {\n    \"account\": \"gabriel@example.com\",\n    \"project\": \"my-proj-123\"\n  }\n}\n"

```

My function receives this text as the variable `raw`.

```python
data = json.loads(raw)
#by using .loads(JSON input), it transforms the JSON into a python dictionary
```

**How to access keys in dictionary?** 

```python
#we can use .get(), for example 
core = data.get("core", {})
account = core.get("account", "")
#if the key dosen't exist, .get() avoids errors
```

**How to return a dictionary?** 

```python
return {"account": account, "project": project}
```

**String methods**

| Methods | what |
| --- | --- |
| `.strip()` | To check if a string is empty or only whitespace. |
| `.startswith()` | To detect HTTPS vs SSH Git URLs. |
| `.split()` | To break URLs into parts. |
| `.rstrip(".git")` | To remove the `.git` suffix. |

**How `.split()` works?** 

```python
url = "https://github.com/eurecom/clouds-lab.git"

parts = url.split("/")
print(parts)
#output: 
['https:', '', 'github.com', 'eurecom', 'clouds-lab.git']
```

## Tools

1. https://github.com/Van-tran-19/cloud
2. https://shell.cloud.google.com/?show=ide%2Cterminal
3. https://cloud.google.com/shell/docs/using-cloud-shell

## Sources

1. [https://cloud.google.com/learn/what-is-iaas](https://cloud.google.com/learn/what-is-iaas)
2. [https://cloud.google.com/learn/paas-vs-iaas-vs-saas](https://cloud.google.com/learn/paas-vs-iaas-vs-saas)
3. https://cloud.google.com/free/docs/free-cloud-features#free-tier
