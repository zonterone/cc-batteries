# cc-batteries

A Claude Code plugin marketplace. `.claude-plugin/marketplace.json` lists every plugin.

## Two kinds of plugin

- **Skill plugin**: `skills/<name>/` holds `SKILL.md` at its root and `.claude-plugin/plugin.json`. One skill per plugin, so each installs on its own.
- **Composite plugin**: `plugins/<name>/` with the standard plugin layout. Use it for anything with hooks, agents, MCP servers or a mod, and for bundles.

## Conventions

- Every skill plugin's manifest `name` is `zonterone-skills`. The manifest name is the namespace, so skills load as `zonterone-skills:<skill>`. The marketplace entry name is the install key and equals the folder name; `displayName` repeats it for the `/plugin` UI.
- A composite plugin takes its own unique manifest `name`. Claude Code registers hooks for one plugin per manifest name.
- Leave `version` out of every manifest and entry. The commit SHA is the version, and each push to `main` reaches users as an update; a written `version` pins users until it is bumped.
- Display fields (`description`, `author`, `repository`, `license`) live in `plugin.json`; a skill adapted from an upstream also carries `homepage` pointing at it. A marketplace entry holds `name` and `source` only.
- A bundle is a manifest with `dependencies` and no components. `dependencies` lists marketplace entry names.

## Adding a skill plugin

1. Create `skills/<name>/SKILL.md` and `skills/<name>/.claude-plugin/plugin.json`, copying the manifest shape from a sibling skill.
2. Add `{ "name": "<name>", "source": "./skills/<name>" }` to `.claude-plugin/marketplace.json`.
3. Add a row to the Plugins table in `README.md`.
4. Run `claude plugin validate .` and fix every error. One `No version specified` warning per plugin is expected.
