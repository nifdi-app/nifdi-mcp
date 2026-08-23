# Changelog

Notable changes to `@nifdi/mcp`. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the package uses
[semantic versioning](https://semver.org/spec/v2.0.0.html).

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
