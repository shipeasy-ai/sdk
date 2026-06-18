# @shipeasy/sdk

Feature gates, runtime configs, experiments, and metrics for the
[Shipeasy](https://shipeasy.ai) hosted service.

> Source-available under the [Shipeasy-SAL 1.0](./LICENSE). Use it freely
> as a client of Shipeasy. Don't use it to build a competing service.

## Install

```bash
npm install @shipeasy/sdk
# or
pnpm add @shipeasy/sdk
```

For React projects use [`@shipeasy/sdk-react`](https://github.com/shipeasy-ai/sdk-react)
which wraps this package with a `<ShipeasyProvider>` and hooks.

## Quickstart — browser

```ts
import { FlagsClientBrowser } from "@shipeasy/sdk/client";

const client = new FlagsClientBrowser({
  sdkKey: process.env.NEXT_PUBLIC_SHIPEASY_CLIENT_KEY!,
});

await client.identify({ user_id: "user-123", plan: "pro" });

if (client.getFlag("new_checkout")) {
  // ship it
}

const cfg = client.getConfig<{ max_uploads: number }>("upload_limits");
const { params } = client.getExperiment("hero_cta", {
  primary_label: "Sign up",
});
```

`identify()` automatically merges browser context (`locale`, `timezone`,
`path`, `referrer`, `screen_*`, `user_agent`) and a persisted
`anonymous_id` into the payload — so gate rules can target by locale or
holdouts can hash anonymous visitors out of the box.

## Quickstart — server (Node, Cloudflare Worker, Deno)

```ts
import { FlagsClient } from "@shipeasy/sdk/server";

const client = new FlagsClient({ sdkKey: process.env.SHIPEASY_SERVER_KEY! });

const cfg = await client.getConfig("plan_limits", { user_id: "u-1" });
```

## Error tracking — `see()`

`see` (shipeasy error) is the structured error reporter — opes-style: every
handled exception documents its **product consequence**, not just its stack.
Works in vanilla JS on both sides; the whole grammar hangs off one import:

```ts
import { see } from "@shipeasy/sdk/client"; // or "@shipeasy/sdk/server"

try {
  await submitOrder(order);
} catch (e) {
  see(e).causes_the("checkout").to("use cached prices").extras({ order_id: order.id });
}

// Non-exception problems — the name is a stable identifier (it participates
// in the issue fingerprint); all variable data goes in .extras():
if (rows.length > LIMIT) {
  see.Violation("large query")
    .causes_the("search results")
    .to("be trimmed")
    .extras({ rows: rows.length });
}

// Expected control-flow exceptions: document them, report nothing —
// auto-capture skips marked errors. The reason must start with "because".
try {
  return decodeFoo(blob);
} catch (e) {
  see.ControlFlowException(e).because("because it wasn't an encoded Foo");
  return decodeBar(blob);
}
```

Reports land in the Shipeasy **errors** primitive: fingerprint-grouped issues
(open / resolved / ignored, regression auto-reopens) with a near-real-time
occurrence timeseries. The chain dispatches on the next microtask — no
`.send()` — and ships immediately (`sendBeacon` in the browser, fire-and-forget
`fetch` on the server), spam-guarded by a 30s dedup window and a per-session
cap.

The client SDK also auto-captures **network failures** (fetch network errors +
5xx) into the same primitive (`autoCollect: { errors }`, on by default) — each
names a specific endpoint and a specific outcome. It deliberately does **not**
blanket-report uncaught exceptions or unhandled promise rejections: those carry
no actionable consequence ("the page hit an error" names the plumbing, not the
feature). Code that knows the consequence reports it explicitly with `see()` at
the catch site.

**Rules**: if you don't know the consequence, don't catch the exception. You
**may** `see()` then re-throw — the re-thrown error links to its inner report
as a `caused_by` chain instead of double-counting. Never use `see.Violation()`
for a caught exception (you'd drop the stack). No PII or high-cardinality data
in extras.

## Drop-in `<script>` loader (no bundler)

```html
<script
  src="https://cdn.shipeasy.ai/sdk/loader.js"
  data-sdk-key="sdk_client_..."
  data-user-id="user-123"
  data-user-email="u@x.com"
  data-user-plan="pro"
  data-attrs='{"country":"US"}'
  defer
></script>
<script>
  await window.shipeasy.ready;
  if (window.shipeasy.getFlag("new_checkout")) { /* … */ }
</script>
```

The loader IIFE is published to a public R2 bucket on every release and
cached for 1y at `loader-vX.Y.Z.js` (immutable) plus a rolling 5-minute
`loader.js`.

## Testing

For unit tests, build a **no-network** client with `forTesting()` and seed every
entity with local overrides (Statsig-style). In test mode the client is already
"initialized"/"ready", `init()`/`initOnce()`/`identify()` are no-ops (they never
fetch), `track()` is a no-op, telemetry is disabled, and no SDK key is required —
so your tests never touch the network.

```ts
// Server (Node / Cloudflare Worker / Deno)
import { FlagsClient } from "@shipeasy/sdk/server";

const client = FlagsClient.forTesting();

client.overrideFlag("new_checkout", true);
client.overrideConfig("upload_limits", { max_uploads: 50 });
client.overrideExperiment("hero_cta", "treatment", { primary_label: "Buy now" });

client.getFlag("new_checkout", { user_id: "u1" }); // true
client.getConfig("upload_limits"); // { max_uploads: 50 }
client.getExperiment("hero_cta", { user_id: "u1" }, { primary_label: "Sign up" });
// → { inExperiment: true, group: "treatment", params: { primary_label: "Buy now" } }

client.track("u1", "purchase"); // no-op — never hits the network
client.clearOverrides(); // reset every override back to default
```

```ts
// Browser (vanilla JS — no React required)
import { FlagsClientBrowser } from "@shipeasy/sdk/client";

const client = FlagsClientBrowser.forTesting();

client.overrideFlag("new_checkout", true);
client.overrideConfig("upload_limits", { max_uploads: 50 });
client.overrideExperiment("hero_cta", "treatment", { primary_label: "Buy now" });

client.getFlag("new_checkout"); // true
client.getConfig("upload_limits"); // { max_uploads: 50 }
client.getExperiment("hero_cta", { primary_label: "Sign up" });
// → { inExperiment: true, group: "treatment", params: { primary_label: "Buy now" } }

client.track("purchase"); // no-op
client.clearOverrides();
```

The `override*` setters also work on a **normal** client (not just `forTesting()`),
mirroring Statsig — a programmatic override always wins over the fetched values.
In the browser the precedence is: programmatic override > URL/devtools override
(`?se_gate_…` / `?se_config_…` / `?se_exp_…`) > the server's evaluation.

**API:**

```ts
overrideFlag(name: string, value: boolean): void;
overrideConfig(name: string, value: unknown): void;
overrideExperiment(name: string, group: string, params: Record<string, unknown>): void;
clearOverrides(): void;
```

## Devtools overlay

Press `Shift+Alt+S` on any page running the SDK (or append `?se=1` to the
URL). The Shipeasy devtools panel mounts in a Shadow DOM overlay and lets
you flip every gate / config / experiment / translation **for the current
session only** — handy for QA, demos, and bug repro.

## Documentation

Full docs at [docs.shipeasy.ai](https://docs.shipeasy.ai). API surfaces
covered there: targeting rules, holdouts, sequential stats, custom
metrics, Slack digests, OAuth/SSO, Claude/MCP integration.

## License

[Shipeasy-SAL 1.0](./LICENSE) — source-available, non-commercial-use,
permitted for use as a Shipeasy client.
