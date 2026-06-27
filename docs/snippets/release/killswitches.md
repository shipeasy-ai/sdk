Read a kill switch (not user-bound — global on/off).

```ts
import { configure, Client } from "@shipeasy/sdk/server"; // or "@shipeasy/sdk/client"

configure({ apiKey: process.env.SHIPEASY_SERVER_KEY! });

const flags = new Client(currentUser);
if (flags.getKillswitch("{{RESOURCE_NAME}}")) {
  // killed — short-circuit the feature
}
```
