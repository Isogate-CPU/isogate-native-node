# Isogate Native Node

This directory is the auditable Native Node provider runtime and command-line
client for Isogate. It contains the complete TypeScript source used to inspect
a host CPU, execute the bounded built-in workloads, and submit those workloads
to an Isogate-compatible verification service.

The canonical replay implementation is vendored in
`packages/isogate-replay`. It is a private internal workspace package, not an
external runtime dependency. The replay vectors in that package are protocol
fixtures: a digest or halted-state change is a compatibility change and should
be reviewed alongside every consumer.

## Requirements

- Node.js 20 or newer
- pnpm 9 or newer

Install the workspace dependencies and build:

```bash
pnpm install
pnpm build
```

The build writes generated JavaScript and declarations to `dist/`; generated
files are intentionally not part of this source export.

## Diagnose a host

```bash
node dist/cli.js diagnose
node dist/cli.js diagnose --json
node dist/cli.js diagnose --output isogate-diagnostic.json
```

The diagnostic report includes operating-system-reported CPU information,
memory totals, a real SHA-256 benchmark, and a report digest. Hostname and
uptime are used only for the local human-readable check and are not written to
the exported JSON report. Reports may contain sensitive hardware details; keep
them private unless disclosure is intentional.

## Run as a provider

Register the diagnostic report through the Isogate service, then use the
provider ID and the one-time credential returned by registration:

```bash
ISOGATE_PROVIDER_CREDENTIAL='paste-credential-here' \
  node dist/cli.js start \
  --server https://your-service.example/api \
  --provider 00000000-0000-0000-0000-000000000000
```

For an interactive terminal, omit the environment variable and the CLI will
prompt with terminal echo disabled. The credential is kept in process memory,
sent only in authenticated provider requests, and is not written to disk or
printed. Do not put a credential in shell history, source code, CI logs, or
issue reports.

The runtime accepts only the built-in `cpu_replay` and `cpu_art_rgb565`
workloads. It does not execute arbitrary code or upload host files,
environment variables, or credentials. The service URL and API contract are
deployment-specific and are not included in this repository.

## Development checks

```bash
pnpm typecheck
pnpm test
```

The test suite checks canonical replay digests, partial and halted state, and
deterministic RGB565 output.

## Release source boundary

This repository export is the reviewable source boundary for Native Node. It
does not contain npm publishing credentials, release secrets, registry tokens,
or npm publishing workflows. A release process, if adopted by a project
maintainer, must build from a reviewed checkout and supply credentials through
the maintainer's private CI or local environment. Generated `dist/` output is
not treated as source and is not committed here.