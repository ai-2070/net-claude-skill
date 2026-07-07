# net-claude-skill — Claude skills for the Net mesh

Two [Claude **Agent Skills**](https://docs.anthropic.com/en/docs/claude-code/skills) that teach Claude how to integrate the [**Net**](https://github.com/ai-2070/net) library (`@net-mesh/sdk`, Rust/Python `net-sdk`, the Go binding, and the C `net.h`):

- **`net-event-bus`** — Net as an event bus: pub/sub over the mesh, nRPC request/response, the MCP bridge (`net wrap` / `net mcp serve`), the gang-claim scheduler, and the RedEX / CortEX / Dataforts persistence layers on top.
- **`net-payments`** — x402-native payments on the mesh: pricing a capability at discovery, signed quotes, the provider-side lifecycle engine (quote → verify → settle → bill), the caller-side pay-to-invoke flow, tiered on-chain verification, and spend policy.

Net looks like Kafka/NATS/Redis Streams on the surface but has a fundamentally different model (no broker, hot subscribers, backpressure-as-silence, every node a peer). Net Payments looks like a dozen payment SDKs but is non-custodial and never moves money — it only signs the commercial facts around invocation. Out of the box, a coding agent will happily write integration code that **compiles, runs, and is wrong**. These skills load the right mental model and verified per-SDK templates so Claude generates correct Net code.

> **Heads up:** these are skills *about* Net — they do not install the Net library itself. Grab the SDK from [ai-2070/net](https://github.com/ai-2070/net).

---

## Quick install (≈1 minute)

A skill is just a directory containing a `SKILL.md`. Installing means dropping the skill folder(s) — `net-event-bus/` and/or `net-payments/` — where Claude Code looks for skills:

- **Personal** → `~/.claude/skills/` — available in every project on your machine.
- **Project** → `<your-repo>/.claude/skills/` — checked in and shared with your team.

Install both, or just the one you need. Pick one of the methods below.

### Option A — Clone straight into your skills directory (recommended)

**macOS / Linux**
```bash
git clone https://github.com/ai-2070/net-claude-skill.git /tmp/net-claude-skill
mkdir -p ~/.claude/skills
cp -R /tmp/net-claude-skill/net-event-bus /tmp/net-claude-skill/net-payments ~/.claude/skills/
```

**Windows (PowerShell)**
```powershell
git clone https://github.com/ai-2070/net-claude-skill.git $env:TEMP\net-claude-skill
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
Copy-Item -Recurse "$env:TEMP\net-claude-skill\net-event-bus" "$env:USERPROFILE\.claude\skills\"
Copy-Item -Recurse "$env:TEMP\net-claude-skill\net-payments" "$env:USERPROFILE\.claude\skills\"
```

You should end up with `~/.claude/skills/net-event-bus/SKILL.md` and `~/.claude/skills/net-payments/SKILL.md`. (Copying just one folder installs just that skill.)

### Option B — Install into a single project

From the root of the repo where you want the skills available to your whole team:

```bash
mkdir -p .claude/skills
git clone https://github.com/ai-2070/net-claude-skill.git /tmp/net-claude-skill
cp -R /tmp/net-claude-skill/net-event-bus /tmp/net-claude-skill/net-payments .claude/skills/
git add .claude/skills/net-event-bus .claude/skills/net-payments && git commit -m "Add Net Claude skills"
```

### Option C — Symlink to stay up to date

Clone once, then symlink so `git pull` updates the installed skills:

```bash
git clone https://github.com/ai-2070/net-claude-skill.git ~/src/net-claude-skill
ln -s ~/src/net-claude-skill/net-event-bus ~/.claude/skills/net-event-bus
ln -s ~/src/net-claude-skill/net-payments ~/.claude/skills/net-payments
```

---

## Verify it's installed

1. Confirm the files are on disk:
   ```bash
   ls ~/.claude/skills/net-event-bus/SKILL.md ~/.claude/skills/net-payments/SKILL.md
   ```
2. Start (or restart) Claude Code and run `/skills` — you should see **net-event-bus** and **net-payments** listed.
3. Trigger one with a prompt that mentions Net, e.g.:
   > *"Wire up a Net publisher and subscriber over the mesh in TypeScript."*
   >
   > *"Price a Net capability with x402 and charge callers to invoke it."*

   Claude loads the matching skill automatically when your request matches — **net-event-bus** on imports of `@net-mesh/sdk` / `net-sdk` or phrases like *pub/sub with Net*, *nRPC*, *mesh RPC*, *RedEX*, *CortEX*, *Dataforts*, *gang scheduler*, *claim an island*, *net wrap / mcp serve*; **net-payments** on imports of `net-payments` / `net_payments` or phrases like *price a capability*, *pay to invoke*, *x402*, *settle on Base/Solana*, *spend limit*, …

---

## What's inside

Both skills are progressive-disclosure: `SKILL.md` is the always-on entry point, and the reference files are loaded on demand.

### `net-event-bus/`

| File | Loaded when |
|---|---|
| `SKILL.md` | Entry point — routing table + the 5-point mental model. |
| `concepts.md` | **Always, before writing code** — why Net is not a broker. |
| `apis.md` | Generating code — verified publish/subscribe/lifecycle templates per SDK. |
| `patterns.md` | Mapping a task ("I need a relay / persistence / fan-out") to a recipe. |
| `mesh.md` | Multi-host deploys — PSK/identity bootstrap, discovery, NAT traversal. |
| `capabilities.md` | Routing to "the GPU node" / a node with model X loaded. |
| `scheduler.md` | Atomically claiming a contended resource (island/slot/seat) + workflows. |
| `streams.md` | Ordered point-to-point delivery with credit-grant backpressure. |
| `nrpc.md` | Request/response — typed call → reply, deadlines, retries, hedging. |
| `mcp.md` | The MCP bridge — `net wrap` a stdio server as mesh capabilities, or `net mcp serve` the mesh to a local host; pinning + credential forwarding. |
| `redex.md` | Durable per-channel append-only logs (replay from offset, retention). |
| `cortex.md` | Folded queryable state (SQLite-shaped queries, NetDB). |
| `dataforts.md` | Greedy caching, data gravity, blob refs, read-your-writes. |
| `runtime.md` | Shutdown contract, error handling, async-runtime integration. |
| `observability.md` | Catching silent drops; Prometheus/OTel wiring. |
| `payloads.md` | Event schema, size limits, cross-language interop traps. |
| `filter-dsl.md` | Consumer-side content filtering — equality `$and`/`$or`/`$not` predicates on the bus. |
| `error-codes.md` | Classifying a specific error variant — the full core-crate + subsystem taxonomy. |
| `cli.md` | The `net-mesh` CLI — `transfer` (blob/dir) and `typegen` commands, exit codes. |
| `testing.md` | Fixtures, race conditions, CI gotchas. |
| `gotchas.md` | Migrating from Kafka / NATS / Redis Streams / Pulsar. |
| `event-semantics.md` | Naming events / what an event may assert — a fact observed at one layer, not an end-to-end `200 OK`. |
| `examples/` | Minimal runnable hello-world per SDK (TS, Py, Rust, Go, C). |

### `net-payments/`

| File | Loaded when |
|---|---|
| `SKILL.md` | Entry point — routing table + the TL;DR mental model + integration workflow. |
| `concepts.md` | **Always first** — the mental model, the category line, the eight doctrines, the review invariant. |
| `object-model.md` | Touching the five Net envelopes — fields, canonical signing regime, versioning, idempotency, amounts. |
| `x402.md` | Touching x402 structures — `X402Carry` byte-preservation, `PaymentRequirements` / `PaymentPayload`, CAIP ids, the `exact` EVM scheme. |
| `provider.md` | Charging for a capability — the `PaymentEngine` lifecycle (quote → verify → settle → serve → bill), pricing at publish, `serve_payments`. |
| `caller.md` | Paying to invoke — `CallerPaymentFlow` over a `ProviderChannel`, spend check, the approval loop, the MCP gateway path. |
| `facilitator.md` | Wiring the `verify` / `settle` boundary — the `Facilitator` trait, the mock, the real `HttpFacilitator`, config packs, auth. |
| `verification.md` | Confidence — the `observed / confirmed(n) / final` tiers, the independent `ChainChecker`, reorg freeze, replay. |
| `signer.md` | Settlement signing — the `SchemeSigner` seam, `ExternalSigner` / `ExternalSvmSigner`, EIP-3009 / SPL, the no-raw-signing invariant. |
| `spend-policy.md` | Limits, budgets, approvals — `SpendPolicyEngine`, the fail-closed default posture, delegation inheritance. |
| `networks.md` | Enabling a network — CAIP-2 / CAIP-19, the signed asset registry, the Base → Solana → xrpl "config, not code" ladder. |
| `billing.md` | Usage records / a billing stream — `BillingLog` (subscribe/read/export), immutability, what billing is NOT. |
| `http402.md` | A Net agent paying an external x402 HTTP API — the outbound `X402HttpFlow`, the header-only v2 transport. |
| `bindings.md` | Per-language support — only Rust + Python have a native flow; Node is read-only pricing; Go is absent. Python `CapabilityGateway`. |
| `testing.md` | Cross-language golden vectors, the mock conformance suite, the key-invariant negative test, the env-gated live run. |
| `gotchas.md` | Wrong mental model, migrating, or before merging — the review invariant, "what not to build," byte-preservation traps. |

---

## Updating

- **Cloned/copied (Options A & B):** re-run the copy step, or `git pull` in the clone and copy again.
- **Symlinked (Option C):** `git pull` in `~/src/net-claude-skill` — the installed skills track it automatically.

## Uninstall

```bash
rm -rf ~/.claude/skills/net-event-bus ~/.claude/skills/net-payments   # personal
rm -rf .claude/skills/net-event-bus .claude/skills/net-payments       # project
```

---

## Links

- **Net library & SDKs:** https://github.com/ai-2070/net
- **Claude Code skills docs:** https://docs.anthropic.com/en/docs/claude-code/skills

## License

Licensed under the [Apache License 2.0](LICENSE).
