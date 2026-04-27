# Command Surface

This reference is derived from the live target parser in `src/target_cli.rs` and
the public acceptance coverage in `tests/target_cli.rs`.

## Public Namespaces

The root help surface exposes only these namespaces:

```text
workspace health config account signer relay store sync runtime job farm listing market basket order
```

These old or deferred namespaces are rejected:

```text
setup status doctor sell find local net myc rpc product message approval agent
```

## Global Flags

Allowed global flags:

```text
--format human|json|ndjson
--account-id <account-id>
--farm-id <farm-id>
--profile <profile-name>
--signer-session-id <session-id>
--relay <relay-url>
--offline
--online
--dry-run
--idempotency-key <key>
--correlation-id <id>
--approval-token <token>
--no-input
--quiet
--verbose
--trace
--no-color
```

Removed global flags:

```text
--output
--json
--ndjson
--yes
--non-interactive
```

JSON output uses the standard envelope with `operation_id` equal to `kind`.
Unsupported output modes return a structured `invalid_input` envelope. Operations
with required approval return `approval_required` with exit code `6` unless
`--approval-token` is present or `--dry-run` is active.

## MVP Operations

Workspace and diagnostics:

- `workspace init`
- `workspace get`
- `health status get`
- `health check run`
- `config get`

Accounts and local store:

- `account create`
- `account import`
- `account get`
- `account list`
- `account remove`
- `account selection get`
- `account selection update`
- `account selection clear`
- `store init`
- `store status get`
- `store export`
- `store backup create`

Runtime and network posture:

- `signer status get`
- `relay list`
- `sync status get`
- `sync pull`
- `sync push`
- `sync watch`
- `runtime status get`
- `runtime start`
- `runtime stop`
- `runtime restart`
- `runtime log watch`
- `runtime config get`
- `job get`
- `job list`
- `job watch`

Seller operations:

- `farm create`
- `farm get`
- `farm profile update`
- `farm location update`
- `farm fulfillment update`
- `farm readiness check`
- `farm publish`
- `listing create`
- `listing get`
- `listing list`
- `listing update`
- `listing validate`
- `listing publish`
- `listing archive`

Buyer operations:

- `market refresh`
- `market product search [query...]`
- `market listing get [key]`
- `basket create [basket-id]`
- `basket get [basket-id]`
- `basket list`
- `basket item add [basket-id] --listing-addr <addr>|--listing <key> --bin-id <bin-id> [--quantity <count>]`
- `basket item update [basket-id] [--item-id <item-id>] [--listing-addr <addr>|--listing <key>] [--bin-id <bin-id>] [--quantity <count>]`
- `basket item remove [basket-id] [item-id]`
- `basket validate [basket-id]`
- `basket quote create [basket-id]`
- `order submit [order-id] [--watch]`
- `order get [order-id]`
- `order list`
- `order event list [order-id]`
- `order event watch [order-id]`

## Acceptance Flows

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
radroots --format json listing create --output listing.toml --key eggs --title Eggs --bin-id bin-1 --quantity-amount 1 --quantity-unit dozen --price-amount 6 --price-currency USD --price-per-amount 1 --price-per-unit dozen --available 10
radroots --format json listing validate listing.toml
radroots --format json --dry-run listing publish listing.toml
radroots --format json order list
```
