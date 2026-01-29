# AGENTS.md

Guidelines for AI agents working in this repository.

## Repository Overview

This repository contains **Agent Skills** for AI agents following the [Agent Skills specification](https://agentskills.io/specification.md). It also serves as a **Claude Code plugin marketplace** via `.claude-plugin/marketplace.json`.

- **Name**: OnboardJS Skills
- **GitHub**: [OnboardJS/onboardjs-skills](https://github.com/OnboardJS/onboardjs-skills)
- **Creator**: OnboardJS
- **License**: MIT

## Repository Structure

```
onboardjs/
├── .claude-plugin/
│   └── marketplace.json   # Claude Code plugin marketplace manifest
├── skills/
│   └── onboardjs-react/
│       ├── SKILL.md       # Main skill file
│       └── references/    # Detailed API documentation
│           ├── hooks-api.md
│           ├── step-config.md
│           └── patterns.md
├── AGENTS.md
├── CLAUDE.md
├── VERSIONS.md
└── LICENSE
```

## Build / Lint / Test Commands

**Not applicable** - This is a content-only repository with no executable code.

Verify manually:
- YAML frontmatter is valid
- `name` field matches directory name exactly
- `name` is 1-64 chars, lowercase alphanumeric and hyphens only
- `description` is 1-1024 characters

## Agent Skills Specification

Skills follow the [Agent Skills spec](https://agentskills.io/specification.md).

### Required Frontmatter

```yaml
---
name: skill-name
description: What this skill does and when to use it. Include trigger phrases.
---
```

### Frontmatter Field Constraints

| Field         | Required | Constraints                                                      |
|---------------|----------|------------------------------------------------------------------|
| `name`        | Yes      | 1-64 chars, lowercase `a-z`, numbers, hyphens. Must match dir.   |
| `description` | Yes      | 1-1024 chars. Describe what it does and when to use it.          |
| `license`     | No       | License name (default: MIT)                                      |
| `metadata`    | No       | Key-value pairs (author, version, etc.)                          |

### Name Field Rules

- Lowercase letters, numbers, and hyphens only
- Cannot start or end with hyphen
- No consecutive hyphens (`--`)
- Must match parent directory name exactly

**Valid**: `onboardjs-react`, `onboardjs-vue`, `wizard-flow`
**Invalid**: `OnboardJS-React`, `-onboardjs`, `onboard--js`

### Skill Directory Structure

```
skills/skill-name/
├── SKILL.md        # Required - main instructions (<500 lines)
├── references/     # Optional - detailed docs loaded on demand
├── scripts/        # Optional - executable code
└── assets/         # Optional - templates, data files
```

## Writing Style Guidelines

### Structure

- Keep `SKILL.md` under 500 lines (move details to `references/`)
- Use H2 (`##`) for main sections, H3 (`###`) for subsections
- Use bullet points and numbered lists liberally
- Short paragraphs (2-4 sentences max)

### Tone

- Direct and instructional
- Second person ("You are helping integrate OnboardJS...")
- Professional but approachable

### Formatting

- Bold (`**text**`) for key terms
- Code blocks for examples and templates
- Tables for reference data
- No excessive emojis

### Clarity Principles

- Clarity over cleverness
- Specific over vague
- Active voice over passive
- One idea per section

### Description Field Best Practices

The `description` is critical for skill discovery. Include:
1. What the skill does
2. When to use it (trigger phrases)
3. Related skills for scope boundaries

## Claude Code Plugin

This repo serves as a plugin marketplace. The manifest at `.claude-plugin/marketplace.json` lists all skills for installation.

See [Claude Code plugins documentation](https://code.claude.com/docs/en/plugins.md) for details.

## Git Workflow

### Branch Naming

- New skills: `feature/skill-name`
- Improvements: `fix/skill-name-description`
- Documentation: `docs/description`

### Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

- `feat: add onboardjs-vue skill`
- `fix: improve Next.js examples in onboardjs-react`
- `docs: update README`

### Pull Request Checklist

- [ ] `name` matches directory name exactly
- [ ] `name` follows naming rules (lowercase, hyphens, no `--`)
- [ ] `description` is 1-1024 chars with trigger phrases
- [ ] `SKILL.md` is under 500 lines
- [ ] No sensitive data or credentials

## Checking for Updates

When using any skill from this repository:

1. **Once per session**, on first skill use, check for updates:
   - Fetch `VERSIONS.md` from GitHub: https://raw.githubusercontent.com/OnboardJS/onboardjs-skills/main/VERSIONS.md
   - Compare versions against local skill files

2. **Only prompt if meaningful**:
   - 2 or more skills have updates, OR
   - Any skill has a major version bump (e.g., 1.x to 2.x)

3. **Non-blocking notification** at end of response:
   ```
   ---
   Skills update available: X OnboardJS skills have updates.
   Say "update skills" to update automatically, or run `git pull` in your onboardjs folder.
   ```

4. **If user says "update skills"**:
   - Run `git pull` in the onboardjs directory
   - Confirm what was updated

## Skill Summary

| Skill | Purpose |
|-------|---------|
| `onboardjs-react` | Integrate OnboardJS into React/Next.js projects for user onboarding flows |

## Key Concepts (onboardjs-react)

The skill teaches agents how to:
1. Detect package manager from lock files and use correct install commands
2. Distinguish React vs Next.js projects (App Router vs Pages Router)
3. Apply `'use client'` directives appropriately for Next.js
4. Configure `OnboardingProvider` with steps, persistence, and callbacks
5. Build step components using `StepComponentProps` pattern
6. Implement conditional navigation, step validation, and backend persistence
