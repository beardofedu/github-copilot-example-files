---
# FILE: .github/skills/example-skill/SKILL.md
#
# PURPOSE: Example agent skill definition for GitHub Copilot
#
# DOCUMENTATION:
#   https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-skills
#   https://docs.github.com/en/copilot/concepts/agents/about-agent-skills
#
# IMPORTANT: This file MUST be named SKILL.md (uppercase). Other names are not recognized.
#
# HOW IT WORKS:
#   Agent skills are modular instruction packages that Copilot loads
#   dynamically — only when the skill is relevant to the current task.
#   Unlike copilot-instructions.md (always loaded), skills are loaded
#   selectively based on the task description matching the skill's
#   "description" field.
#
# WHY USE SKILLS INSTEAD OF CUSTOM INSTRUCTIONS?
#   Custom instructions (.github/copilot-instructions.md) are loaded for EVERY
#   Copilot interaction in the repo. This is appropriate for general standards.
#   Skills are loaded ONLY when needed, which is better for:
#     - Detailed, step-by-step processes (e.g., a deployment checklist)
#     - Domain-specific workflows (e.g., how to debug a specific service)
#     - Instructions that would clutter context if always present
#     - Reusable skills you want to share across multiple projects
#
# FILE LOCATION:
#   PROJECT SKILLS (this repo only):
#     .github/skills/<skill-name>/SKILL.md
#     .claude/skills/<skill-name>/SKILL.md
#   PERSONAL SKILLS (available across all your projects):
#     ~/.copilot/skills/<skill-name>/SKILL.md
#     ~/.claude/skills/<skill-name>/SKILL.md
#
# DIRECTORY NAMING:
#   The skill directory name should:
#     - Be lowercase
#     - Use hyphens instead of spaces
#     - Match the "name" field in the frontmatter (by convention)
#   Example: .github/skills/github-actions-debugging/SKILL.md
#
# FRONTMATTER FIELDS:
#   name        - (required) Unique identifier. Must be lowercase with hyphens.
#                 Typically matches the directory name.
#   description - (required) Explains what this skill does and — critically —
#                 WHEN Copilot should use it. Copilot uses this description to
#                 decide whether to load the skill based on your prompt.
#                 Write this as: "Use this skill when [situation]."
#   license     - (optional) License that applies to this skill's content.
#
# ADDITIONAL FILES IN THE SKILL DIRECTORY:
#   You can include supplementary resources alongside SKILL.md:
#     - Additional Markdown files with detailed reference information
#     - Scripts that the skill instructions tell Copilot to run
#     - Template files or examples
#   Reference these files by their relative path in your SKILL.md instructions.

name: github-actions-debugging
description: >
  Provides a systematic process for diagnosing and fixing failing GitHub Actions
  workflows. Use this skill when asked to debug, investigate, or fix a failing
  CI/CD pipeline, workflow, or GitHub Actions job.
license: MIT
---

# GitHub Actions Debugging Skill

<!--
  SKILL INSTRUCTIONS:
    The Markdown body below is injected into Copilot's context when this
    skill is selected. Write it as a clear, actionable guide that tells
    Copilot exactly how to approach the task.

  STRUCTURE TIPS:
    - Use numbered steps for sequential processes
    - Use headers to organize logical phases
    - Include concrete examples, not just abstractions
    - Reference any scripts or helper files in this directory
    - Be explicit about which tools or commands to use
-->

## Overview

Use this systematic approach to diagnose and fix failing GitHub Actions workflows.
Work through each phase in order, gathering information before making changes.

## Phase 1: Gather Information

### 1.1 Identify the failing run

Use the `list_workflow_runs` tool from the GitHub MCP server to find recent
workflow runs and identify which one is failing:

```
list_workflow_runs(owner=<owner>, repo=<repo>, status="failure")
```

Note the `run_id` of the failing run.

### 1.2 Get a summary of failures

Before downloading full logs (which can be very large), use the
`summarize_job_log_failures` tool to get an AI-generated summary:

```
summarize_job_log_failures(owner=<owner>, repo=<repo>, run_id=<run_id>)
```

This prevents overwhelming your context window with thousands of log lines.

### 1.3 Get full logs if needed

If the summary isn't sufficient, retrieve the detailed logs for specific
failed jobs using `get_job_logs`:

```
get_job_logs(owner=<owner>, repo=<repo>, job_id=<job_id>)
```

## Phase 2: Understand the Failure

After gathering logs, categorize the failure type before attempting a fix:

| Failure Type | Common Causes | Typical Fix |
|---|---|---|
| **Build failure** | Syntax error, missing dependency, compilation error | Fix the code or update dependencies |
| **Test failure** | Broken test, flaky test, environment mismatch | Fix the test or the code under test |
| **Linting failure** | Code style violation | Run the linter locally and fix violations |
| **Authentication failure** | Expired secret, missing permissions | Rotate secrets or update repo permissions |
| **Infrastructure failure** | Runner unavailability, network issue | Retry the workflow; escalate if persistent |
| **Dependency failure** | Package not found, version conflict | Update `package.json` / `requirements.txt` |

## Phase 3: Reproduce Locally

When possible, reproduce the failure in your own environment before fixing:

1. Check out the branch that triggered the failing workflow
2. Install the same tool versions used in the workflow (check `uses:` versions)
3. Set any required environment variables (without real secret values)
4. Run the failing command directly

Reproducing locally gives you fast feedback while iterating on a fix.

## Phase 4: Apply the Fix

Make the smallest change that resolves the root cause:

- **Don't mask symptoms**: A passing build with `|| true` appended to a failing
  command is not a fix — it hides a real problem.
- **Fix the root cause**: If a test reveals a bug, fix the bug (not just the test).
- **Update pinned versions carefully**: Check changelogs before upgrading
  actions or dependencies to newer versions.
- **Verify environment variables and secrets**: Missing secrets won't appear in
  logs; check the repo's Settings → Secrets and Variables if auth is failing.

## Phase 5: Verify

After applying a fix:

1. Push your change and wait for the workflow to run
2. Use `list_workflow_runs` to confirm the run succeeded
3. Check that other jobs in the workflow also passed (not just the one that was failing)
4. If the workflow was flaky (intermittently failing), run it several times to
   confirm consistent success

## Common Gotchas

- **Action version pinning**: Using `@main` or `@latest` can break builds when
  the action author pushes a breaking change. Pin to a specific SHA or version tag.
- **YAML indentation**: GitHub Actions YAML is whitespace-sensitive. A single
  extra space can cause confusing parse errors.
- **Matrix job failures**: One failing matrix combination fails the entire job.
  Check the matrix section for environment-specific issues.
- **Permissions**: Workflows need explicit `permissions:` blocks in repositories
  with restricted default permissions. Check for `403` errors in the logs.
- **Cache invalidation**: A stale cache can cause hard-to-reproduce failures.
  Try clearing the Actions cache from the repository's Actions tab.
