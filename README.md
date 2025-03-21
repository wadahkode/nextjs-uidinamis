# Membangun UI Dinamis dengan ThemeProvider & UIProvider di Next.js

## Pendahuluan
Dalam proyek Next.js yang menggunakan app/ directory, kita ingin mengelola dua aspek utama secara bersamaan:
1. Menentukan tampilan antara Home atau Landing Page berdasarkan sesi pengguna.
2. Menerapkan tema (dark/light mode) dengan sistem yang fleksibel.

Untuk itu, kita akan mengintegrasikan:
- defaultConfig() → Mengelola sesi dan data awal dengan caching Redis.
- UIProvider → Memilih komponen UI berdasarkan sesi pengguna.
- ThemeProvider → Mengatur tema yang dipilih pengguna.

## Struktur Direktori
Agar kode tetap rapi dan modular, kita akan menggunakan struktur berikut:

```
/my-project
├── /app
│   ├── /(site)
│   │   ├── /home
│   │   │   ├── page.tsx
│   │   │   └── Home.tsx
│   │   ├── /landing
│   │   │   ├── page.tsx
│   │   │   └── Landing.tsx
│   ├── layout.tsx
│   ├── page.tsx
│── /components
│   ├── UIProvider.tsx
│   ├── ThemeProvider.tsx
│── /lib
│   ├── auth.ts
│   ├── redis.ts
│── /db
│   ├── user.ts
│── default.config.ts
```

## 1. Membuat defaultConfig() untuk Mengambil Data Awal
File: `default.config.ts`
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

  const data = {
    session,
    user
  };

  await redis.set("defaultConfig", JSON.stringify(data), "EX", 300);

  return data;
};
```

## 2. Membuat ThemeProvider untuk Mengatur Tema
File: `ThemeProvider.tsx`
```tsx
"use client";

import { ThemeProvider as NextThemesProvider } from "next-themes";

export default function ThemeProvider({ children }) {
  return (
    <NextThemesProvider attribute="class" enableSystem={true} enableColorScheme={false}>
      {children}
    </NextThemesProvider>
  );
}
```

## 3. Membuat UIProvider untuk Menentukan Komponen yang Digunakan
File: `UIProvider.tsx`
```tsx
"use client";

import { useEffect, useState } from "react";
import { useTheme } from "next-themes";
import Home from "@/app/(site)/home/Home";
import Landing from "@/app/(site)/landing/Landing";

export default function UIProvider({ data }) {
  const { theme } = useTheme();
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  useEffect(() => {
    if (data?.session) {
      setIsAuthenticated(true);
    }
  }, [data]);

  return (
    <div className={theme === "dark" ? "dark" : ""}>
      {isAuthenticated ? <Home /> : <Landing />}
    </div>
  );
}
```

## Menyesuaikan app/page.tsx untuk Menggunakan UIProvider
File: `page.tsx`
```tsx
import UIProvider from "@/components/UIProvider";
import defaultConfig from "@/default.config";

export default async function Page() {
  const data = await defaultConfig();
  return <UIProvider data={data} />;
}
```

## Menyesuaikan app/layout.tsx untuk Struktur yang Konsisten
File: `layout.tsx`
```tsx
import ThemeProvider from "@/components/ThemeProvider";

export default function Layout({ children }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

## Kesimpulan
Dengan pendekatan ini:
- `defaultConfig()` mengelola data sesi dan caching.
- `UIProvider` menentukan apakah pengguna melihat Home atau Landing.
- `ThemeProvider` memastikan tema tetap konsisten tanpa konflik dengan `next-themes`.
- Struktur proyek tetap rapi dan scalable.

Kini, tampilan dan tema dapat menyesuaikan secara otomatis berdasarkan status login pengguna! 🎉
