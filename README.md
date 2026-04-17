# radroots_cli

`radroots_cli` provides the `radroots` command-line interface for local-first
food trade workflows on Nostr.

The current CLI exposes a human-first workflow layer for setup, farm authoring,
market browsing, selling, and ordering while preserving the lower-level
machine-first command families for automation and troubleshooting. The
authoritative command surface lives in `src/cli.rs`, and the current help
contract is covered by `tests/help.rs`.

## Workflow Commands

Start here:

- `setup [seller|buyer|both]`
- `status`

Sell from your farm:

- `farm init|set|check|show|publish`
- `sell add|show|check|publish|update|pause|reprice|restock`

Buy from the market:

- `market update|search|view`
- `order create|view|list|submit|watch|cancel|history`

`order cancel` is currently a truthful availability surface: it explains why
durable cancel remains unavailable until the local order read plane persists
the required trade-chain references.

Accounts and settings:

- `account create|view|list|select`
- `config show`

Advanced and troubleshooting families remain available:

- `doctor`
- `local`
- `sync`
- `relay`
- `net`
- `signer`
- `rpc`
- `myc`
- `runtime`
- `job`
- `listing`
- `find`

## Global Behavior

- default output is human-oriented
- `--output <human|json|ndjson>` is the global format selector
- `--json` and `--ndjson` remain supported compatibility aliases
- `--no-input` with alias `--non-interactive` disables prompting
- `--yes` approves supported confirmation prompts
- `--quiet`, `--verbose`, and `--trace` change the human information budget
- `--dry-run` is available on supported mutation commands
- `--no-color` disables ANSI color in human output
- path-writing flags remain command-local where needed:
  - `sell add --file`
  - `listing new --output`
  - `local export --output`
  - `local backup --output`

## Reference Docs

- `docs/command-surface.md` explains the current command families and global
  behavior
- `docs/nix.md` documents the canonical development and validation commands

## Validation

Run validation from this repo root:

```bash
nix run .#check
nix run .#test
```

Use `nix run .#release-acceptance` when preparing a release candidate, and use
`nix develop` before narrower ad hoc `cargo` commands.

## Copyright

Except as otherwise noted, all files in the `radroots_cli` distribution are

    Copyright (c) 2020-2026 Tyson Lupul and others.

For information on usage and redistribution, and for a DISCLAIMER OF ALL
WARRANTIES, see LICENSE included in the `radroots_cli` distribution.
