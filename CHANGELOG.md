# Changelog

Notable changes to `@nifdi/mcp`. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the package uses
[semantic versioning](https://semver.org/spec/v2.0.0.html).

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
