# OnboardJS Skills for Claude Code

A collection of AI agent skills for building user onboarding experiences. Built for developers who want Claude Code (or similar AI coding assistants) to help with onboarding flows, multi-step wizards, guided tours, and user activation in React and Next.js applications.

Built by [OnboardJS](https://onboardjs.com). OnboardJS is a headless library for building user onboarding experiences - you control the UI, OnboardJS handles flow logic, state, persistence, and navigation.

**Contributions welcome!** Found a way to improve a skill or have a new one to add? [Open a PR](#contributing).

## What are Skills?

Skills are markdown files that give AI agents specialized knowledge and workflows for specific tasks. When you add these to your project, Claude Code can recognize when you're working on an onboarding task and apply the right patterns and best practices.

## Available Skills

<!-- SKILLS:START -->

| Skill                                      | Description                                                                                                                                                                                     |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [onboardjs-react](skills/onboardjs-react/) | Integrate OnboardJS into React projects for building user onboarding flows. Use when setting up OnboardJS in a React/Next.js project, creating multi-step onboarding, wizards, or guided flows. |

<!-- SKILLS:END -->

## Installation

### Option 1: CLI Install (Recommended)

Use [npx skills](https://github.com/vercel-labs/skills) to install skills directly:

```bash
# Install all skills
npx skills add OnboardJS/onboardjs-skills

# Install specific skills
npx skills add OnboardJS/onboardjs-skills --skill onboardjs-react

# List available skills
npx skills add OnboardJS/onboardjs-skills --list
```

You may also use `pnpm`.

```bash
pnpm dlx skills add OnboardJS/onboardjs-skills
```

This automatically installs to your `.claude/skills/` directory.

### Option 2: Claude Code Plugin

Install via Claude Code's built-in plugin system:

```bash
# Add the marketplace
/plugin marketplace add OnboardJS/onboardjs-skills

# Install all OnboardJS skills
/plugin install onboardjs
```

### Option 3: Clone and Copy

Clone the entire repo and copy the skills folder:

```bash
git clone https://github.com/OnboardJS/onboardjs-skills.git
cp -r onboardjs-skills/skills/* .claude/skills/
```

### Option 4: Git Submodule

Add as a submodule for easy updates:

```bash
git submodule add https://github.com/OnboardJS/onboardjs-skills.git .claude/onboardjs-skills
```

Then reference skills from `.claude/onboardjs-skills/skills/`.

### Option 5: Fork and Customize

1. Fork this repository
2. Customize skills for your specific needs
3. Clone your fork into your projects

### Option 6: SkillKit (Multi-Agent)

Use [SkillKit](https://github.com/rohitg00/skillkit) to install skills across multiple AI agents (Claude Code, Cursor, Copilot, etc.):

```bash
# Install all skills
npx skillkit install OnboardJS/onboardjs-skills

# Install specific skills
npx skillkit install OnboardJS/onboardjs-skills --skill onboardjs-react

# List available skills
npx skillkit install OnboardJS/onboardjs-skills --list
```

## Usage

Once installed, just ask Claude Code to help with onboarding tasks:

```
"Add user onboarding to my React app"
→ Uses onboardjs-react skill

"Create a multi-step wizard flow"
→ Uses onboardjs-react skill

"Set up OnboardJS with Next.js App Router"
→ Uses onboardjs-react skill

"Add step persistence to my onboarding"
→ Uses onboardjs-react skill
```

You can also invoke skills directly:

```
/onboardjs-react
```

## What the Skills Cover

### onboardjs-react

- **Setup** — Package detection, installation, React vs Next.js configuration
- **Provider Pattern** — OnboardingProvider setup with steps, callbacks, persistence
- **Step Components** — Building steps with StepComponentProps, payload typing, data handling
- **Navigation** — Conditional navigation, dynamic step routing, skip logic
- **Persistence** — localStorage, custom backend (Supabase, REST API, Server Actions)
- **Validation** — Client-side with onDataChange, server-side with onStepComplete
- **Next.js Patterns** — 'use client' directives, dynamic imports, App Router, Pages Router
- **Advanced** — Event system, plugins, TypeScript patterns, analytics integration

## Planned Skills

- `onboardjs-vue` — Vue.js integration
- `onboardjs-svelte` — Svelte integration
- `wizard-analytics` — Onboarding analytics and funnel tracking
- `product-tours` — Guided product tours and tooltips

## Contributing

Found a way to improve a skill? Have a new skill to suggest? PRs and issues welcome!

### Adding a New Skill

1. Create a directory under `skills/` with your skill name (lowercase, hyphens only)
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`)
3. Keep the main file under 500 lines — use `references/` for detailed docs
4. Run `node .github/scripts/sync-skills.js` to update marketplace.json and README

See [AGENTS.md](AGENTS.md) for full guidelines on skill structure and writing style.

## License

[MIT](LICENSE) — Use these however you want.
