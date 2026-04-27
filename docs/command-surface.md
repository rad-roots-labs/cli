# Command Surface

This reference is derived from the live target parser in `src/target_cli.rs` and
the acceptance coverage in `tests/target_cli.rs`.

## Namespaces

The root help surface exposes these namespaces:

```text
workspace health config account signer relay store sync runtime job farm listing market basket order
```

## Global Flags

Supported global flags:

```text
--format human|json|ndjson
--account-id <account-id>
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

Human output is the default and uses concise status text. JSON output uses the
standard envelope with `operation_id` equal to `kind`. NDJSON output uses
newline-delimited frame records for operations that support it. Unsupported
output modes or unsupported NDJSON requests return structured `invalid_input`
output. Operations with required approval return `approval_required` with exit
code `6` unless `--approval-token` is present or `--dry-run` is active.
`--approval-token` is local mutation-confirmation input, not authentication or
remote signer approval.

## Signer Runtime Mode

The `signer status get` operation reports signer mode and readiness. The
supported signer mode is `local`, which uses the selected local account signer.
Signer mode is selected by runtime configuration:

```text
RADROOTS_SIGNER=local
```

```toml
[signer]
mode = "local"
```

MYC-selected configuration fails closed with `signer_mode_deferred` and must
not execute the configured MYC binary. Remote signer success flows and public
signer-session lifecycle commands are deferred.

## Operations

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
- `order submit [order-id]`
- `order get [order-id]`
- `order list`
- `order event list [order-id]`
- `order event watch [order-id]`

## Flows

Buyer flow:

```bash
radroots --format json market product search eggs
radroots --format json basket create basket_flow
radroots --format json basket item add basket_flow --listing-addr 30402:1111111111111111111111111111111111111111111111111111111111111111:AAAAAAAAAAAAAAAAAAAAAg --bin-id bin-1 --quantity 2
radroots --format json basket quote create basket_flow > quote.json
ORDER_ID="$(jq -r '.result.quote.order_id' quote.json)"
radroots --format json order list
radroots --format json --dry-run order submit "$ORDER_ID"
```

Seller flow:

```bash
radroots --format json account create
radroots --format json farm create --name "Green Farm" --location farmstand --country US --delivery-method pickup
radroots --format json listing create --output listing.toml --key eggs --title Eggs --category eggs --summary "Fresh eggs" --bin-id bin-1 --quantity-amount 1 --quantity-unit each --price-amount 6 --price-currency USD --price-per-amount 1 --price-per-unit each --available 10
radroots --format json listing validate listing.toml
radroots --format json --dry-run listing publish listing.toml
radroots --format json --approval-token approve listing publish listing.toml
```
