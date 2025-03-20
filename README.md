# Mengelola UIProvider dan ThemeProvider di Next.js

Artikel ini membahas bagaimana mengintegrasikan **UIProvider** dan **ThemeProvider** dalam proyek Next.js menggunakan **next-themes**, serta bagaimana mengatur konfigurasi default dengan **defaultConfig()**.

## 1. Struktur Direktori

Berikut adalah struktur direktori yang digunakan:

```
/app
  ├── layout.tsx
  ├── page.tsx
  ├── providers.tsx
  ├── theme/
  │   ├── default/
  │   │   ├── home.tsx
  │   │   ├── landing.tsx
  ├── lib/
  │   ├── default.config.ts
  │   ├── redis.ts
  │   ├── auth.ts
  ├── db/
      ├── user.ts
```

## 2. Implementasi UIProvider

`UIProvider` menentukan tampilan **Home** atau **Landing** berdasarkan status login pengguna.

**providers.tsx:**

```tsx
"use client";

import { ThemeProvider } from "next-themes";
import { useEffect, useState } from "react";
import { defaultConfig } from "@/lib/default.config";

export default function Providers({ children }: { children: React.ReactNode }) {
  const [config, setConfig] = useState<any>(null);

  useEffect(() => {
    defaultConfig().then(setConfig);
  }, []);

  if (!config) return <p>Loading...</p>;

  return (
    <ThemeProvider attribute="class">
      {config.session ? <HomeComponent /> : <LandingComponent />}
    </ThemeProvider>
  );
}
```

## 3. ThemeProvider dan next-themes

Di dalam `layout.tsx`, kita bungkus `children` dengan `Providers` agar **tema** dan **UIProvider** berfungsi dengan baik.

**layout.tsx:**

```tsx
import Providers from "./providers";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="id">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

### Menggunakan Tema

Di dalam komponen toggle:

```tsx
import { useTheme } from "next-themes";
import { HiSun, HiMoon } from "react-icons/hi2";

export default function ToggleTheme() {
  const { theme, setTheme } = useTheme();

  return (
    <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
      {theme === "light" ? <HiMoon /> : <HiSun />}
    </button>
  );
}
```

## 4. Konfigurasi Default

File `default.config.ts` bertugas mengambil data awal seperti **session** dan **user** dari database.

**default.config.ts:**

```tsx
import { getUser } from "@/db/user";
import { authOptions } from "@/lib/auth";
import { getRedisClient } from "@/lib/redis";
import { getServerSession } from "next-auth";

export const defaultConfig = async () => {
  const redis = await getRedisClient();
  const cached = await redis.get("defaultConfig");

  if (cached) return JSON.parse(cached);

  const session = await getServerSession(authOptions);
  const username = session?.user?.name || null;

  const user = username ? await getUser(username) : null;

  const data = { session, user };

  await redis.set("defaultConfig", JSON.stringify(data), "EX", 300);

  return data;
};
```

## Kesimpulan

- **UIProvider** menentukan tampilan **Home** atau **Landing**.
- **ThemeProvider** dari `next-themes` digunakan untuk mengatur tema.
- **defaultConfig()** mengambil data awal dan menyimpannya di Redis untuk caching.
- **layout.tsx** memastikan semua provider terbungkus dengan benar.

Dengan pendekatan ini, tema dan UI dapat dikelola secara dinamis di Next.js.
