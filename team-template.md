# Team Template Specification

**Status:** Draft v1.0
**Date:** 2026-02-21
**License:** Apache 2.0

## Abstract

This specification defines the TeamCard template format used to declare pre-configured agent teams deployable via the AgentTeam CLI. A team template specifies agent roles, engine requirements, skill assignments, room configuration, and prerequisites, enabling one-command deployment of multi-agent teams.

## 1. Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHOULD", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

| Term | Definition |
|------|-----------|
| **Team template** | A TeamCard body that declares a deployable team configuration. |
| **Agent role** | A named position in the team, mapped to an engine and optional skills. |
| **Engine** | An AI tool runtime required to run an agent role. |
| **Skill** | A SkillsCard slug referencing a behavior pattern to load for the agent. |

## 2. Agent Roles

### 2.1 Role Definition

Each agent role describes a position in the team. On deployment, a role becomes a running agent named after the role.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `role` | string | REQUIRED | Role name. MUST match `^[a-z0-9-]+$`. Becomes the agent name (e.g. `tenant-agent` creates `@tenant-agent.{owner}:{server}`). |
| `engine` | string | REQUIRED | Engine type required for this role (e.g. `"claude-code"`, `"gemini-cli"`). |
| `skills` | array of strings | OPTIONAL | SkillsCard slugs to load for this agent. |
| `description` | string | OPTIONAL | Human-readable description of the role's purpose. |
| `config` | object | OPTIONAL | Engine-specific configuration (arbitrary key-value pairs). |

### 2.2 Naming Constraints

Role names follow the same constraints as agent names:

- Lowercase alphanumeric characters and hyphens only: `[a-z0-9-]`
- 1-32 characters
- MUST NOT start or end with a hyphen

### 2.3 Role Example

```json
{
  "role": "code-reviewer",
  "engine": "claude-code",
  "skills": ["code-review", "security-audit"],
  "description": "Reviews pull requests for correctness, security, and style"
}
```

## 3. Room Configuration

### 3.1 Fields

| Field | Type | Description |
|-------|------|-------------|
| `team` | boolean | Whether to create the `#team` coordination room. Defaults to `true`. All agents and the owner are members. |
| `custom` | array of strings | Additional room names. Each room is created with all agents as members. |

### 3.2 Room Behavior

- When `team` is `true` (the default), a `#team` room is created containing all agents and the owner. This serves as the shared coordination channel.
- Each agent always gets a DM room with the owner, regardless of `rooms` configuration.
- Custom rooms are created with all agents as members. Room-specific membership filtering is not supported in this version.

### 3.3 Example

```json
{
  "team": true,
  "custom": ["maintenance-alerts", "daily-standup"]
}
```

## 4. Prerequisites

### 4.1 Fields

| Field | Type | Description |
|-------|------|-------------|
| `engines` | array of strings | Engine types that MUST be installed on the Node before deployment. |
| `tools` | array of strings | MCP tool names that MUST be available before deployment. |

### 4.2 Validation

Before deploying a team template, the CLI MUST:

1. Verify that all engines listed in `prerequisites.engines` are detected on the system.
2. Verify that all tools listed in `prerequisites.tools` are available.
3. Verify that each agent role's `engine` is available (either via `prerequisites` or direct detection).
4. If any prerequisite is missing, display missing items and suggest alternative engines. Deployment continues with user confirmation; hard abort only if no compatible engine exists.

### 4.3 Example

```json
{
  "engines": ["claude-code", "gemini-cli"],
  "tools": ["browser-automation"]
}
```

## 5. CLI Deployment

### 5.1 Command

```bash
agentteam up --template {slug}
```

The `cli_command` field in the TeamCard body records the canonical deployment command. It MUST match the pattern `^agentteam up --template [a-z0-9-]+$`.

### 5.2 Deployment Sequence

When a user runs `agentteam up --template {slug}`, the CLI:

1. Fetches the TeamCard from the registry (`teamcard.cc/api/v1/teams/{slug}.json`).
2. Displays the team configuration (roles, engines, skills, rooms) for user review.
3. Validates prerequisites (Section 4.2).
4. For each agent role, executes the equivalent of:

   ```sh
   agentteam agent add {role} --engine {engine}
   ```

5. Loads referenced skills for each agent.
6. Creates rooms as configured (DM rooms + `#team` room + custom rooms).
7. Starts all agents.

### 5.3 Idempotency

Re-running `agentteam up --template {slug}` SHOULD be safe. If agents from the template already exist, the CLI SHOULD skip creation and report which agents were already present.

## 6. Validation

### 6.1 Required Checks

A TeamCard MUST pass the following validation before being accepted into the registry:

1. **Schema validation**: The card MUST conform to the TeamCard body JSON Schema.
2. **Role uniqueness**: No two entries in `agents` MAY have the same `role` value.
3. **Slug consistency**: If `cli_command` is present, the slug in the command MUST match the card's envelope `slug`.
4. **Skill references**: All skill slugs referenced in agent roles SHOULD exist in the SkillsCard registry. Missing references produce a warning, not an error.

### 6.2 Runtime Checks

At deployment time (not registry time), the CLI performs additional checks:

- Engine availability for each role.
- MCP tool availability from `prerequisites.tools`.
- Network connectivity to the AgentTeam platform.

## 7. Complete Example

```json
{
  "kind": "team",
  "slug": "dev-team",
  "version": "1.0.0",
  "name": "Development Team",
  "description": "Code review, testing, and documentation agents for software projects",
  "author": {
    "name": "agentteam",
    "matrix_id": "@qi:agentteam.app"
  },
  "tags": ["development", "engineering"],
  "license": "Apache-2.0",
  "body": {
    "agents": [
      {
        "role": "reviewer",
        "engine": "claude-code",
        "skills": ["code-review", "security-audit"],
        "description": "Reviews PRs for correctness, security, and style"
      },
      {
        "role": "tester",
        "engine": "claude-code",
        "skills": ["testing"],
        "description": "Writes and maintains test suites"
      },
      {
        "role": "documenter",
        "engine": "gemini-cli",
        "skills": ["documentation"],
        "description": "Keeps project documentation up to date"
      }
    ],
    "rooms": {
      "team": true
    },
    "cli_command": "agentteam up --template dev-team",
    "prerequisites": {
      "engines": ["claude-code", "gemini-cli"]
    }
  }
}
```
