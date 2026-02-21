# Specs -- AgentTeam Open Standards

## What This Is
Normative specifications for the AgentTeam ecosystem, published at agentteam.org.

## Specifications
- agent-card-spec.md -- Card envelope + AgentCard body (A2A superset)
- skill-format.md -- SkillsCard content format (claude-skill-v1 + markdown)
- team-template.md -- TeamCard template format
- atp-protocol.md -- AgentTeam Protocol overview (Matrix + NATS + MCP + A2A)

## Key Decisions
- AgentCard body IS a complete A2A Agent Card (true superset, not just compatible)
- Card envelope shared across all 3 types (kind discriminator)
- Extensions use x-agentteam prefix (OpenAPI convention)
- Git-native registry (no server, static JSON API)
