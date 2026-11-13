# claude-toolbelt

A collection of custom skills for Claude.

## Skills

| Skill | Description |
|-------|-------------|
| [learning-roadmap](skills/learning-roadmap/SKILL.md) | Prerequisite-first learning coach: maps the building-block concepts behind a topic, quizzes you on them to find real gaps, then teaches from the lowest gap upward. |
| [project-launchpad](skills/project-launchpad/SKILL.md) | Turns a software idea into a ready-to-build GitHub repo: scaffold, CI with GitHub Actions, CD pipeline, repo hygiene, an MVP issue backlog, and an optional daily Claude/Gemini agent that opens issues and PRs. |

## Installing a skill

Each skill lives in its own folder under `skills/` with a `SKILL.md` file.

- **Claude.ai:** zip the skill folder (e.g. `skills/learning-roadmap`) and upload it under Settings → Capabilities → Skills.
- **Claude Code:** copy the skill folder into `~/.claude/skills/` (personal) or `.claude/skills/` (project).
