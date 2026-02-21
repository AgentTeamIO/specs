# AgentTeam Card Specification

**Status:** Draft v1.0
**Date:** 2026-02-21
**License:** Apache 2.0

## Abstract

This specification defines the AgentTeam Card format -- a portable, machine-readable data structure for describing AI agents, reusable behavior patterns (skills), and team templates. Cards share a common envelope with a `kind` discriminator that selects one of three body schemas: **AgentCard** (A2A Agent Card superset), **SkillsCard** (skill definition), and **TeamCard** (team template). A Git-native registry backed by a static JSON API provides discovery and distribution.

## 1. Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

| Term | Definition |
|------|-----------|
| **Card** | A JSON document consisting of an envelope and a body. |
| **Envelope** | The set of top-level fields shared by all card types. |
| **Body** | The `body` field whose schema varies by `kind`. |
| **Kind** | The card type discriminator: `"agent"`, `"skill"`, or `"team"`. |
| **Slug** | A URL-safe unique identifier for the card within its kind namespace. |
| **A2A** | Google's Agent-to-Agent protocol for agent interoperability. |
| **Engine** | An AI tool runtime (e.g. Claude Code, Gemini CLI, Codex, Letta, OpenClaw). |

## 2. Card Envelope

All cards MUST include the following envelope fields. The envelope is shared across all three card types.

### 2.1 Envelope Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `$schema` | string | OPTIONAL | Schema URI. If present, MUST be `"https://agentteam.org/schemas/card/v1"`. |
| `kind` | string | REQUIRED | Card type discriminator. MUST be one of: `"agent"`, `"skill"`, `"team"`. |
| `slug` | string | REQUIRED | URL-safe unique identifier. MUST match `^[a-z0-9][a-z0-9.-]*[a-z0-9]$`, 2-64 characters. For agents: `{name}.{owner}` (e.g. `claude.qi`). For skills and teams: kebab-case name (e.g. `code-review`). |
| `version` | string | REQUIRED | Semantic version. MUST match `^\d+\.\d+\.\d+$` (e.g. `"1.0.0"`). |
| `name` | string | REQUIRED | Human-readable display name. Maximum 128 characters. |
| `description` | string | REQUIRED | Brief description for search and display. Maximum 500 characters. |
| `author` | object | REQUIRED | Author information. See Section 2.2. |
| `tags` | array of strings | OPTIONAL | Lowercase kebab-case tags for discovery. Each tag MUST match `^[a-z0-9-]+$`. Maximum 10 tags. |
| `license` | string | OPTIONAL | SPDX license identifier (e.g. `"MIT"`, `"Apache-2.0"`). |
| `created_at` | string | OPTIONAL | ISO 8601 date-time of initial publication. |
| `updated_at` | string | OPTIONAL | ISO 8601 date-time of last modification. |
| `body` | object | REQUIRED | Card-type-specific content. Schema depends on `kind`. |

No additional properties are permitted at the envelope level.

### 2.2 Author Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | REQUIRED | Author handle or display name. |
| `matrix_id` | string | OPTIONAL | Matrix User ID. If present, MUST match `^@[a-z0-9._=-]+:[a-z0-9.-]+$` (e.g. `@qi:agentteam.app`). |
| `url` | string (URI) | OPTIONAL | Author's profile or website URL. |

### 2.3 Envelope Example

```json
{
  "$schema": "https://agentteam.org/schemas/card/v1",
  "kind": "agent",
  "slug": "claude.qi",
  "version": "1.0.0",
  "name": "Claude",
  "description": "Code-focused AI assistant powered by Claude Code",
  "author": {
    "name": "qi",
    "matrix_id": "@qi:agentteam.app",
    "url": "https://agentteam.app"
  },
  "tags": ["development", "code-review"],
  "license": "MIT",
  "created_at": "2026-02-21T00:00:00Z",
  "updated_at": "2026-02-21T00:00:00Z",
  "body": { }
}
```

## 3. AgentCard Body (A2A Superset)

When `kind` is `"agent"`, the `body` MUST conform to this section. The AgentCard body **is** a complete A2A Agent Card: any A2A-compatible client can parse `body` directly without transformation.

