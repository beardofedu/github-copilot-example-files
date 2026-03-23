# GitHub Copilot Example Files

Examples of every customization file you can create to interact with GitHub Copilot.
Each file includes extensive inline comments and development notes to help you
understand both the mechanics and the best practices.

---

## 📁 Files in This Repository

### 1. Custom Instructions — Repository-Wide

**File**: [`.github/copilot-instructions.md`](.github/copilot-instructions.md)

**What it does**: Provides Copilot with repository-specific context that is
automatically included in every Copilot Chat interaction, code review, and
coding agent task for this repository. No manual prompting required.

**Use this for**: Tech stack overview, coding standards, architectural patterns,
team conventions, contribution guidelines.

**Documentation**: [Adding repository custom instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions)

---

### 2. Custom Instructions — Path-Specific

**File**: [`.github/instructions/frontend.instructions.md`](.github/instructions/frontend.instructions.md)

**What it does**: Provides Copilot with instructions that apply only to files
matching a specified glob pattern (defined in the file's YAML frontmatter).
Multiple `*.instructions.md` files can target different parts of the codebase.

**Use this for**: Framework-specific conventions, language-specific style rules,
instructions that only apply to a subset of files (e.g., tests, API layer, frontend).

**Documentation**: [Path-specific custom instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions#creating-path-specific-custom-instructions)

---

### 3. Custom Agent Profile

**File**: [`.github/agents/example-agent.agent.md`](.github/agents/example-agent.agent.md)

**What it does**: Defines a specialized Copilot agent with a custom persona,
a curated set of tools, and detailed behavioral instructions. Custom agents
appear in the agent selector dropdown wherever Copilot coding agent is available.

**Use this for**: Creating specialized agents for specific tasks (e.g., a
dedicated code reviewer, security auditor, or documentation writer).

**Documentation**: [Creating custom agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents)

---

### 4. Agent Skill

**File**: [`.github/skills/example-skill/SKILL.md`](.github/skills/example-skill/SKILL.md)

**What it does**: Defines a reusable skill package that Copilot loads
selectively — only when the skill is relevant to the current task. Skills
contain step-by-step instructions and can reference supplementary scripts
or resources in the same directory.

**Use this for**: Detailed, domain-specific workflows that would be too
verbose to include in `copilot-instructions.md` (e.g., debugging runbooks,
deployment checklists, specialized testing procedures).

**Documentation**: [Creating agent skills](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-skills)

---

### 5. MCP Server Configuration

**File**: [`.vscode/mcp.json`](.vscode/mcp.json)

**What it does**: Configures Model Context Protocol (MCP) servers for use
with Copilot in Visual Studio Code. MCP servers extend Copilot's built-in
capabilities with external tools — for example, fetching web pages, querying
databases, or interacting with GitHub's API.

**Use this for**: Connecting Copilot to external data sources and tools that
aren't available by default (web fetching, internal APIs, GitHub deep integration).

**Documentation**: [Extending Copilot Chat with MCP](https://docs.github.com/en/copilot/how-tos/provide-context/use-mcp/extend-copilot-chat-with-mcp)

---

### 6. Spec-Driven Development Files (spec-kit)

These files implement the **Spec-Driven Development (SDD)** methodology, which
treats specifications as the primary artifact and generates code from them,
rather than writing docs after the fact.

**Documentation**: [spec-kit on GitHub](https://github.com/github/spec-kit)

| File | Purpose |
|---|---|
| [`.github/specs/CONSTITUTION.md`](.github/specs/CONSTITUTION.md) | Foundational project principles, architectural constraints, and development philosophy. Created once; informs all other specs. |
| [`.github/specs/SPEC.md`](.github/specs/SPEC.md) | Product Requirements Document (PRD). Describes WHAT to build and WHY, using user stories and acceptance criteria. No implementation details. |
| [`.github/specs/PLAN.md`](.github/specs/PLAN.md) | Technical implementation plan. Describes HOW to build it — technology choices, architecture, data models, and API contracts. |
| [`.github/specs/TASKS.md`](.github/specs/TASKS.md) | Executable task list derived from the plan. Ordered tasks with clear deliverables, parallel execution markers, and links back to acceptance criteria. |

**The SDD workflow**:
```
CONSTITUTION → SPEC → PLAN → TASKS → IMPLEMENT
```

**Spec-kit commands** (in a Copilot-enabled tool):
```
/speckit.constitution  — Generate the constitution via AI dialogue
/speckit.specify       — Turn a feature description into a structured SPEC
/speckit.plan          — Generate a PLAN from an existing SPEC
/speckit.tasks         — Generate a TASKS list from an existing PLAN
/speckit.implement     — Execute all tasks and build the feature
```

---

## 🔗 Further Reading

- [About customizing GitHub Copilot responses](https://docs.github.com/en/copilot/concepts/prompting/response-customization)
- [Custom agents configuration reference](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [About agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [About Model Context Protocol (MCP)](https://docs.github.com/en/copilot/concepts/about-mcp)
- [MCP servers repository](https://github.com/modelcontextprotocol/servers)
- [spec-kit: Spec-Driven Development toolkit](https://github.com/github/spec-kit)
