Configure once, then read a flag per user with a bound `Client`.

```ts
import { configure, Client } from "@shipeasy/sdk/server"; // or "@shipeasy/sdk/client"

configure({ apiKey: process.env.SHIPEASY_SERVER_KEY! });

const flags = new Client(currentUser);
if (flags.getFlag("{{RESOURCE_NAME}}")) {
  // ship it
}
```
