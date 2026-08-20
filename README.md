# @nifdi/mcp

[nifdi](https://nifdi.app) diagrams as code, for AI agents: a local, file-backed MCP diagram
server with git tooling and an optional live browser preview.

**Status: pre-release.** The package is not yet on npm — the commands below describe the
first release.

```sh
claude mcp add nifdi-diagram -s project -- npx -y @nifdi/mcp@latest
```

The agent authors diagrams as `.svg` files in your own repository — real SVGs that render
anywhere, carrying the editable model alongside the picture, so they diff and merge in git
like source rather than like binaries.

## About this repository

This is the home of the published npm package, **not its source**. `@nifdi/mcp` is
proprietary software; this repo exists so that the package has a public issue tracker, a
public changelog, and a licence you can read without unpacking a tarball.

- **Bugs and feature requests** — [Issues](../../issues)
- **Full documentation** — the package README on [npm](https://www.npmjs.com/package/@nifdi/mcp)
- **Release history** — [CHANGELOG.md](CHANGELOG.md)
- **Terms** — [LICENSE.md](LICENSE.md)

## The skills are open source

The Agent Skills that teach an agent to use this server are Apache-2.0 and take
contributions: [nifdi-app/nifdi-plugin](https://github.com/nifdi-app/nifdi-plugin).
Installing the plugin brings the server with it:

```sh
claude plugin marketplace add nifdi-app/nifdi-plugin
claude plugin install nifdi@nifdi-plugin
```

The plugin repo's licence covers the skills only. This server is licensed separately, on the
terms below.

## Licence

`@nifdi/mcp` is proprietary software, © Nifdi Systems Limited — see [LICENSE.md](LICENSE.md).
It is **free to use**, including commercially, in CI, and by AI agents acting on your behalf,
and unmodified copies may be redistributed as a dependency or inside a larger product. It may
not be modified or reverse engineered. Your own application or service stays licensed on
whatever terms you choose, including an open source licence.
