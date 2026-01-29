# OnboardJS Skills Versions

Current versions of all skills. Agents can compare against local versions to check for updates.

| Skill | Version | Last Updated |
|-------|---------|--------------|
| onboardjs-react | 1.0.1 | 2026-01-29 |

## Recent Changes

### 2026-01-29 (v1.0.1)
- Added Step Indicator / Progress section to SKILL.md
- Clarified that `state.steps` does NOT exist on EngineState
- Documented recommended pattern: use STEP_IDS array + currentStep.id
- Added warning to hooks-api.md about missing steps property

### 2026-01-29 (v1.0.0)
- Initial release of `onboardjs-react` skill
- React/Next.js integration with OnboardingProvider, useOnboarding hook
- References: hooks-api.md, step-config.md, patterns.md
- Covers persistence (localStorage, custom backend), conditional navigation, validation
