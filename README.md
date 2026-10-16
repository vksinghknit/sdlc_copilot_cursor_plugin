# SDLC Copilot — Cursor Plugin

Cursor port of [sdlc-copilot-plugin](https://github.com/vksinghknit/sdlc-copilot-plugin). Walks you from a raw idea to a production deployment one phase at a time; each phase adopts the right persona, produces an artifact, and waits for your explicit sign-off.

## Layout

```
.cursor-plugin/marketplace.json
plugins/sdlc-copilot/
  .cursor-plugin/plugin.json
  skills/      11 phase skills (SKILL.md, same as the Claude version)
  commands/    one slash command per phase (/requirements-elicitation, /system-design, ...)
  rules/       sdlc-copilot-workflow.mdc (phase map + sign-off rules)
  reference/   shared framework + per-phase specs
```

## Install

- **Team marketplace / repo import:** push this folder to GitHub and add the repo as a plugin source in Cursor (Settings → Plugins).
- **Local try-out:** symlink the plugin folder into Cursor's local plugins directory:

```bash
mkdir -p ~/.cursor/plugins/local && ln -s "$(pwd)/plugins/sdlc-copilot" ~/.cursor/plugins/local/sdlc-copilot
```

Restart Cursor, then type `/requirements-elicitation` in Agent chat, or just describe your idea.

## Differences from the Claude version

- Manifest is `.cursor-plugin/plugin.json` instead of `.claude-plugin/`.
- Added `commands/` and a `rules/` file for Cursor.
- UX skill saves the wireframe as an HTML file instead of publishing a Claude Artifact.
- Skill "platform note" rows now describe the Cursor realization.

MIT — see `LICENSE`.
