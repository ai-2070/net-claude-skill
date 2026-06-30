# net-event-bus — a Claude skill for the Net mesh event bus

A [Claude **Agent Skill**](https://docs.anthropic.com/en/docs/claude-code/skills) that teaches Claude how to integrate the [**Net**](https://github.com/ai-2070/net) library (`@net-mesh/sdk`, Rust/Python `net-sdk`, the Go binding, and the C `net.h`) as an event bus — pub/sub over the mesh, nRPC request/response, and the RedEX / CortEX / Dataforts persistence layers on top.

Net looks like Kafka/NATS/Redis Streams on the surface but has a fundamentally different model (no broker, hot subscribers, backpressure-as-silence, every node a peer). Out of the box, a coding agent will happily write integration code that **compiles, runs, and is wrong**. This skill loads the right mental model and verified per-SDK templates so Claude generates correct Net code.

> **Heads up:** this is a skill *about* Net — it does not install the Net library itself. Grab the SDK from [ai-2070/net](https://github.com/ai-2070/net).

---

## Quick install (≈1 minute)

A skill is just a directory containing a `SKILL.md`. Installing it means dropping the `net-event-bus/` folder where Claude Code looks for skills:

- **Personal** → `~/.claude/skills/` — available in every project on your machine.
- **Project** → `<your-repo>/.claude/skills/` — checked in and shared with your team.

Pick one of the methods below.

### Option A — Clone straight into your skills directory (recommended)

**macOS / Linux**
```bash
git clone https://github.com/ai-2070/claude-skill-net.git /tmp/claude-skill-net
mkdir -p ~/.claude/skills
cp -R /tmp/claude-skill-net/net-event-bus ~/.claude/skills/
```

**Windows (PowerShell)**
```powershell
git clone https://github.com/ai-2070/claude-skill-net.git $env:TEMP\claude-skill-net
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
Copy-Item -Recurse "$env:TEMP\claude-skill-net\net-event-bus" "$env:USERPROFILE\.claude\skills\"
```

You should end up with `~/.claude/skills/net-event-bus/SKILL.md`.

### Option B — Install into a single project

From the root of the repo where you want the skill available to your whole team:

```bash
mkdir -p .claude/skills
git clone https://github.com/ai-2070/claude-skill-net.git /tmp/claude-skill-net
cp -R /tmp/claude-skill-net/net-event-bus .claude/skills/
git add .claude/skills/net-event-bus && git commit -m "Add net-event-bus Claude skill"
```

### Option C — Symlink to stay up to date

Clone once, then symlink so `git pull` updates the installed skill:

```bash
git clone https://github.com/ai-2070/claude-skill-net.git ~/src/claude-skill-net
ln -s ~/src/claude-skill-net/net-event-bus ~/.claude/skills/net-event-bus
```

---

## Verify it's installed

1. Confirm the file is on disk:
   ```bash
   ls ~/.claude/skills/net-event-bus/SKILL.md
   ```
2. Start (or restart) Claude Code and run `/skills` — you should see **net-event-bus** listed.
3. Trigger it with a prompt that mentions Net, e.g.:
   > *"Wire up a Net publisher and subscriber over the mesh in TypeScript."*

   Claude loads the skill automatically when your request matches (imports of `@net-mesh/sdk` / `net-sdk`, or phrases like *pub/sub with Net*, *nRPC*, *mesh RPC*, *RedEX*, *CortEX*, *Dataforts*, *gang scheduler*, *claim an island*, …).

---

## What's inside

The skill is progressive-disclosure: `SKILL.md` is the always-on entry point, and the reference files are loaded on demand.

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
| `redex.md` | Durable per-channel append-only logs (replay from offset, retention). |
| `cortex.md` | Folded queryable state (SQLite-shaped queries, NetDB). |
| `dataforts.md` | Greedy caching, data gravity, blob refs, read-your-writes. |
| `runtime.md` | Shutdown contract, error handling, async-runtime integration. |
| `observability.md` | Catching silent drops; Prometheus/OTel wiring. |
| `payloads.md` | Event schema, size limits, cross-language interop traps. |
| `testing.md` | Fixtures, race conditions, CI gotchas. |
| `gotchas.md` | Migrating from Kafka / NATS / Redis Streams / Pulsar. |
| `examples/` | Minimal runnable hello-world per SDK (TS, Py, Rust, Go, C). |

---

## Updating

- **Cloned/copied (Options A & B):** re-run the copy step, or `git pull` in the clone and copy again.
- **Symlinked (Option C):** `git pull` in `~/src/claude-skill-net` — the installed skill tracks it automatically.

## Uninstall

```bash
rm -rf ~/.claude/skills/net-event-bus      # personal
rm -rf .claude/skills/net-event-bus        # project
```

---

## Links

- **Net library & SDKs:** https://github.com/ai-2070/net
- **Claude Code skills docs:** https://docs.anthropic.com/en/docs/claude-code/skills
