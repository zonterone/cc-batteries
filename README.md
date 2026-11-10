# cc-batteries

A collection of Claude Code skills, packaged as a plugin marketplace. Each skill is its own plugin, so you install only the ones you want.

## Install

Add the marketplace once:

```sh
claude plugin marketplace add zonterone/cc-batteries
```

Then install the bundle:

```sh
claude plugin install ts-coding@zonterone-cc-batteries
```

or any single plugin:

```sh
claude plugin install ts-pattern@zonterone-cc-batteries
```

Skills load under the `zonterone-skills` namespace, for example `zonterone-skills:ts-pattern`.

## Plugins

| Plugin | What it does |
| --- | --- |
| `ts-coding` | Bundle. Installs `ts-code-style`, `karpathy-guidelines` and `deep-code-review`. |
| `ts-code-style` | Opinionated TypeScript/JavaScript code style guide. |
| `karpathy-guidelines` | Behavioral guidelines that reduce common LLM coding mistakes. |
| `deep-code-review` | Code review by three parallel reviewer subagents, every finding verified against the code. |
| `neverthrow` | Project convention for error handling with [neverthrow](https://github.com/supermacro/neverthrow). |
| `ts-pattern` | Project convention for pattern matching with [ts-pattern](https://github.com/gvergnaud/ts-pattern). |

`neverthrow` and `ts-pattern` state a convention for the project they are enabled in. Install them at project scope, in projects that use the library:

```sh
claude plugin install neverthrow@zonterone-cc-batteries --scope project
```

## Updates

Plugins carry no version number, so every commit on `main` is a new version. Claude Code does not auto-update third-party marketplaces by default. Update by hand:

```sh
claude plugin marketplace update zonterone-cc-batteries
claude plugin update ts-code-style@zonterone-cc-batteries
```

or turn on auto-update for the marketplace in `/plugin`.

## Credits

`karpathy-guidelines` is adapted from [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) by forrestchang, MIT licensed. The skill description is changed; the body is unmodified.

## License

[MIT](LICENSE)
