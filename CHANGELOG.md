# Changelog

Notable changes to `@nifdi/mcp`. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the package uses
[semantic versioning](https://semver.org/spec/v2.0.0.html).

## 0.3.0

### Changed

- A saved diagram embeds the WOFF2 subsets its text actually uses, rather than whole TrueType
  faces: only the unicode ranges the diagram's codepoints fall in, one rule per distinct file, and
  one file shared across the weights that use it in a variable family. Files get smaller — the
  schematic in the README went from 147,787 to 82,821 bytes — and they still open with the exact
  faces their geometry was measured against on a machine with neither nifdi nor a network. A face
  whose web files could not be had embeds its TrueType bytes instead, so a file never names a face
  it carries no bytes for.
- Text is markdown: several paragraphs, headings, bullet and numbered lists, code blocks, bold,
  italic and inline code in one text block. That holds for `set_text` content, for `<text>` in
  `insert` and `set_diagram` XML, and for the text already in a file. `set_text` asks for the bold
  and monospace faces the content needs before it edits, so a heading or a code span refuses up
  front rather than failing mid-edit.
- Existing text is read as markdown with no escaping, so a diagram saved by an earlier version whose
  text holds `*`, `_` or backticks, or a line starting `#`, `- ` or `1. `, draws differently when
  opened.
- Multi-line text is saved with `xml:space="preserve"` and read back exactly. A `<text>` without it
  is trimmed and its continuation lines dedented, so hand-indented XML reads as the text it means.
- Fira Mono is a canonical family: code spans and code blocks are drawn in it, and startup and
  `fetch-assets --fonts` download it beside the others.
- `fetch-assets --fonts` takes family names: `nifdi-mcp fetch-assets --fonts "DM Sans" Cardo`
  (or `"DM Sans,Cardo"`) puts any Google Fonts family into the store at every weight it advertises,
  which is how a machine that will run with `NIFDI_OFFLINE=true` is given a family beyond the
  canonical set. A name that is not a Google Fonts family is listed as not fetched and the command
  exits non-zero; faces left without web files are listed too.
- `render`, `textconv` and `merge` install any missing canonical faces before they run, as the
  server does at startup. A git diff or merge driver can therefore reach the network on first use,
  unless `NIFDI_OFFLINE=true`, and reports missing assets on stderr.
- Opening a diagram repairs geometry that cannot be resized: fractional coordinates move onto whole
  pixels, and fractional border widths and margins round up. Centring puts an odd pixel on the high
  side everywhere. A file saved by an earlier version can therefore shift by a pixel when opened and
  saved.
- A file whose icon colour is anything but `natural` or a hex colour, or whose font family is not a
  valid family name, is refused on open rather than drawn.
- `connect` takes explicit connector pairs instead of a list of block ids. `connections` is a list of
  `{from, to}` pairs, and each pair becomes one connector pointing from `from` to `to`. An end is
  `{block}`, which leaves routing to choose the side and choose again when the block moves, or
  `{block, attach: "left" | "right" | "top" | "bottom"}`, which pins that side. These are the same
  two kinds of end a `<path>` carries in `get_diagram` (`current=` and `attach=`). A pair is always
  added, even between blocks that are already joined, and the summary says when that happened.
- `connect` takes style. `style` sets path declarations such as
  `path-colour: #64748b; line-style: dashed` on every connector the call creates. A connection's own
  `style` sets them on that connector only and wins where the two overlap. Both use `set_style`'s
  path vocabulary, report skipped declarations the same way, and keep their values on the connectors
  they name. When either is given, the result carries `applied` and `skips` as `set_style`'s does.
- A new connector with no style of its own takes the look the diagram's connectors share, the
  `path {}` rule in `get_diagram`. Before, every connector came out black with a triangle arrowhead,
  whatever that rule said.
- `pathIds` in the `connect` result lines up with `connections`: one id per pair, or `null` for a
  pair that could not be routed, which the summary names.

### Removed

- Cloud and workspace modes. The server no longer reads `NIFDI_BRIDGE_URL`, `NIFDI_AGENT_TOKEN`,
  `NIFDI_WORKSPACE_TOKEN` or `NIFDI_MCP_MODE`, and the agent tokens those modes took are no longer
  issued. `@nifdi/mcp` works on local `.nifdi.svg` files only. To work on diagrams stored in the
  nifdi cloud, add the hosted server instead, which signs in with OAuth:
  `claude mcp add --transport http nifdi https://nifdi.app/api/mcp`.
- The `add_font` tool. A font is no longer something an agent supplies to a server: every face a
  diagram names comes from the per-user asset store that `nifdi-mcp fetch-assets --fonts` fills, and
  a face that is not there is refused by name, pointing at that command, rather than being drawn in
  Inter. The `.nifdi/fonts` sidecar a supplied font was kept in is gone with it, so a workspace
  holds history and nothing else.
- `connect`'s `ids` and `mode` parameters: the `"auto"` ranking into rows and columns, and
  `"chain"`. To get a chain or a fan-out, list its pairs.
- The `new-line-behaviour` style property, with the one-block-per-line text model it governed.
  Files whose `<style>` names it still load, and the declaration is skipped with a warning.

## 0.2.0

### Changed

- Every tool that works on a diagram now requires `diagramId`, on every call. There is no current
  diagram: not the last one created, not the only file in the folder, not one pinned with
  `NIFDI_DIAGRAM_ID`. A call that names none is rejected at the schema and touches nothing. The id
  is the one `create_diagram` returns or `list_diagrams` reports, and a pasted link to the diagram's
  editor page — `/diagram/local-workspace/<path>` locally, `/diagram/nifdi-cloud/<id>` in the cloud —
  is accepted as a spelling of it.
- `normalize_diagram` takes the diagram whose asset store its `#asset-…` references resolve against.
- `create_diagram` says what it returns and how long the id lasts; `list_diagrams` is offered on
  every deployment, including a server bound to one diagram, as the way to find an id again.
- `NIFDI_DIAGRAM_ID` on a cloud-mode server still binds it to that one diagram; a call naming another
  id is refused with the binding's name. On local and workspace servers it set only the default and
  now does nothing.
- A diagram's id in local mode is now its file path relative to the workspace root, so diagrams
  can live in sub-folders: `design/architecture/system.nifdi.svg` is a first-class id for
  `list_diagrams`, `editor_url`, every tool that takes a `diagramId` and the browser editor. Ids
  are POSIX paths with forward slashes and never absolute; `..`, empty and `.` segments, backslashes and symlinks that lead out of the workspace
  are refused with a message that says which rule was broken. A plain `.svg` id is still accepted
  directly in the root, as before.
- `list_diagrams` finds `*.nifdi.svg` files at any depth below the workspace root, and plain
  `*.svg` files directly in the root, never entering hidden folders, `node_modules` or symlinks.
  Nested plain `.svg` files are not diagrams: a project's icons and exported pictures are SVG too.
- `editor_url` opens the diagram before it returns a link, so an id the workspace cannot open —
  malformed, unsafe or naming a file that is not there — fails in the tool with that reason,
  where before it returned a plausible URL and the browser tab said only that the diagram could
  not be opened.
- A diagram whose file is moved, renamed or deleted outside the server drops out of
  `list_diagrams` on the next call, even while it is open; an edit through the old id refuses to
  recreate the file at its old path and names what happened. The new path is listed as soon as it
  exists. A call naming the old id fails naming that file rather than landing in another diagram.
  A browser tab viewing the diagram is told that live editing stopped and why.
- History sidecars for every diagram live under the workspace's one `.nifdi/` folder, mirroring
  the diagram's path, so a nested diagram never seeds a second `.nifdi/` in its sub-folder.

### Removed

- `select_diagram`, and the `.nifdi/active-diagram` record beside the diagrams. A file left by an
  older server is ignored and can be deleted.

### Fixed

- The server announces its real version in the MCP handshake — the number in `package.json`, inlined
  at build time — where it had said `1.0.0` since the first commit.
- The browser editor reports why a local diagram could not be opened — the server's own message
  and status — instead of a generic "could not open" for every failure, and its access-denied
  message now says to open the link `editor_url` returns.

## 0.1.3

### Changed

- Cleaned up the README.

## 0.1.2

### Changed

- The server no longer stops itself after a spell without tool calls. A quiet session is still a
  session, and no MCP client restarts a stdio server that exits under it, so an idle timeout cost
  a working connection and offered nothing back. It now shuts down only on evidence its client has
  gone: a termination signal, stdin reaching end-of-file, stdout no longer being writable, the
  launching process dying, or a client that answers none of two liveness probes five minutes apart.
  Every shutdown names its reason on stderr.

### Removed

- `NIFDI_IDLE_TIMEOUT`, which set that timeout. It was never documented; a value set for it is now
  ignored rather than rejected.

### Fixed

- A server whose client vanished without closing the pipe — a shape seen behind launcher shims —
  could hold its editor port indefinitely. Each server now records itself in `~/.nifdi/run`, and a
  starting server stops the siblings whose own client is gone, says which and why on stderr, and
  waits for them to exit before claiming a port. A pid is only ever signalled while it is still
  running the command recorded for it, so a reused pid is never mistaken for one of these servers.

## 0.1.1

First public release.
