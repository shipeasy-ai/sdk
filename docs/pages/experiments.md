# A/B experiments (`getExperiment` + `track`)

`getExperiment` enrolls a user into an experiment and returns the assigned
group plus its parameters. You read parameters from the result and record a
conversion with `track`.

## Read an experiment

```ts
import { configure, Client } from "@shipeasy/sdk/server"; // or /client

configure({ apiKey: process.env.SHIPEASY_SERVER_KEY! });
const flags = new Client(req.user);

const { inExperiment, group, params } = flags.getExperiment("hero_cta", {
  primary_label: "Sign up", // default params (the control / not-enrolled value)
});

render(params.primary_label);
```

## `ExperimentResult`

```ts
interface ExperimentResult<P> {
  inExperiment: boolean;             // false if not enrolled (targeting/holdout/allocation)
  group: string;                     // "control" | "treatment" | … (your variation key)
  params: P;                         // the variation's params, or your defaults
}
```

When the user isn't enrolled, `inExperiment` is `false`, `group` is `"control"`,
and `params` is exactly the `defaultParams` you passed — so reading
`params.<key>` is always safe.

### Decoding params

Pass an optional `decode` to validate/shape the params:

```ts
const { params } = flags.getExperiment(
  "hero_cta",
  { primary_label: "Sign up" },
  (raw) => HeroSchema.parse(raw),
);
```

## Track conversions

Record the success event so the analysis pipeline can compute lift. Conversion
events are attributed to the enrolled user.

```ts
// Server — track takes the user id explicitly
import { track } from "@shipeasy/sdk/server";
track(req.user.id, "{{SUCCESS_EVENT}}", { value: order.total });

// Browser — the user is already bound via configure()/identify()
import { track } from "@shipeasy/sdk/client";
track("{{SUCCESS_EVENT}}", { value: order.total });
```

> The bound `Client` does **not** expose `track` — it lives on the top-level
> facade / `Engine`. On the server, `track(userId, event, props?)` takes the
> user id; in the browser, `track(event, props?)` (the active user is implicit).

## Low-level `Engine` form

```ts
import { Engine } from "@shipeasy/sdk/server";
const engine = new Engine({ apiKey: process.env.SHIPEASY_SERVER_KEY! });
await engine.initOnce();

const { group, params } = engine.getExperiment(
  "hero_cta",
  { user_id: "u1" },              // user/attribute bag
  { primary_label: "Sign up" },   // default params
);
engine.track("u1", "{{SUCCESS_EVENT}}");
```

## Exposure logging

By default reading an experiment logs an exposure. To control exactly when the
exposure fires (e.g. log on render of the treatment, not on read), see
[Advanced → manual exposure](./advanced.md).
