# radroots_cli

`radroots_cli` provides the `radroots` command-line interface for local-first
food trade workflows on Nostr.

The CLI exposes resource-oriented commands. The authoritative parser lives in
`src/target_cli.rs`, and the command contract is covered by `tests/target_cli.rs`.

## Commands

Top-level namespaces:

- `workspace`
- `health`
- `config`
- `account`
- `signer`
- `relay`
- `store`
- `sync`
- `runtime`
- `job`
- `farm`
- `listing`
- `market`
- `basket`
- `order`

## Global Behavior

- default output is `human`
- `--format <human|json|ndjson>` is the global format selector
- `--account-id` and `--relay` scope runtime context
- `--offline` and `--online` select network posture
- `--dry-run` is available on registry-approved mutation operations
- `--approval-token` carries local mutation-confirmation input for operations
  that require it
- `--no-input`, `--quiet`, `--verbose`, `--trace`, and `--no-color` control
  interaction and output posture

## Signer Runtime Mode

Signer mode is selected by runtime configuration. The default mode is `local`,
which uses the selected local account signer. Use `RADROOTS_SIGNER=local` or
this app or workspace config:

```toml
[signer]
mode = "local"
```

Inspect the selected mode with `radroots --format json signer status get`. MYC
configuration remains guarded by tests for status parsing and no-fallback
behavior, but remote signer success flows are not part of the documented public
workflow.

Machine-readable command output uses the standard envelope:

- `schema_version`
- `operation_id`
- `kind`
- `request_id`
- `correlation_id`
- `idempotency_key`
- `dry_run`
- `actor`
- `result`
- `warnings`
- `errors`
- `next_actions`

## Example Flows

Buyer flow:

```bash
radroots --format json market product search eggs
radroots --format json basket create basket_flow
radroots --format json basket item add basket_flow --listing-addr 30402:1111111111111111111111111111111111111111111111111111111111111111:AAAAAAAAAAAAAAAAAAAAAg --bin-id bin-1 --quantity 2
radroots --format json basket quote create basket_flow
radroots --format json --dry-run order submit <order-id>
```

Seller flow:

```bash
radroots --format json listing create --output listing.toml --key eggs --title Eggs --bin-id bin-1 --quantity-amount 1 --quantity-unit each --price-amount 6 --price-currency USD --price-per-amount 1 --price-per-unit each --available 10
radroots --format json listing validate listing.toml
radroots --format json --dry-run listing publish listing.toml
radroots --format json order list
```

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
