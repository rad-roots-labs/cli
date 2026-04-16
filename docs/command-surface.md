# Command Surface

This reference is derived from the live parser and help surface in `src/cli.rs`
and `tests/help.rs`.

## Workflow-First Entry Points

Start here:

- `setup [seller|buyer|both]` prepares the local workflow posture for sellers,
  buyers, or both; the default role is `both`
- `status` reports what is ready and what still needs attention

Sell from your farm:

- `farm init` creates or refreshes a farm draft progressively
- `farm set <field> <value...>` updates one writable farm field
- `farm check` reports farm readiness
- `farm show` prints the current farm draft
- `farm publish` submits the current farm draft
- `farm setup` remains available as the one-command compatibility path

- `sell add <product>` creates a local listing draft
- `sell show <file>` reads a local listing draft
- `sell check <file>` validates a local listing draft
- `sell publish <file>` publishes a listing draft
- `sell update <file>` updates a published listing from a draft
- `sell pause <file>` pauses a published listing
- `sell reprice <file> <price_expr>` changes draft pricing
- `sell restock <file> <available>` changes draft stock

Buy from the market:

- `market update` uses the current sync update surface and stays honest about
  present ingest limitations
- `market search <query...>` searches local market data
- `market view <listing>` shows one published listing

- `order create` creates a local order draft
- `order view <order>` shows one order
- `order list` lists local orders
- `order submit <order>` submits a draft
- `order submit <order> --watch` submits and appends human watch snapshots
- `order watch <order>` watches a submitted order
- `order cancel <order>` cancels a submitted order
- `order history` prints submitted order history

Accounts and settings:

- `account create`, `account view`, `account list`, and `account select`
- `config show`

## Preserved Compatibility Surface

The human-first layer is additive. The lower-level machine-first families remain
available for automation, troubleshooting, and deeper runtime control:

- `doctor`
- `local init|status|export|backup`
- `sync status|pull|push|watch`
- `relay list`
- `net status`
- `signer status`
- `rpc status|sessions`
- `myc status`
- `runtime install|uninstall|status|start|stop|restart|logs|config show|config set`
- `job list|get|watch`
- `listing new|validate|get|publish|update|archive`
- `find`

Compatibility aliases also remain available where the source defines them:

- `account new|whoami|ls|use`
- `farm status|get`
- `order new|get|ls`
- `relay ls`
- `job ls`

## Global Behavior

Output modes:

- human output is the default
- `--output <human|json|ndjson>` is the global output selector
- `--json` and `--ndjson` remain supported aliases for compatibility
- singular commands remain human or JSON only unless the parser explicitly
  allows NDJSON

NDJSON is currently supported on these parser surfaces:

- `account list`
- `relay list`
- `job list`
- `job watch`
- `rpc sessions`
- `order list`
- `order watch`
- `order history`
- `sync watch`
- `find`
- `market search`

Interactivity and verbosity:

- `--no-input` with alias `--non-interactive` disables prompts
- `--yes` approves supported confirmations
- `--quiet`, `--verbose`, and `--trace` adjust human output detail
- `--no-color` disables ANSI color in human output

Mutation behavior:

- `--dry-run` is available on supported mutation commands
- file-writing flags stay local to the commands that own files:
  - `sell add --file`
  - `listing new --output`
  - `local export --output`
  - `local backup --output`

## Example Flows

Seller setup and farm draft:

```bash
radroots setup seller
radroots farm init --name "South Field" --location "Santa Fe, NM"
radroots farm set delivery pickup
radroots farm check
```

Listing authoring:

```bash
radroots sell add tomatoes --pack "1 kg" --price "10 USD/kg" --stock 25
radroots sell check ./listing.toml
radroots sell publish ./listing.toml
```

Market and order flow:

```bash
radroots market search eggs
radroots market view sf-tomatoes
radroots order create --listing sf-tomatoes --bin bin-1 --qty 2
radroots order submit ord_demo --watch
```
