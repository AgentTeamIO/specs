# Skill Content Format Specification

**Status:** Draft v1.0
**Date:** 2026-02-21
**License:** Apache 2.0

## Abstract

This specification defines the content formats for skill files referenced by SkillsCard entries. A skill file contains the instructions and methodology that an AI agent loads into its context to perform a specific task. Three formats are supported: `claude-skill-v1` (primary), `markdown` (engine-agnostic), and `yaml-frontmatter` (structured metadata with content).

## 1. Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHOULD", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

| Term | Definition |
|------|-----------|
| **Skill file** | A Markdown file (`skill.md`) containing agent instructions for a specific task. |
| **Format** | The structure of the skill file, declared in the SkillsCard body `format` field. |
| **Engine** | An AI tool runtime that loads and executes skill instructions. |

## 2. `claude-skill-v1` Format

The primary format, designed for Claude Code and compatible engines. A skill file in this format is a pure Markdown document with a conventional heading structure. It contains no frontmatter.

### 2.1 Structure

```markdown
# {Skill Name}

{One-paragraph description of the skill and when to use it.}

## {Section Heading}

{Methodology, process steps, checklists, or reference material.}

## Output Format

{Expected output structure or template.}
```

### 2.2 Requirements

- The file MUST begin with a level-1 heading (`# {name}`).
- The file MUST contain at least one level-2 section.
- The file SHOULD include an "Output Format" section describing the expected response structure.
- The file MUST be valid Markdown (CommonMark).
- The file SHOULD NOT exceed 5,000 tokens to stay within typical context budgets.

### 2.3 Conventions

- Use imperative voice ("Review the code", not "The code should be reviewed").
- Include concrete examples where the methodology benefits from illustration.
- Structure content for progressive disclosure: summary first, details in subsections.

### 2.4 Example

```markdown
# Systematic Code Review

You are performing a structured code review. Follow this methodology
to produce actionable, severity-rated feedback.

## Review Process

1. **Understand context** -- Read the PR description and surrounding code.
2. **Review in layers** -- Multiple passes, each focused on one concern.
3. **Rate every finding** -- Assign a severity (P0-P3).
4. **Suggest, don't just criticize** -- Every problem includes a fix.

## Severity Levels

| Level | Label | Meaning |
|-------|-------|---------|
| P0 | Blocker | Security vulnerability, data loss, crash. |
| P1 | Major | Logic error, missing error handling. |
| P2 | Minor | Readability, naming, minor inefficiency. |
| P3 | Nit | Style preference. Author's discretion. |

## Output Format

For each finding:

**[P{n}]** `file:line` -- {summary}

{explanation}

Suggestion: {fix}
```

## 3. `markdown` Format

An engine-agnostic fallback. Identical to `claude-skill-v1` structurally, but explicitly signals that the content contains no engine-specific conventions and is designed to work across all engines.

### 3.1 Requirements

- All requirements from Section 2.2 apply.
- The content MUST NOT rely on engine-specific features (e.g. Claude Code tool names, Gemini-specific formatting).
- The content SHOULD use generic instruction patterns that any LLM can follow.

### 3.2 When to Use

Use `markdown` format when the skill is designed for maximum portability across engines and the content does not depend on any engine-specific capabilities.

## 4. `yaml-frontmatter` Format

A structured format that embeds machine-readable metadata directly in the skill file. Useful when skill content is consumed by both agents and automated pipelines.

### 4.1 Structure

```markdown
---
name: {skill-name}
description: {one-line description}
metadata:
  author: {author-handle}
  version: "{semver}"
  category: {category}
  triggers: "{comma-separated trigger phrases}"
---

# {Skill Name}

{Skill content in Markdown.}
```

### 4.2 Frontmatter Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | REQUIRED | Skill name (kebab-case). |
| `description` | string | REQUIRED | One-line description. |
| `metadata` | object | OPTIONAL | Additional metadata. |
| `metadata.author` | string | OPTIONAL | Author handle. |
| `metadata.version` | string | OPTIONAL | Semantic version. |
| `metadata.category` | string | OPTIONAL | Category for namespacing (e.g. `"legal"`, `"development"`). |
| `metadata.triggers` | string | OPTIONAL | Comma-separated trigger phrases for discovery. |

### 4.3 Requirements

- The frontmatter MUST be valid YAML enclosed in `---` delimiters.
- The body after the frontmatter MUST be valid Markdown (CommonMark).
- Engines that do not support YAML frontmatter MUST ignore the frontmatter block and load only the Markdown body.

### 4.4 Example

```markdown
---
name: contract-review
description: Review contracts for risks, identify problematic clauses, and suggest improvements.
metadata:
  author: agentteam
  version: "1.0"
  category: legal
  triggers: "review contract, check agreement, NDA review"
---

# Contract Review

## When to Use

Use when the user asks to review, analyze, or check any legal document.

## How to Review

1. Read the full document.
2. Identify key clauses (termination, liability, IP, payment).
3. Flag risks by severity.
4. Suggest specific improvements.
```

## 5. Engine Compatibility

### 5.1 Loading Behavior

When an engine loads a skill file, it SHOULD:

1. Check the `format` field in the SkillsCard body.
2. For `claude-skill-v1` and `markdown`: load the entire file as context.
3. For `yaml-frontmatter`: parse and strip the frontmatter, then load the Markdown body as context. Frontmatter metadata MAY be used for routing or display but MUST NOT be injected as agent instructions.

### 5.2 Engine Compatibility Matrix

| Engine | `claude-skill-v1` | `markdown` | `yaml-frontmatter` |
|--------|-------------------|------------|---------------------|
| Claude Code | Full support | Supported | Supported (strips frontmatter) |
| Gemini CLI | Supported | Supported | Supported (strips frontmatter) |
| Codex | Supported | Supported | Supported (strips frontmatter) |
| Letta | Supported | Supported | Supported |
| OpenClaw | Supported | Supported | Supported |

All engines listed SHOULD support all three formats. Engines that do not support `yaml-frontmatter` parsing MUST strip the YAML frontmatter block and load the remaining Markdown body.

## 6. Security

### 6.1 Prompt Injection Defense

Skill files are injected into agent context and represent a prompt injection surface. Defenses are layered:

1. **Curation** (Phase 0): Only core-team-authored skills. No injection risk.
2. **CI Scanning** (Phase 1): Automated pattern matching flags suspicious content (`"ignore previous"`, `"system prompt"`, `"you are now"`, etc.) during PR review.
3. **Sandbox Verification** (Phase 2): Skills run in isolated environments with test cases before earning `"verified"` status.

### 6.2 Content Size Limits

Implementations SHOULD enforce a maximum skill file size of 50 KB. Skill content SHOULD NOT exceed 5,000 tokens to stay within typical context window budgets and minimize injection surface area.
