# Command Surface

This reference is derived from the live target parser in `src/target_cli.rs` and
the acceptance coverage in `tests/target_cli.rs`.

## Namespaces

The root help surface exposes these namespaces:

```text
workspace health config account signer relay store sync farm listing market basket order
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
output.

For supported mutation operations, `--dry-run` validates arguments, local
configuration, local state, path collisions, parseability, selected account
requirements, and local authority where applicable. It does not create durable
files, write accounts, sign, deliver to relays, or progress workflows.
Seller order decision dry-run also performs the relay-backed request lookup and
prior-decision preflight required for non-dry execution, then skips only signing
and relay publication.

Operations with required approval return `approval_required` with exit code `6`
unless `--approval-token` is present and non-empty after trimming whitespace or
`--dry-run` is active. `--approval-token` is local mutation-confirmation input,
not authentication or remote signer approval.

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

## Approval And Mutation Failures

Required approval operations:

- `account import`
- `account remove`
- `farm publish`
- `listing publish`
- `listing archive`
- `order submit`
- `order accept`
- `order decline`

Mutation commands return structured non-zero failures when a required mutation
target is absent. Missing local files, drafts, orders, listings, and other
concrete targets return `not_found` with exit code `4`. Missing actor or signer
authority returns the applicable account, signer, or provider error. Malformed
targets return `validation_failed`.

Relay-required trade mutations keep local validation and dry-run preflight.
Non-dry `farm publish`, `listing publish`, `listing archive`, `order submit`,
`order accept`, and `order decline` require `--approval-token`, a selected
secret-backed local account, and at least one configured relay from `--relay` or
runtime config. Successful publish output reports event ids, event kinds, target
relays, acknowledged relays, and failed relays. Buyer `market refresh` ingests
seller profile, farm, and active listing events from configured relays into the
local replica.

Order inventory accounting is reducer-owned. Non-dry `order submit` must fail
before signing when the draft quantity is zero or exceeds current local replica
availability. It must also check configured relays for same-order request
candidates: an identical visible valid request is returned as deduplicated, and
a changed or poisoned visible request fails validation before signing. Seller
`order accept` must verify the current active listing event, exact
request/commitment equality, prior decisions, and remaining relay-visible
listing availability before signing. Seller `order decline` does not reserve
inventory.

`order accept --dry-run` and `order decline --dry-run` reject `--offline`
because a truthful seller decision preflight requires relay access. Public
seller decision commands also reject a second visible valid seller decision for
the same request before signing or publishing.

Read and inspection commands may return successful `missing` views when no
mutation was requested.

## Dry-Run Operations

Supported mutating dry-run operations:

- `workspace init`
- `account create`
- `account import`
- `account remove`
- `account selection update`
- `account selection clear`
- `store init`
- `store backup create`
- `sync pull`
- `sync push`
- `farm create`
- `farm profile update`
- `farm location update`
- `farm fulfillment update`
- `farm publish`
- `listing create`
- `listing update`
- `listing publish`
- `listing archive`
- `market refresh`
- `basket create`
- `basket item add`
- `basket item update`
- `basket item remove`
- `basket quote create`
- `order submit`
- `order accept`
- `order decline`

Commands that do not advertise dry-run support return structured
`invalid_input` when invoked with `--dry-run`.

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
- `order event list [order-id]`
- `order accept <order-id>`
- `order decline <order-id> --reason <text>`

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

Shared order inspection:

- `order status get <order-id>`
- `order event watch [order-id]`

`order get` and `order list` inspect local buyer order drafts. Relay-backed
order requests and decisions are read from configured relays.

`order submit [order-id]` requires local market data, a current local listing
event id, positive item quantities, enough current listing availability, a local
buyer signer, and configured relays before publishing a kind `3422` order
request. Repeat submit checks visible same-order request events and deduplicates
an identical request instead of publishing a second event. `order submit
--dry-run` runs the same local listing, quantity, signer, and visible-request
preflights, then skips only the final publish.

`order event list [order-id]` is a seller-side bounded relay read for buyer
`TradeOrderRequested` events targeted at the selected seller account.

`order accept <order-id>` and `order decline <order-id> --reason <text>` are
seller-side signed writes. They fetch the seller-targeted buyer request from
configured relays, verify the selected seller authority, publish a kind `3423`
`TradeOrderDecision` event, and chain the decision to the buyer request event.
Dry-run performs the same relay-backed request, signer authority,
canonicalization, and prior-decision preflight without signing or publishing.
Non-dry execution rejects an already visible valid seller decision for the same
request. Malformed same-order seller-targeted request candidates and multiple
valid same-order seller-targeted request candidates fail as `validation_failed`
before signing or publishing. Decline dry-run preserves the declined decision
intent and trimmed reason while omitting event id, event kind, and acknowledged
relays. Accept preflight also verifies the referenced current listing event,
the accepted inventory commitment multiset, and remaining listing/bin
availability; decline remains no-reserve.

`order status get <order-id>` is a bounded relay read for buyer and seller
inspection. It fetches active request and decision events for the order id and
returns `missing`, `requested`, `accepted`, `declined`, or `invalid` from the
active order reducer. Reducer output is deterministic for the same signed event
set and reports invalid multiple-request, wrong-counterparty-decision, or
conflicting-decision state instead of choosing by relay arrival order. When a
local account is selected, request candidate relevance is scoped to events where
that account is the buyer or seller. Status output also exposes inventory
reservation state, commitment validity, listing event id, and per-bin reserved
and remaining quantities when available. Order issues expose stable codes,
fields, messages, and sorted event ids when event-scoped, so machine clients do
not depend on Rust debug strings.

Status inspection uses the same active request validity semantics as seller
decision preflight. Malformed same-order seller-targeted request candidates and
multiple valid same-order request candidates are invalid order state in status
output, not merely skipped relay events.

`order event watch` remains unavailable as an indefinite subscription surface.
Payment, fulfillment, buyer receipts, validation receipts, cancellation,
negotiation, revision, discount, question, and dispute commands are not active
order commands.

## Flows

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
radroots --format json --relay "$RELAY_URL" --approval-token approve order accept "$ORDER_ID"
radroots --format json --relay "$RELAY_URL" order status get "$ORDER_ID"
radroots --format json --relay "$RELAY_URL" --approval-token approve listing archive "$LISTING_FILE"
```
