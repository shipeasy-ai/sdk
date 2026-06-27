Read a typed dynamic config (with a default when the key is absent).

```ts
import { configure, Client } from "@shipeasy/sdk/server"; // or "@shipeasy/sdk/client"

configure({ apiKey: process.env.SHIPEASY_SERVER_KEY! });

const flags = new Client(currentUser);
const cfg = flags.getConfig<{ max: number }>("{{RESOURCE_NAME}}", {
  defaultValue: { max: 50 },
});
```
