Read an experiment's params, then record the conversion event.

```ts
import { configure, Client, track } from "@shipeasy/sdk/server"; // or "@shipeasy/sdk/client"

configure({ apiKey: process.env.SHIPEASY_SERVER_KEY! });

const flags = new Client(currentUser);
const { params } = flags.getExperiment("{{RESOURCE_NAME}}", {
  primary_label: "Sign up",
});

render(params.primary_label);

// On conversion (server: pass the user id; browser: track(event, props?)):
track(currentUser.id, "{{SUCCESS_EVENT}}");
```
