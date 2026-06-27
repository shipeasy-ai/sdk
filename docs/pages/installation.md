# Installation

## Install

```bash
npm install @shipeasy/sdk
# or
pnpm add @shipeasy/sdk
# or
yarn add @shipeasy/sdk
```

The only runtime dependency is `murmurhash-js`. `zod` is an **optional** peer
dependency (only needed if you decode configs/experiments with a Zod schema).

## Runtime requirements

- **Node** ≥ 18 (the server build targets modern Node; also runs on Cloudflare
  Workers and Deno).
- Any evergreen **browser** for the `/client` build.
- TypeScript ≥ 5 recommended (full types ship with the package).

## Entrypoints / import lines

One package, conditionally-exported entrypoints:

```ts
// Server (Node / Cloudflare Worker / Deno) — uses the SERVER key
import { configure, Client, Engine, see } from "@shipeasy/sdk/server";

// Browser — uses the public CLIENT key
import { configure, Client, Engine, see, i18n } from "@shipeasy/sdk/client";

// Next.js helpers (App Router)
import { shipeasy } from "@shipeasy/sdk/server"; // SSR bootstrap handle

// OpenFeature providers (optional peer deps)
import { ShipeasyProvider } from "@shipeasy/sdk/openfeature-server";
import { ShipeasyProvider } from "@shipeasy/sdk/openfeature-web";
```

> **One key per entrypoint.** The server uses the server key, the browser uses
> the client key — never pass `clientKey` to the server entry or `apiKey` to the
> browser entry.

## No-bundler script loader

For sites without a build step, drop the script loader in — no `npm install`:

```html
<script
  src="https://cdn.shipeasy.ai/sdk/loader.js"
  data-sdk-key="sdk_client_..."
  data-user-id="user-123"
  data-attrs='{"plan":"pro","country":"US"}'
  defer
></script>
<script>
  await window.shipeasy.ready;
  if (window.shipeasy.getFlag("new_checkout")) { /* … */ }
</script>
```

For React projects, use [`@shipeasy/sdk-react`](https://github.com/shipeasy-ai/sdk-react)
(a thin wrapper over this package with a `<ShipeasyProvider>` and hooks).
