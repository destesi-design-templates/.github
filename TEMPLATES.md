# Writing a Destesi Design template

Every repository in this organization is one template: a starting design that
Destesi Design seeds into a merchant's own repository. From that moment the
merchant's copy is ordinary source — it diverges, and nothing here ever
rewrites it.

## What a template is

A template is an **overlay** on a base Design owns. The `backend` in its
manifest picks the base:

| `backend` | Base | Who can use it |
|---|---|---|
| `commerce` | Design's shop base: the shell (`App.jsx`, `ShopChrome.jsx`), the section runtime (`sections.jsx`), the checkout, `shop.css`, and `src/shop/` — the vendored Commerce client (`@destesi/shop-contract v1`) | only the workspace's **shop project**. Its backend is Destesi Commerce, which reads catalog and stock from Destesi Inventory; the page reaches it through the same-origin `shop-api/` its host proxies |
| `none` | Design's plain React + Vite + Tailwind starter | any project |

```
manifest.json     id, name, description, best_for, backend, es {name, description, best_for}, hero_compact
src/**            overlaid on the base's src/
public/assets/*   images the pages or theme.css use as assets/<name>
skills/*.md       flat; each starts with "# Skill: " — seeded beside Design's own skills
README.md, LICENSE, .github/   the repository's own, never seeded
```

`id` is the repository name and never changes. `es` is the merchant-facing copy
in Spanish. `hero_compact` (commerce only) is true exactly when `Home()`'s hero
is the compact variant, so the shop host skips preloading a banner the page
never draws.

## Refused paths

Design's tests refuse a commit that writes any of these, before it is pinned:

- `src/shop/**` — the Commerce client, one copy for every template.
- `src/template.json` — the retired page document.
- For `commerce`: `src/App.jsx`, `src/ShopChrome.jsx`, `src/Checkout.jsx`,
  `src/sections.jsx`, `src/main.jsx`. The shell owns the routes Commerce links
  buyers to (`/`, `/producto/<id>`, `/legal/<doc>`); the runtime is what the
  visual editor reads. A new kind of section goes into Design's block library
  once, not into a template.
- A skill named like one of Design's (`accessibility`, `color`, `components`,
  `copy`, `ecommerce`, `landing`, `layout`, `motion`, `typography`). Name yours
  after the template: `skills/<id>-style.md`.

## Commerce templates

**`src/pages.jsx`** keeps one shape, because the section editor rewrites it:
every block a flat child of `<main>`, a `<Section section={{ id, type, props }} />`
object literal, ids unique per page, no `.map(`. Its copy is buyer-facing
Spanish and **goes live on the merchant's shop until they edit it**, so it is
generic for the vertical and promises nothing a merchant may not keep — no
delivery times, return windows, warranty terms, free shipping, discounts,
scarcity, ratings or claims about how the products are made ("hecho a mano")
— and never `testimonials`. No block sets `image_url`, and nothing names
`shop-api/products/<id>`: a template ships to every shop.

**`src/theme.css`** sits in `@layer components` after `shop.css`, sets the
`--shop-*` tokens the base reads everywhere, then restyles by
`section[data-section-type="…"]`. `--shop-accent` is the merchant's brand colour
on a live shop, so no design depends on the demo's.

**`src/demo.js`** is the only demo-only file (brand, products, COP prices,
photos). The base reads it only when `shop-api` answers 404: a template demo
host (`demo-<id>.shop.destesi.io`) or a project with no shop. The photos are
generated for the template and ship in the merchant's repo.

The catalog, prices, stock, promotions, checkout and payment rules are never a
template's: Commerce decides them and Inventory owns product identity and
stock. A template draws them, it never re-derives them.

## Images

WebP, PNG, JPEG or GIF (never SVG), each at most 400 KB and 2 MB per template,
and every one used: imported (`./demo/p1.webp` in `src/demo.js`) or named
(`assets/texture.webp`). Design's loader refuses anything else.

## Designing in Design (staff)

Templates → a template → **Open in workbench** opens it as a project of its
own, in demo mode, with the agent, the sections rail and hand editing; it is
never published. **Propose as template** sends the design back here as one
pull request (files a template may not change are listed as left out). Review
and merge it like any other.

## Developing locally

From a Destesi checkout, passing the absolute path of this repository's clone:

```
make -C apps/design/api template-dev DIR=<absolute path to this repo>
```

lays the overlay on the base and runs the Vite dev server in demo mode;
every save in this repository hot-reloads.

## Getting a change to merchants

Merging to `main` publishes nothing. Design pins every template to a reviewed
commit in `apps/design/api/internal/templates/LOCK.json`
(`make -C apps/design/api sync-templates`), which refuses a commit that is not
on `main` and prints what changed. The checks above run with the lock bump:
`go test ./internal/templates/ ./internal/connect/` in design-api, and
design-api refuses to boot on a template that fails to load. The pin is the
trust boundary: template code runs in merchants' sandboxes and is served to
real buyers.
