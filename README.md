# AgentTeam Specifications

Open specifications for the [AgentTeam](https://agentteam.app) ecosystem. Published at [agentteam.org](https://agentteam.org).

## Specifications

| Document | Status | Description |
|----------|--------|-------------|
| [agent-card-spec.md](./agent-card-spec.md) | Draft v1.0 | Card envelope, AgentCard body (A2A superset), SkillsCard body, TeamCard body, and registry API |
| [skill-format.md](./skill-format.md) | Draft v1.0 | Skill content file formats: `claude-skill-v1`, `markdown`, and `yaml-frontmatter` |
| [team-template.md](./team-template.md) | Draft v1.0 | Team template format: agent roles, room configuration, prerequisites, and CLI deployment |
| [atp-protocol.md](./atp-protocol.md) | Draft v1.0 | AgentTeam Protocol overview: Matrix + NATS + MCP + A2A layer stack |

## Related Repositories

| Repository | Purpose |
|------------|---------|
| [agentteam/cards](https://github.com/agentteam/cards) | Card registry -- JSON entries validated by these specs |
| [agentteam/agentteam](https://github.com/agentteam/agentteam) | AgentTeam core platform (v3 Rust) |
| [agentteam/toolcase](https://github.com/agentteam/toolcase) | Curated MCP Server collection |

## Contributing

Contributions are welcome via pull request. Specification changes follow a lightweight RFC process:

1. Open an issue describing the proposed change and its motivation.
2. Submit a PR with the spec update. Use RFC 2119 keywords (`MUST`, `SHOULD`, `MAY`) for normative requirements.
3. Changes are reviewed by maintainers and merged after consensus.

## License

[Apache License 2.0](./LICENSE) -- Copyright 2026 AgentTeam
