# Stroma.js

**Lightweight SEO meta tag generator. One call. Every platform covered.**

[![npm](https://img.shields.io/npm/v/stroma)](https://npmjs.com/package/stroma)
[![gzip size](https://img.shields.io/bundlephobia/minzip/stroma)](https://bundlephobia.com/package/stroma)
[![license](https://img.shields.io/npm/l/stroma)](LICENSE)

---

You pass in your page data once. Stroma writes all of the following into `<head>`:

- `<title>` and `meta[name=description]`
- Open Graph tags (`og:title`, `og:image`, `og:type`, …) — for Facebook, WeChat, Slack, etc.
- Twitter Card tags (`twitter:card`, `twitter:image`, …)
- JSON-LD structured data (`WebPage`, `Article`, `Product`, `BreadcrumbList`)
- PWA / Apple Web App hints
- `link[rel=canonical]`

**No dependencies. No framework lock-in. Works in vanilla JS, React, Vue, Svelte, and on the server.**

---

## Install

```bash
npm install stroma
```

Or via CDN (no build step):

```html
<script src="https://cdn.jsdelivr.net/npm/stroma/dist/stroma.umd.js"></script>
```

---

## Quick start

```js
import Stroma from 'stroma';

Stroma.init({
  title:       'My Product Page',
  description: 'The best product in the world, at an unbeatable price.',
  url:         'https://example.com/product/widget',
  image:       'https://example.com/images/widget-og.png',
  author:      'Jane Doe',
  siteName:    'Example Co.',
  themeColor:  '#2563eb',
  keywords:    ['widget', 'example', 'product'],
  schema:      'product',
  price:       49.99,
});
```

That one call produces ~25 tags, covering Google, Facebook, X (Twitter), WeChat, and structured data crawlers.

---

## API

### `Stroma.init(options)` → `Stroma`

Writes all tags to the DOM. Clears any tags from a previous call first.

| Option | Type | Default | Description |
|---|---|---|---|
| `title` | `string` | — | **Required.** Page title. Truncated to `maxTitleLength`. |
| `description` | `string` | — | **Required.** Page description. Truncated to `maxDescriptionLength`. |
| `url` | `string` | `location.href` | Canonical URL. |
| `image` | `string` | first `<img>` in DOM | Absolute URL of the share image. |
| `imageAlt` | `string` | `title` | Alt text for the share image. |
| `imageWidth` | `string` | `'1200'` | OG image width. |
| `imageHeight` | `string` | `'630'` | OG image height. |
| `siteName` | `string` | — | Populates `og:site_name`. |
| `author` | `string` | — | Populates `meta[name=author]` and `article:author`. |
| `keywords` | `string \| string[]` | — | Meta keywords. Arrays are joined with `', '`. |
| `locale` | `string` | `'en_US'` | OG locale. |
| `robots` | `string` | `'index, follow'` | Robots directive. |
| `themeColor` | `string` | — | Browser theme color (`#rrggbb`). |
| `canonical` | `string` | `url` | Explicit canonical if different from `url`. |
| `ogType` | `string` | `'website'` | Open Graph type. |
| `twitterCard` | `string` | auto | `'summary'` or `'summary_large_image'` (auto-selected when image present). |
| `twitterSite` | `string` | — | Twitter @handle for the site. |
| `twitterCreator` | `string` | — | Twitter @handle for the author. |
| `schema` | `string` | `'webpage'` | JSON-LD schema: `'webpage'`, `'article'`, `'product'`, `'breadcrumb'`. |
| `datePublished` | `string` | — | ISO 8601 date. Used in Article schema. |
| `dateModified` | `string` | — | ISO 8601 date. Falls back to `datePublished`. |
| `price` | `number` | — | Product price. Used in Product schema. |
| `priceCurrency` | `string` | `'USD'` | ISO 4217 currency. |
| `breadcrumb` | `Array<{name, url}>` | — | Items for BreadcrumbList schema. |
| `pwa` | `boolean` | `false` | Inject Apple / PWA meta tags. |
| `appName` | `string` | `title` | PWA display name. |
| `logo` | `string` | — | Publisher logo URL for Article schema. |
| `maxTitleLength` | `number` | `60` | Override title truncation limit for this call. |
| `maxDescriptionLength` | `number` | `160` | Override description truncation limit for this call. |

---

### `Stroma.update(patch)` → `Stroma`

Merges `patch` into the last config and re-applies all tags. Efficient for SPA route changes.

```js
// React Router / Vue Router — on route change:
Stroma.update({
  title:       newRoute.meta.title,
  description: newRoute.meta.description,
  url:         window.location.href,
});
```

---

### `Stroma.renderToString(options)` → `string`

**Server-Side Rendering.** Returns a raw HTML string of all tags. No DOM access, no side-effects. Works in Node.js, Deno, Cloudflare Workers, and any non-browser runtime.

```js
// Node.js / Express
import Stroma from 'stroma';

app.get('/product/:id', async (req, res) => {
  const product = await db.getProduct(req.params.id);

  const headTags = Stroma.renderToString({
    title:       product.name,
    description: product.summary,
    image:       product.imageUrl,
    url:         `https://example.com/product/${product.id}`,
    schema:      'product',
    price:       product.price,
  });

  res.send(`
    <html>
      <head>${headTags}</head>
      <body>...</body>
    </html>
  `);
});
```

```js
// Next.js App Router — inject via a Server Component
import Stroma from 'stroma';

export default function Layout({ children }) {
  const tags = Stroma.renderToString({ title: 'My App', description: '...' });
  return (
    <html>
      <head dangerouslySetInnerHTML={{ __html: tags }} />
      <body>{children}</body>
    </html>
  );
}
```

---

### `Stroma.reset()` → `Stroma`

Removes all Stroma-injected tags from `<head>`.

---

### `Stroma.getConfig()` → `Object`

Returns a snapshot of the last resolved config, including computed fields (`titleFull`, `descriptionFull`).

---

### `Stroma.defaults(patch?)` → `Stroma | Object`

Read or update module-level defaults that apply to every subsequent `init()` / `renderToString()` call, unless overridden in the individual options.

```js
// Set once at app startup
Stroma.defaults({
  maxTitleLength:       70,   // some platforms support up to 70
  maxDescriptionLength: 155,
  robots:               'index, follow',
});

// Read current defaults
console.log(Stroma.defaults()); // → { maxTitleLength: 70, ... }
```

---

### `Stroma.breadcrumb(items)` → `Stroma`

Injects a standalone `BreadcrumbList` JSON-LD block (client-side only).  
For SSR, use `renderToString({ schema: 'breadcrumb', breadcrumb: [...] })`.

```js
Stroma.breadcrumb([
  { name: 'Home',    url: '/' },
  { name: 'Blog',    url: '/blog' },
  { name: 'My Post', url: '/blog/my-post' },
]);
```

---

## Framework recipes

### React (SPA)

```jsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';
import Stroma from 'stroma';

function SeoManager({ title, description, image }) {
  const { pathname } = useLocation();

  useEffect(() => {
    Stroma.init({ title, description, image, url: window.location.href });
  }, [pathname, title, description, image]);

  return null;
}
```

### Vue 3 (SPA)

```js
// router/index.js
import Stroma from 'stroma';

router.afterEach((to) => {
  Stroma.update({
    title:       to.meta.title,
    description: to.meta.description,
    url:         window.location.href,
  });
});
```

### Next.js (App Router, SSR)

```jsx
// app/layout.jsx
import Stroma from 'stroma';

export default function RootLayout({ children }) {
  const tags = Stroma.renderToString({
    title:    'My Site',
    siteName: 'My Site',
    robots:   'index, follow',
  });

  return (
    <html>
      <head dangerouslySetInnerHTML={{ __html: tags }} />
      <body>{children}</body>
    </html>
  );
}
```

---

## How cleanup works

Every tag Stroma writes gets a `data-stroma` attribute. On `init()`, `update()`, and `reset()`, all elements carrying that attribute are removed before new tags are written. This prevents duplicate or stale tags from accumulating during SPA navigation.

Tags that Stroma did **not** write (e.g. a `<title>` already in your HTML template) are never touched.

---

## TypeScript

Stroma ships a `stroma.d.ts` declaration file. You get full autocompletion and type-checking with no additional setup.

```ts
import Stroma, { StromaOptions } from 'stroma';

const opts: StromaOptions = {
  title:       'Typed page',
  description: 'Full IntelliSense for every option.',
};

Stroma.init(opts);
```

---

## Building from source

```bash
git clone https://github.com/your-org/stroma.git
cd stroma
npm install
npm run build   # outputs ESM + CJS + .d.ts to /dist
npm test        # Vitest + happy-dom
```

---

## License

MIT © Ansel S
