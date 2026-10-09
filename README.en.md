<div align="center">
  <img src=".github/assets/icon.png" width="108" alt="personal-homepage" />
  <h1>personal-homepage</h1>
  <p><b>The comment backend for the 启明星-长庚星 blog</b><br /><sub>A Waline server deployed on Vercel</sub></p>
  <p>
    <a href="https://personal-homepage-phi-eight.vercel.app"><img src="https://img.shields.io/badge/live-Vercel-000000" alt="Live on Vercel"></a>
    <img src="https://img.shields.io/badge/Waline-comments-00B96B" alt="Waline">
    <img src="https://img.shields.io/badge/PostgreSQL-storage-4169E1" alt="PostgreSQL">
    <img src="https://img.shields.io/badge/Node.js-%E2%89%A518-339933" alt="Node >= 18">
  </p>
  <p>
    <a href="README.md">简体中文</a> ·
    <b>English</b>
  </p>
</div>

> ⚠️ **The repository name is misleading**: this is **not** a personal homepage — it is the **comment server** for the blog [启明星-长庚星](https://sunfleeting-debug.github.io/).
> The name is a leftover (the original plan was to host Waline under a `personal-homepage` repo); the contents have always been Waline.

## At a glance

| | |
| --- | --- |
| **What it is** | The server side of the [Waline](https://waline.js.org/) comment system, running as a Vercel serverless function |
| **Live URL** | <https://personal-homepage-phi-eight.vercel.app> |
| **Who uses it** | The blog's `valaxy/demo/yun/site-meta.ts`, via `WALINE_SERVER_URL` |
| **Storage** | PostgreSQL (via `POSTGRES_*` env vars) |
| **Contents** | Three effective files: `index.cjs` · `vercel.json` · `robots.txt` |
| **Indexing** | `robots.txt` is `Disallow: /` — a comment API should not be indexed |

---

## 📁 What's in here

```text
.
├── index.cjs         Waline app entry (add plugins, write a postSave hook)
├── vercel.json       builds and routing: everything but robots.txt rewrites to index.cjs
├── robots.txt        User-agent: * / Disallow: /  — do not index
├── package.json      the single dependency: @waline/vercel
└── .env.example      PostgreSQL connection template
```

The whole of `index.cjs` is a few lines — Waline's storage, auth and admin dashboard all come from `@waline/vercel`; this file just mounts it:

```js
const Application = require('@waline/vercel');

module.exports = Application({
  plugins: [],
  async postSave(comment) {
    // do what ever you want after comment saved
  },
});
```

Two details in `vercel.json` are worth knowing:

- **`rewrites`**: `/((?!robots\.txt$).*)` → `index.cjs`, i.e. **every path except `robots.txt`** is handed to Waline (which brings its own admin dashboard and API routes).
- **`includeFiles`**: explicitly bundles `@mathjax/mathjax-newcm-font`, `mhchemparser` and `ip2region/data` — these are **read as files at runtime**, so Vercel's bundler can't track them; without this the function is missing files at runtime.

---

## 🚀 Deploy

### On Vercel

```bash
npm i -g vercel
vercel            # follow the prompts, or Import this repo from the Vercel dashboard
```

### Environment variables

Set the PostgreSQL connection in the Vercel project's **Settings → Environment Variables** (see [`.env.example`](.env.example)):

| Variable | Purpose |
| --- | --- |
| `POSTGRES_HOST` / `POSTGRES_PORT` | database host and port |
| `POSTGRES_DATABASE` | database name |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` | credentials |
| `POSTGRES_PREFIX` | table name prefix |
| `POSTGRES_SSL` | whether to use SSL |

Once deployed, point the blog at it:

```ts
// valaxy/demo/yun/site-meta.ts
export const WALINE_SERVER_URL = 'https://<your-project>.vercel.app'
```

---

## 🔧 Customising

- **Do something after a comment is saved**: implement the `postSave(comment)` hook in `index.cjs` (notify, moderate, sync, …).
- **Add plugins**: pass them into `Application({ plugins: [...] })`.
- **Admin dashboard**: Waline ships one; the first registered user becomes the administrator (see the [Waline docs](https://waline.js.org/)).

---

## 📄 License

This repository **does not ship an open-source license** — it is a deployment config; the comment system itself remains under [Waline](https://github.com/walinejs/waline)'s MIT license.
