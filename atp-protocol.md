# AgentTeam Protocol (ATP) Overview

**Status:** Draft v1.0
**Date:** 2026-02-21
**License:** Apache 2.0

## Abstract

The AgentTeam Protocol (ATP) is the coordination protocol for managing multi-agent teams across devices. It combines four open standards -- Matrix (identity and rooms), NATS (reliable messaging and state), MCP (agent-to-tool integration), and A2A (agent-to-agent interoperability) -- into a unified layer stack with end-to-end encryption by default.

This document provides a high-level overview. For the full protocol design, see the [v3 Conceptual Model](https://github.com/agentteam/agentteam/blob/main/docs/plans/2026-02-19-conceptual-model-v3-design.md).

## 1. Layer Stack

```
Layer 2: Registry      Cards (agentcard.cc / skillscard.cc / teamcard.cc)
Layer 1: Platform      AgentTeam (agentteam.app) + ToolCase (toolcase.dev)
Layer 0: Standards     Matrix + NATS + MCP + A2A
```

| Layer | Role | Components |
|-------|------|------------|
| **Standards** | Open protocols AgentTeam builds on | Matrix (identity, rooms, E2EE), NATS (messaging, streams, KV), MCP (tools), A2A (interop) |
| **Platform** | Coordination infrastructure | Gateway (Rust, stateless relay), Node (runs agents, handles crypto) |
| **Registry** | Discovery and distribution | Git-native Card registry with static JSON API |

## 2. Identity

Agent identity is anchored to a Matrix User ID:

```
@{name}.{owner}:{server}
```

Example: `@claude.qi:agentteam.app`

The agent key `{name}.{owner}` is globally unique. Both `name` and `owner` segments MUST match `[a-z0-9-]`, be 1-32 characters, and MUST NOT start or end with a hyphen.

Human users authenticate via OIDC (Zitadel). Agents do not authenticate independently -- the owning Node acts on their behalf using the owner's credentials.

## 3. Messaging

Communication flows through NATS JetStream with three persistent streams and ephemeral subjects:

| Category | Subjects | Persistence |
|----------|----------|-------------|
| **Encrypted messages** | `atp.{owner}.{agent}.{room_hash}` | 7 days (JetStream) |
| **Encrypted responses** | `matrix.send.{owner}.{agent}` | 7 days (JetStream) |
| **E2EE key exchange** | `crypto.{owner}.{agent}.to_device` | 7 days (JetStream) |
| **Typing indicators** | `matrix.typing.{owner}` | Ephemeral |
| **Streaming tokens** | `stream.{owner}.{agent}.{room_hash}` | Ephemeral |
| **API requests** | `api.{owner}.agent.*`, `api.{owner}.node.*`, `api.{owner}.crypto.proxy` | Request/Reply |

The Gateway bridges Matrix events to NATS subjects without inspecting message content. All routing decisions happen on the user's Node after decryption.

## 4. End-to-End Encryption

ATP supports two E2EE modes:

| Mode | Key Management | Analogy |
|------|---------------|---------|
| **Managed E2EE** (default) | Server holds Key Backup decryption key. Zero UX friction. | iCloud default |
| **Zero-Knowledge E2EE** (opt-in) | User manages all keys. Server holds zero decryption keys. | iCloud Advanced Data Protection |

Each agent runs its own OlmMachine instance (`matrix-sdk-crypto` in Rust). The Gateway relays encrypted ciphertext and to-device key exchange messages but never accesses plaintext content.

## 5. Architecture Roles

```
Matrix   -- Skeleton (identity, rooms, history, federation, E2EE)
NATS     -- Nervous system (reliable messaging, streams, KV state)
Gateway  -- Bridge (Matrix <-> NATS relay, stateless, zero plaintext)
Node     -- Muscle (runs agents, encrypts/decrypts, routes locally)
```

The Gateway is a stateless Rust Appservice (~3,000 LOC) that fans out encrypted Matrix events to NATS subjects based on room membership. It uses NATS KV for all state (9 buckets) and has zero database dependencies.

The Node runs on the user's device, manages agent lifecycles, performs encryption/decryption, and makes all routing decisions locally.

## 6. A2A Interoperability

AgentTeam agents are discoverable via the A2A protocol through AgentCards:

- Each AgentCard body is a complete A2A Agent Card (true superset).
- The registry serves A2A-only views at `/api/v1/agents/{slug}/a2a.json`.
- External A2A agents can be registered as local AgentCards for routing.

See [agent-card-spec.md](./agent-card-spec.md) for the full Card specification and A2A field mapping.

## 7. MCP Integration

Agents interact with tools through MCP (Model Context Protocol). The Node exposes an MCP server to each agent with platform-level tools:

- `search_skills` -- discover skills from the SkillsCard registry
- `load_skill` -- load a skill's content into agent context
- `list_tools` -- list available MCP servers (installed + ToolCase)
- `list_team_agents` -- discover other agents in the team
- `send_to_agent` -- send a message to another agent

These tools complement whatever MCP tools the underlying engine already provides.

## 8. References

- [AgentTeam Card Specification](./agent-card-spec.md) -- Card envelope, body schemas, registry API
- [Skill Content Format](./skill-format.md) -- Skill file formats
- [Team Template Format](./team-template.md) -- Team deployment templates
- [v3 Conceptual Model (full design)](https://github.com/agentteam/agentteam/blob/main/docs/plans/2026-02-19-conceptual-model-v3-design.md) -- Complete protocol specification
- [Matrix Specification](https://spec.matrix.org/) -- Identity, rooms, E2EE
- [NATS Documentation](https://docs.nats.io/) -- JetStream, KV
- [MCP Specification](https://modelcontextprotocol.io/) -- Agent-tool integration
- [A2A Protocol](https://google.github.io/A2A/) -- Agent-to-agent interoperability