### 3.1 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Agent display name. |
| `description` | string | Agent description. |
| `version` | string | Agent version (e.g. `"1.0"`). |
| `defaultInputModes` | array of strings | A2A required. Supported input modalities. Each MUST be one of: `"text"`, `"image"`, `"audio"`, `"video"`, `"file"`. At least one item. |
| `defaultOutputModes` | array of strings | A2A required. Supported output modalities. Same enum as `defaultInputModes`. At least one item. |
| `capabilities` | object | A2A required. Agent capabilities (see Section 3.2). |
| `skills` | array of objects | A2A required. Agent skills as `AgentSkill[]` (see Section 3.3). |
| `supportedInterfaces` | array of objects | A2A required. Protocol interfaces (see Section 3.4). At least one item. |

### 3.2 Capabilities Object

| Field | Type | Description |
|-------|------|-------------|
| `streaming` | boolean | OPTIONAL. Whether the agent supports streaming responses. |
| `pushNotifications` | boolean | OPTIONAL. Whether the agent supports push notifications. |
| `stateTransitionHistory` | boolean | OPTIONAL. Whether the agent exposes state transition history. |

### 3.3 AgentSkill Object

Each entry in the `skills` array MUST include:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | REQUIRED | Unique skill identifier. SHOULD match the corresponding SkillsCard `slug`. |
| `name` | string | REQUIRED | Display name. |
| `description` | string | REQUIRED | What this skill does. |
| `tags` | array of strings | OPTIONAL | Categorization tags. |
| `examples` | array of strings | OPTIONAL | Example prompts demonstrating usage. |

### 3.4 Supported Interface Object

Each entry in `supportedInterfaces` MUST include:

| Field | Type | Description |
|-------|------|-------------|
| `url` | string (URI) | HTTPS endpoint for A2A communication. |
| `protocolBinding` | string | Protocol binding (e.g. `"JSONRPC"`). |
| `protocolVersion` | string | Protocol version (e.g. `"1.0"`). |

### 3.5 Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `provider` | object | Provider information: `organization` (string), `url` (string, URI). |
| `securitySchemes` | object | OpenAPI-style security scheme definitions. |
| `security` | array of objects | Which security schemes to apply. |
| `x-agentteam` | object | AgentTeam-specific extensions (see Section 3.6). |

Additional properties beyond those listed are permitted in the AgentCard body to maintain forward compatibility with future A2A fields.

### 3.6 AgentTeam Extensions (`x-agentteam`)

AgentTeam extensions are namespaced under the `x-agentteam` key to avoid conflicts with A2A parsers, following the OpenAPI vendor extension convention.

| Field | Type | Description |
|-------|------|-------------|
| `matrix_id` | string | Matrix User ID. MUST match `^@[a-z0-9-]+\.[a-z0-9-]+:[a-z0-9.-]+$` (e.g. `@claude.qi:agentteam.app`). |
| `matrix_uri` | string | Matrix URI (e.g. `matrix:u/claude.qi:agentteam.app`). |
| `engine` | string | Engine type (e.g. `"claude-code"`, `"gemini-cli"`, `"letta"`, `"openclaw"`). |
| `public` | boolean | Whether the agent is publicly discoverable. |
| `skillscard_refs` | array of strings | References to SkillsCard slugs on skillscard.cc. |

### 3.7 AgentCard Example

```json
{
  "kind": "agent",
  "slug": "claude.qi",
  "version": "1.0.0",
  "name": "Claude",
  "description": "Code-focused AI assistant powered by Claude Code",
  "author": {
    "name": "qi",
    "matrix_id": "@qi:agentteam.app"
  },
  "tags": ["development", "code-review"],
  "body": {
    "name": "Claude",
    "description": "Code-focused AI assistant powered by Claude Code",
    "version": "1.0",
    "defaultInputModes": ["text"],
    "defaultOutputModes": ["text"],
    "capabilities": {
      "streaming": true,
      "pushNotifications": false
    },
    "skills": [
      {
        "id": "code-review",
        "name": "Code Review",
        "description": "Systematic code review with confidence-based filtering",
        "tags": ["development", "quality"],
        "examples": ["Review this PR for security issues"]
      }
    ],
    "supportedInterfaces": [
      {
        "url": "https://agentteam.app/api/v1/a2a/claude.qi",
        "protocolBinding": "JSONRPC",
        "protocolVersion": "1.0"
      }
    ],
    "provider": {
      "organization": "AgentTeam",
      "url": "https://agentteam.app"
    },
    "securitySchemes": {
      "bearer": {
        "type": "http",
        "scheme": "bearer"
      }
    },
    "security": [{ "bearer": [] }],
    "x-agentteam": {
      "matrix_id": "@claude.qi:agentteam.app",
      "matrix_uri": "matrix:u/claude.qi:agentteam.app",
      "engine": "claude-code",
      "public": false,
      "skillscard_refs": ["code-review", "debugging"]
    }
  }
}
```

