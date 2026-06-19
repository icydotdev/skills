# @icydotdev/skills

Agent skills for the [@icydotdev](https://github.com/icydotdev) suite, installable
with the [`skills`](https://skills.sh) CLI.

## Install

```bash
# install a specific skill (prompts for project vs global)
npx skills add icydotdev/skills --skill react-design-system-extractor

# or browse everything in this repo
npx skills add icydotdev/skills
```

Then, in Claude Code (or any supported agent), trigger it:

```
/react-design-system-extractor
```

…or just ask: *"extract the design system from this codebase."*

## Skills

| Skill | What it does | Engine |
| ----- | ------------ | ------ |
| [`react-design-system-extractor`](skills/react-design-system-extractor) | Extracts the design system hiding in an existing React/Next.js codebase and scaffolds a structured `ui/` folder (unified components, tokens, Storybook stories, a11y + unit tests) while the live [Lumen](https://github.com/icydotdev/lumen) dashboard visualises progress. | [`@icydotdev/lumen`](https://github.com/icydotdev/lumen) |

## How these skills work

A skill is just instructions (`SKILL.md`) the agent follows. Heavy lifting lives
in standalone npm engines the skill drives via `npx` — so installing a skill pulls
only the markdown, and the engine is fetched on demand when you run it.

## License

MIT © Sam Kavanagh
