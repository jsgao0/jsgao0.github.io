---
title: "Next.js Edge vs Node.js: Middleware, Cookies, and Set-Cookie Best Practices"
date: 2025-09-26
author: "Anson"
tags: ["Next.js", "Edge", "Node.js", "Cookies", "Set-Cookie", "Web Dev"]
description: "Explaining the runtime differences between Edge and Node.js in Next.js, cookie handling, and best practices for working with Set-Cookie headers."
image: "2025/09/26/nextjs-cookie-set-get/cookie-in-nextjs.png"
og:
  title: "Next.js Edge vs Node.js: Middleware, Cookies, and Set-Cookie Best Practices"
  description: "Explaining the runtime differences between Edge and Node.js in Next.js, cookie handling, and best practices for working with Set-Cookie headers."
  image: "2025/09/26/nextjs-cookie-set-get/cookie-in-nextjs.png"
---

![Next.js Edge vs Node.js – Cookies Illustration](2025/09/26/nextjs-cookie-set-get/cookie-in-nextjs.png)

In Next.js 13/14 (App Router), the **runtime environment** defines what APIs you can use and how certain behaviors differ. While building unified login flows and tracking systems, I’ve run into several recurring issues around cookies and `Set-Cookie`. This post summarizes key points:  
1) Middleware runs on **Edge Runtime**, while pages/layouts/route handlers run on **Node.js Runtime**.  
2) How to access cookies in both environments, and how Node.js can read upstream and server `Set-Cookie`.  
3) Why you must use `response.headers.append` (not `set`) when writing multiple `Set-Cookie` values.  
4) What to watch out for when parsing `Set-Cookie`.

---

## 1) Execution Environments

- **`middleware.ts` / `middleware.js`** → Runs on **Edge Runtime**.  
  You get `NextRequest` and `NextResponse`, use Web Standard APIs (Headers, Request, Response), but no Node.js built-ins.

- **`app/**` pages, layouts, route handlers (e.g. `app/api/**/route.ts`)** → Run in **Node.js Runtime by default**.  
  You can use `next/headers` (`cookies()`, `headers()`), Node’s `fetch` behavior (can read `set-cookie`), and Node features.

- You can override runtime for specific routes with:  
  ```ts
  export const runtime = 'edge';
  ```

---

## 2) Reading Cookies in Edge vs Node.js

### In **Middleware (Edge)**

```ts
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(req: NextRequest) {
  // Read
  const token = req.cookies.get('access_token')?.value;

  // Write
  const res = NextResponse.next();
  res.cookies.set({
    name: 'from_mw',
    value: '1',
    httpOnly: true,
    path: '/',
    sameSite: 'Lax',
  });
  return res;
}
```

### In **Node.js Runtime (pages/server actions/route handlers)**

```ts
import { cookies, headers } from 'next/headers';

export async function GET() {
  const cookieStore = cookies();
  const access = cookieStore.get('access_token')?.value;

  const h = headers();
  const auth = h.get('authorization');

  return new Response(JSON.stringify({ access, auth }), {
    headers: { 'content-type': 'application/json' },
  });
}
```

### Reading upstream `Set-Cookie` in **Node.js**

Unlike browsers, **Node.js `fetch` can access `set-cookie`** from upstream responses:

```ts
const upstream = await fetch('https://auth.example.com/login', {
  method: 'POST',
  body: JSON.stringify({ username, password }),
  headers: { 'content-type': 'application/json' },
});

// ✅ Available in Node.js
const setCookieHeader = upstream.headers.get('set-cookie');
```

### Reading server `Set-Cookie` via `next/headers`

In Node.js runtime, `headers()` also exposes response-bound headers.  
That means you can read **cookies that your server is about to set** with `headers().get('set-cookie')`.

```ts
// app/api/demo/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const res = NextResponse.json({ msg: 'ok' });
  res.cookies.set('a', '1', { httpOnly: true, path: '/' });
  res.cookies.set('b', '2', { httpOnly: true, path: '/' });
  return res;
}

// app/api/check/route.ts
import { headers } from 'next/headers';

export async function GET() {
  const h = headers();
  const setCookieHeader = h.get('set-cookie');

  return Response.json({
    'set-cookie': setCookieHeader,
  });
}
```

