Render a translated label with `i18n.t(key, fallback, variables?)`. The fallback is the source string; it shows until the loader resolves.

```ts
import { i18n } from "@shipeasy/sdk/client";

i18n.t("checkout.cta", "Place order");
i18n.t("cart.count", "{count} items", { count: cart.length });
```