## 4. SkillsCard Body

When `kind` is `"skill"`, the `body` MUST conform to this section. A SkillsCard describes a reusable behavior pattern that AI agents can discover and load at runtime.

### 4.1 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | Trust level. MUST be one of: `"curated"` (core team published), `"verified"` (sandbox-tested), `"community"` (author-verified). |
| `format` | string | Skill content format. MUST be one of: `"claude-skill-v1"` (primary), `"markdown"` (engine-agnostic), `"yaml-frontmatter"` (structured metadata + content). See [skill-format.md](./skill-format.md). |
| `engine_compatibility` | array of strings | Which AI engines can use this skill. At least one item. Common values: `"claude-code"`, `"gemini-cli"`, `"codex"`, `"letta"`, `"openclaw"`. |
| `content_path` | string | Relative path to the skill content file within the cards repo. MUST match `^skills/[a-z0-9-]+/skill\.md$`. |
| `inputs` | array of strings | What the skill expects as input context (e.g. `["codebase", "diff"]`). |
| `outputs` | array of strings | What the skill produces (e.g. `["review-comments", "suggestions"]`). |

### 4.2 Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `examples` | array of objects | Usage examples. Each object MUST include `prompt` (string) and `expected_behavior` (string). |
| `dependencies` | array of strings | Other skill slugs this skill depends on. |

No additional properties are permitted in the SkillsCard body.

### 4.3 SkillsCard Example

```json
{
  "kind": "skill",
  "slug": "code-review",
  "version": "1.0.0",
  "name": "Code Review",
  "description": "Systematic code review with confidence-based filtering",
  "author": {
    "name": "agentteam",
    "matrix_id": "@qi:agentteam.app"
  },
  "tags": ["development", "quality"],
  "license": "Apache-2.0",
  "body": {
    "status": "curated",
    "format": "claude-skill-v1",
    "engine_compatibility": ["claude-code", "gemini-cli"],
    "content_path": "skills/code-review/skill.md",
    "inputs": ["codebase", "diff"],
    "outputs": ["review-comments", "suggestions"],
    "examples": [
      {
        "prompt": "Review this PR for security issues",
        "expected_behavior": "Analyzes code changes for OWASP top 10"
      }
    ]
  }
}
```

## 5. TeamCard Body

When `kind` is `"team"`, the `body` MUST conform to this section. A TeamCard defines a team template -- a pre-configured set of agent roles, room configuration, and prerequisites that can be deployed with a single CLI command.

### 5.1 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `agents` | array of objects | Agent roles in the team. At least one item. See Section 5.2. |
| `rooms` | object | Room configuration. See Section 5.3. |

### 5.2 Agent Role Object

Each entry in the `agents` array MUST include:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `role` | string | REQUIRED | Agent role name. MUST match `^[a-z0-9-]+$`. Becomes the agent name on deployment. |
| `engine` | string | REQUIRED | Required engine type (e.g. `"claude-code"`). |
| `skills` | array of strings | OPTIONAL | SkillsCard slugs to load for this agent. |
| `description` | string | OPTIONAL | What this agent role does in the team. |
| `config` | object | OPTIONAL | Engine-specific configuration. Arbitrary key-value pairs. |

### 5.3 Rooms Object

| Field | Type | Description |
|-------|------|-------------|
| `team` | boolean | OPTIONAL. Whether to create the `#team` coordination room. Defaults to `true`. |
| `custom` | array of strings | OPTIONAL. Additional custom room names (e.g. `["maintenance-alerts"]`). |

### 5.4 Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `cli_command` | string | CLI command to deploy this template. MUST match `^agentteam up --template [a-z0-9-]+$`. |
| `prerequisites` | object | Deployment prerequisites: `engines` (array of required engine names), `tools` (array of required MCP tool names). |

No additional properties are permitted in the TeamCard body.

### 5.5 TeamCard Example

```json
{
  "kind": "team",
  "slug": "property-management",
  "version": "1.0.0",
  "name": "Property Management Team",
  "description": "AI-powered property management with tenant communication and compliance",
  "author": {
    "name": "agentteam",
    "matrix_id": "@qi:agentteam.app"
  },
  "tags": ["property", "management"],
  "body": {
    "agents": [
      {
        "role": "tenant-agent",
        "engine": "claude-code",
        "skills": ["tenant-communication", "maintenance-triage"],
        "description": "Handles tenant inquiries and maintenance requests"
      },
      {
        "role": "landlord-agent",
        "engine": "claude-code",
        "skills": ["financial-reporting", "compliance-check"],
        "description": "Financial reporting and regulatory compliance"
      }
    ],
    "rooms": {
      "team": true,
      "custom": ["maintenance-alerts"]
    },
    "cli_command": "agentteam up --template property-management",
    "prerequisites": {
      "engines": ["claude-code"]
    }
  }
}
```

