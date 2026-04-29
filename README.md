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
- `--dry-run` validates supported mutation operations without durable writes,
  signing, relay delivery, or workflow progression
- `--approval-token` carries local mutation-confirmation input for required
  operations; absent, empty, and whitespace-only values are rejected
- `--no-input`, `--quiet`, `--verbose`, `--trace`, and `--no-color` control
  interaction and output posture

## Signer Runtime Mode

The supported signer mode is `local`, which uses the selected local account
signer. Use `RADROOTS_SIGNER=local` or this app or workspace config:

```toml
[signer]
mode = "local"
```

Inspect the selected mode with `radroots --format json signer status get`.
MYC-selected configuration fails closed with `signer_mode_deferred` and must not
execute the configured MYC binary. Remote signer success flows and public
signer-session lifecycle commands are deferred.

Human output is concise status text for interactive use. JSON output uses the
standard envelope:

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

NDJSON output uses newline-delimited frame records for operations that support
streaming or machine-followable frame output. Unsupported NDJSON requests fail
with structured `invalid_input` output.

Required approval gates are `account import`, `account remove`, `farm publish`,
`listing publish`, `listing archive`, `order submit`, `order accept`, and
`order decline`. Dry-run skips approval requirements because it does not execute
the mutation.

Mutation commands fail with structured non-zero output when a required target is
missing. Read commands may return successful `missing` views when no mutation is
requested.

Relay-required trade mutations keep local validation and dry-run preflight.
Non-dry `farm publish`, `listing publish`, `listing archive`, `order submit`,
`order accept`, and `order decline` require `--approval-token`, a selected
secret-backed local account, and at least one configured relay from `--relay` or
runtime config. Successful publish output reports event ids, event kinds, target
relays, acknowledged relays, and failed relays. Buyer `market refresh` ingests
seller profile, farm, and active listing events from configured relays into the
local replica.

Seller `order accept --dry-run` and `order decline --dry-run` perform the same
relay-backed request lookup, local signer authority, canonical payload, and
prior-decision preflight as non-dry execution. They reject `--offline`, skip
only signing and publishing, and fail when a valid seller decision for the same
request is already visible. Malformed same-order seller-targeted request
candidates and multiple valid same-order seller-targeted request candidates
fail as validation errors before signing or publishing. Order issue output uses
stable issue codes and sorted event ids for machine clients.

`order status get` inspects the same signed active order event set used by
seller decision preflight. It reports malformed same-order request candidates,
multiple valid request candidates, and conflicting decisions as invalid order
state with stable issue codes instead of presenting those event sets as valid
requested orders. When a local account is selected, status inspection admits
only request events where the selected account is the buyer or seller, so
same-order events for another actor cannot poison the selected actor's view.

Order inventory accounting is reducer-owned. Buyer `order submit` must reject
zero or over-available requested quantities before signing, and repeat submit
deduplicates an identical already visible request instead of publishing a
second kind `3422` event. Seller `order accept` must verify the referenced
current listing event, exact request/commitment equality, prior decisions, and
remaining listing availability before signing. Seller `order decline` is
terminal and does not reserve inventory. `order status get` exposes inventory
reservation state, commitment validity, listing event id, and per-bin
reserved/remaining details when available.

## Example Flows

Buyer flow:

```bash
RELAY_URL="ws://127.0.0.1:8080"
radroots --format json account create
radroots --format json signer status get
radroots --format json store init
radroots --format json --relay "$RELAY_URL" market refresh
radroots --format json market product search eggs
radroots --format json basket create basket_flow
radroots --format json basket item add basket_flow --listing-addr 30402:1111111111111111111111111111111111111111111111111111111111111111:AAAAAAAAAAAAAAAAAAAAAg --bin-id bin-1 --quantity 2
radroots --format json basket quote create basket_flow > quote.json
ORDER_ID="$(jq -r '.result.quote.order_id' quote.json)"
radroots --format json order list
radroots --format json --dry-run order submit "$ORDER_ID"
radroots --format json --relay "$RELAY_URL" --approval-token approve order submit "$ORDER_ID"
radroots --format json --relay "$RELAY_URL" order status get "$ORDER_ID"
```

Seller flow:

```bash
radroots --format json account create
radroots --format json signer status get
radroots --format json farm create --name "Green Farm" --location farmstand --country US --delivery-method pickup
radroots --format json listing create --key eggs --title Eggs --category eggs --summary "Fresh eggs" --bin-id bin-1 --quantity-amount 1 --quantity-unit each --price-amount 6 --price-currency USD --price-per-amount 1 --price-per-unit each --available 10 > listing-create.json
LISTING_FILE="$(jq -r '.result.file' listing-create.json)"
radroots --format json listing list
radroots --format json listing validate "$LISTING_FILE"
radroots --format json --dry-run listing publish "$LISTING_FILE"
RELAY_URL="ws://127.0.0.1:8080"
radroots --format json --relay "$RELAY_URL" --approval-token approve farm publish
radroots --format json --relay "$RELAY_URL" --approval-token approve listing publish "$LISTING_FILE"
radroots --format json --relay "$RELAY_URL" order event list
ORDER_ID="<buyer-order-id>"
radroots --format json --relay "$RELAY_URL" --dry-run order accept "$ORDER_ID"
radroots --format json --relay "$RELAY_URL" --approval-token approve order accept "$ORDER_ID"
radroots --format json --relay "$RELAY_URL" order status get "$ORDER_ID"
radroots --format json --relay "$RELAY_URL" --approval-token approve listing archive "$LISTING_FILE"
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
