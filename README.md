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
| `react-code-style` | Recommendations for writing React code: derived state, Effects, re-renders and memoization. |
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

`react-code-style` distils the React docs, mainly the [Escape Hatches](https://react.dev/learn/escape-hatches) chapter, and these articles; the wording and the examples are its own:

- Kent C. Dodds: [Don't Sync State. Derive It!](https://kentcdodds.com/blog/dont-sync-state-derive-it), [State Colocation will make your React app faster](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster)
- Dominik Dorfmeister: [Don't over useState](https://tkdodo.eu/blog/dont-over-use-state), [Putting props to useState](https://tkdodo.eu/blog/putting-props-to-use-state), [Deriving Client State from Server State](https://tkdodo.eu/blog/deriving-client-state-from-server-state)
- Nadia Makarevich: [React components composition: how to get it right](https://www.developerway.com/posts/components-composition-how-to-get-it-right), [React re-renders guide: everything, all at once](https://www.developerway.com/posts/react-re-renders-guide)
- Dan Abramov: [Before You memo()](https://overreacted.io/before-you-memo/)

## License

[MIT](LICENSE)
