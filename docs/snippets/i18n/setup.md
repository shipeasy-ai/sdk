Wire the i18n loader via the SSR bootstrap (no separate i18n init — it rides `shipeasy()`). The loader serves the `{{PROFILE}}` profile and hydrates `window.i18n`.

```tsx
// app/layout.tsx — Next.js root layout (React Server Component)
import { shipeasy } from "@shipeasy/sdk/server";

const se = await shipeasy({ serverKey: process.env.SHIPEASY_SERVER_KEY ?? "" });
const boot = se.getBootstrapData({
  clientKey: process.env.NEXT_PUBLIC_SHIPEASY_CLIENT_KEY, // profile: {{PROFILE}}
});

// Render REAL <script> elements:
<script src={boot.bootstrap.src} {...boot.bootstrap.attrs} />;
{boot.i18nLoader && <script src={boot.i18nLoader.src} {...boot.i18nLoader.attrs} />}
```
