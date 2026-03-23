# Copilot Repository Instructions

<!--
  FILE: .github/copilot-instructions.md
  PURPOSE: Repository-wide custom instructions for GitHub Copilot
  SCOPE: Applies to ALL Copilot interactions within this repository

  DOCUMENTATION: https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions

  HOW IT WORKS:
    When Copilot is invoked in the context of this repository (in Copilot Chat,
    code review, coding agent tasks, etc.), the contents of this file are
    automatically included as additional context. You do not need to reference
    this file manually in your prompts.

  WHEN TO USE THIS FILE vs. OTHER INSTRUCTION FILES:
    - Use THIS file (.github/copilot-instructions.md) for instructions that
      apply broadly across ALL files in the repository. Examples: coding
      standards, architectural patterns, team conventions, tech stack overview.
    - Use .github/instructions/*.instructions.md (path-specific files) for
      instructions that only apply to certain file types or directories.
    - Use .github/agents/*.agent.md for creating specialized Copilot agents
      with custom behavior, tools, and personas.

  TIPS FOR WRITING EFFECTIVE INSTRUCTIONS:
    - Be specific and concrete rather than vague ("Use 2-space indentation"
      rather than "Keep code clean").
    - Describe your tech stack so Copilot doesn't suggest incompatible
      libraries or patterns.
    - Document your testing strategy so Copilot generates appropriate tests.
    - Include examples of preferred patterns when helpful.
    - Keep instructions concise — overly long files can dilute the signal.
    - This file supports full Markdown formatting.

  NOTE ON SENSITIVE INFORMATION:
    Do NOT include API keys, passwords, tokens, or other secrets in this file.
    It is committed to source control and visible to everyone with repo access.
-->

## Project Overview

This repository is a collection of example customization files for GitHub Copilot.
It demonstrates how to configure Copilot using every available customization
mechanism: custom instructions, agent profiles, skills, MCP configurations, and
spec-driven development files.

<!--
  DEVELOPMENT NOTE: Describe your project here in 2-4 sentences. This helps
  Copilot understand the domain and purpose of the codebase, which leads to
  more relevant suggestions. Include what problem the project solves and who
  uses it.
-->

## Tech Stack

<!--
  DEVELOPMENT NOTE: List your primary languages, frameworks, and tools. This
  prevents Copilot from suggesting patterns that don't fit your stack. For
  example, if you use React, specify whether you prefer hooks vs. class
  components; if you use TypeScript, note your tsconfig strictness level.
-->

- **Languages**: Markdown, JSON, YAML
- **Purpose**: Documentation and configuration examples only (no application code)
- **Version control**: Git with GitHub

## Coding Standards

<!--
  DEVELOPMENT NOTE: List your team's coding standards here. These become
  implicit requirements Copilot will try to follow in every suggestion.
  Common things to include:
    - Indentation style (spaces vs. tabs, how many)
    - Naming conventions (camelCase, snake_case, PascalCase)
    - Comment style and when to add comments
    - File organization preferences
    - Error handling approach
    - Logging conventions
-->

- Use 2-space indentation for JSON and YAML files
- All Markdown files should use ATX-style headings (`#` prefixes, not underlines)
- Include a comment block at the top of each configuration file explaining its purpose
- Prefer explicit, verbose configuration over terse or implicit defaults
- When in doubt, add a comment explaining the "why" behind a configuration choice

## Contribution Guidelines

<!--
  DEVELOPMENT NOTE: Describe how contributions should be structured. This
  guides Copilot when it generates commits, PRs, or new code. You can
  reference external CONTRIBUTING.md files if you have them.
-->

- Each new example file should include extensive inline comments
- Comments should explain both WHAT the configuration does and WHY it might be used
- Include links to official documentation wherever possible
- New files should follow the existing structure and naming conventions of this repo

## Testing and Validation

<!--
  DEVELOPMENT NOTE: Describe your testing approach. This helps Copilot
  generate appropriate tests when you ask it to add test coverage. Include:
    - Testing framework(s) used
    - Where test files are located
    - Naming conventions for test files
    - Whether you use TDD/BDD
    - Code coverage targets
-->

- No automated test suite (documentation-only repository)
- Validate JSON files with `jq` or a JSON linter before committing
- Validate YAML files with `yamllint` before committing
- Verify that Markdown renders correctly on GitHub before finalizing

## File Structure Reference

<!--
  DEVELOPMENT NOTE: Providing a map of important directories helps Copilot
  place new files in the right location and understand the project layout
  without needing to explore the filesystem.
-->

```
.github/
  copilot-instructions.md   ← This file: repo-wide Copilot instructions
  instructions/             ← Path-specific instruction files
    *.instructions.md
  agents/                   ← Custom Copilot agent profiles
    *.agent.md
  skills/                   ← Agent skill definitions
    <skill-name>/
      SKILL.md
  specs/                    ← Spec-Driven Development files (spec-kit)
    CONSTITUTION.md
    SPEC.md
    PLAN.md
    TASKS.md
.vscode/
  mcp.json                  ← MCP server configuration for VS Code
README.md
```

## AI Agent Behavior

<!--
  DEVELOPMENT NOTE: If you use Copilot coding agent to autonomously work on
  issues and PRs, this section sets expectations for how the agent should
  behave. It's especially useful for communicating things like:
    - Which branches to target
    - Commit message conventions
    - When to ask for clarification vs. make assumptions
    - Whether to run tests before opening a PR
-->

- Always create a new branch from `main` when working on a task
- Write descriptive commit messages in the imperative mood ("Add example", not "Added example")
- Open a draft PR early so progress is visible
- Ask for clarification if a task is ambiguous rather than making assumptions