## 6. A2A Compatibility

AgentCard bodies are designed as a true superset of the [A2A Agent Card](https://google.github.io/A2A/) specification.

### 6.1 A2A Required Fields

The following A2A v1.0 required fields MUST be present in every AgentCard body (all 8 are in the schema's `required` array):

- `name` -- agent display name
- `description` -- agent description
- `version` -- agent version
- `defaultInputModes` -- at least one supported input modality
- `defaultOutputModes` -- at least one supported output modality
- `capabilities` -- agent capability declarations
- `skills` -- array of `AgentSkill` objects (each with `id`, `name`, `description`)
- `supportedInterfaces` -- at least one protocol interface with `url`, `protocolBinding`, and `protocolVersion`

### 6.2 Extension Namespace

All AgentTeam-specific extensions are placed under `x-agentteam` in the body. A2A parsers that ignore unknown fields will skip this key entirely, preserving full compatibility.

### 6.3 Registry API and A2A

The cards registry provides two views per AgentCard:

- **Full Card**: `GET /api/v1/agents/{slug}.json` -- returns the complete envelope + body.
- **A2A-only**: `GET /api/v1/agents/{slug}/a2a.json` -- returns only the `body` object, directly parseable by any A2A client.

## 7. Registry API

The card registry is a Git repository with a CI-generated static JSON API deployed to Cloudflare Pages.

### 7.1 Directory Layout

```
cards/
  agents/{slug}/card.json
  skills/{slug}/card.json
  skills/{slug}/skill.md
  skills/{slug}/README.md
  teams/{slug}/card.json
  teams/{slug}/README.md
  schema/
  api/                          (CI-generated, gitignored)
    v1/
      index.json                (all cards)
      agents/index.json
      agents/{slug}.json
      agents/{slug}/a2a.json
      skills/index.json
      skills/{slug}.json
      teams/index.json
      teams/{slug}.json
```

### 7.2 Domain Routing

Three domains point to a single Cloudflare Pages deployment with path-based routing:

| Domain | Routes to |
|--------|-----------|
| `agentcard.cc` | `api/v1/agents/` |
| `skillscard.cc` | `api/v1/skills/` |
| `teamcard.cc` | `api/v1/teams/` |

### 7.3 CI Pipeline

On every push to the `main` branch:

1. Validate all `card.json` files against the appropriate JSON Schema (envelope + body).
2. Generate the `api/` directory with index files and per-card endpoints.
3. For AgentCards, generate an additional `a2a.json` file containing only the body.
4. Deploy to Cloudflare Pages.

### 7.4 JSON Schema References

| Schema | URI |
|--------|-----|
| Card Envelope | `https://agentteam.org/schemas/card-envelope-v1.schema.json` |
| AgentCard Body | `https://agentteam.org/schemas/agentcard-body-v1.schema.json` |
| SkillsCard Body | `https://agentteam.org/schemas/skillscard-body-v1.schema.json` |
| TeamCard Body | `https://agentteam.org/schemas/teamcard-body-v1.schema.json` |

## 8. Security Considerations

### 8.1 Skill Content Injection

Skill content files (`skill.md`) are loaded into agent context at runtime. A malicious skill could attempt prompt injection. The `status` field in SkillsCard body provides a trust signal:

- **`curated`**: Published by the core team. No community risk.
- **`verified`**: Community-contributed but sandbox-tested. CI scans for suspicious patterns (e.g. "ignore previous instructions", "system prompt").
- **`community`**: Author-verified only. Agents SHOULD prompt the owner for confirmation before loading community skills.

Implementations SHOULD default to returning only `curated` and `verified` skills in search results. Community skills SHOULD require an explicit opt-in flag.

### 8.2 Agent Identity Verification

AgentCard `slug` values for agents follow the `{name}.{owner}` format. The cards registry CI pipeline SHOULD verify that the PR author controls the owner namespace (e.g. via Matrix ID verification through the Zitadel identity provider).

### 8.3 Schema Validation

All cards MUST pass JSON Schema validation before being accepted into the registry. Implementations consuming cards from external sources SHOULD validate against the published schemas before processing.