Output:

```json
{
  "set-cookie": "a=1; Path=/; HttpOnly; SameSite=Lax, b=2; Path=/; HttpOnly; SameSite=Lax"
}
```

👉 **Key point**:  
- `headers().get('cookie')` → gives you cookies from the **client request**.  
- `headers().get('set-cookie')` → gives you cookies that the **server response is about to send** (only in Node.js runtime).  

---

## 3) Writing `Set-Cookie`: Use `append`, not `set`

HTTP allows multiple `Set-Cookie` headers. Using `headers.set('set-cookie', ...)` **overwrites previous cookies**. Always use `append`.

```ts
// ❌ Overwrites
res.headers.set('set-cookie', 'a=1; Path=/; HttpOnly');
res.headers.set('set-cookie', 'b=2; Path=/; HttpOnly'); // 'a' is gone

// ✅ Correct
res.headers.append('set-cookie', 'a=1; Path=/; HttpOnly');
res.headers.append('set-cookie', 'b=2; Path=/; HttpOnly');
```

> 💡 Tip: Prefer `response.cookies.set()` where available, as it handles multiple cookies automatically.

```ts
import { NextResponse } from 'next/server';

const res = NextResponse.json({ ok: true });
res.cookies.set('a', '1', { httpOnly: true, path: '/' });
res.cookies.set('b', '2', { httpOnly: true, path: '/' });
```

---

## 4) Parsing `Set-Cookie`: Common Pitfalls

The tricky part is that:  
- A response may include **multiple `Set-Cookie` headers**.  
- Inside a cookie, the `Expires` attribute itself contains a **comma** (e.g., `Fri, 01 Jan 2038 00:00:00 GMT`).  
- So **you cannot just split on commas**.

### Recommendations

1. **Don’t `split(',')` blindly.** You’ll break cookies with `Expires`.  
2. **Use a library like `set-cookie-parser`**:
   - `splitCookiesString()` splits multiple cookies safely.  
   - `parseString()` parses each into structured objects.  
3. **Attribute rules:**
   - `Max-Age` overrides `Expires`.  
   - `SameSite=None` requires `Secure`.  
   - Be mindful of defaults for `Path` and `Domain`.  
   - Encode values properly if they contain special characters.  
4. **Forwarding cookies:**  
   - When proxying upstream logins, forward all `Set-Cookie` headers with `append`.  
   - If you need to rewrite cookies, parse and reconstruct carefully.

### Example: Robust parsing & forwarding (Node.js)

```ts
import setCookieParser from 'set-cookie-parser';

export async function POST() {
  const upstream = await fetch('https://auth.example.com/login', { method: 'POST' });
  const rawSetCookie = upstream.headers.get('set-cookie') || '';

  const cookieStrings = setCookieParser.splitCookiesString(rawSetCookie);
  const cookies = cookieStrings.map((c) => setCookieParser.parseString(c));

  const res = new Response(await upstream.text(), {
    status: upstream.status,
    headers: { 'content-type': upstream.headers.get('content-type') ?? 'text/plain' },
  });

  for (const c of cookies) {
    const attrs = [
      `${c.name}=${c.value}`,
      `Path=${c.path ?? '/'}`,
      c.domain ? `Domain=${c.domain}` : '',
      c.expires ? `Expires=${c.expires.toUTCString()}` : '',
      typeof c.maxAge === 'number' ? `Max-Age=${c.maxAge}` : '',
      c.secure ? 'Secure' : '',
      c.httpOnly ? 'HttpOnly' : '',
      c.sameSite ? `SameSite=${c.sameSite}` : '',
    ].filter(Boolean);

    res.headers.append('set-cookie', attrs.join('; '));
  }

  return res;
}
```

---

## Takeaways

- **middleware = Edge**, **pages/layouts/handlers = Node.js by default**.  
- **Reading cookies** works in both; **reading `Set-Cookie`** works in Node.js via both `fetch` and `headers().get('set-cookie')`.  
- **Writing cookies**: use `cookies.set()` or `headers.append`.  
- **Parsing `Set-Cookie`**: never split on commas, use a parser, and handle attributes properly.  

Following these patterns avoids the classic “Why did my cookie disappear?” or “Why does login only work on one device?” headaches.  
