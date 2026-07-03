# Deploying to IONOS (under dteman.com)

The whole tool is **one self-contained file** — `index.html` (≈51 KB, no build, no backend, no
external dependencies). Hosting = "put this file somewhere a web server serves it." Pick one
of the options below. Keep the filename `index.html` so the folder URL serves it directly.

---

## Option A (recommended) — install as a path under dteman.com, via Plesk
→ **`https://dteman.com/elevator/`**

IONOS manages most hosting plans with **Plesk**; this is the simplest route for a subpage.

1. Open **Plesk** (IONOS control panel → your hosting package → **Open Plesk**, or browse to
   `https://<your-server>:8443` and sign in).
2. **Websites & Domains** → expand **dteman.com**.
3. Click **File Manager**.
4. Open the document root **`httpdocs`** (the folder that holds dteman.com's existing site
   files — its `index.*` lives here).
5. **`+`  →  Create Directory** → name it **`elevator`**. This folder name becomes the URL path.
6. Open `httpdocs/elevator`, click **Upload**, and choose **`index.html`** (drag-and-drop also
   works).
7. Visit **`https://dteman.com/elevator/`** — Plesk/Apache serves `index.html` for the folder
   automatically; no rules or config needed.
8. HTTPS (if not already on): **dteman.com → SSL/TLS Certificates → Install** a free
   **Let's Encrypt** certificate — it covers the subfolder too.

To update later: re-upload `index.html` into `httpdocs/elevator`, overwriting the old one.

Notes:
- One static file — no PHP, no database, no build — so there is nothing else to configure.
- On a few setups the web root is `httpdocs/` under the domain, or a per-domain folder shown
  right in Plesk's File Manager; use whichever folder Plesk shows as dteman.com's document root.

---

## Option B — subdomain (cleaner URL), via Plesk
→ e.g. **`https://elevator.dteman.com/`**

1. Plesk → **Websites & Domains** → **Add Subdomain**.
2. Subdomain name **`elevator`**, parent domain **dteman.com**. Plesk pre-fills a document
   root (e.g. `elevator`); keep it.
3. Open that subdomain's **File Manager** → its document root → **Upload** `index.html`.
4. **SSL/TLS Certificates** → install a free **Let's Encrypt** certificate for the subdomain.
5. Visit **`https://elevator.dteman.com/`**.

---

## Option C — IONOS Deploy Now (Git-based, auto-deploy on push)
Good if you want updates to go live by `git push` and you're OK with a separate project.

1. Put this folder in a Git repo (e.g. GitHub).
2. IONOS → **Deploy Now** → **Create project** → connect the repo.
3. Framework preset: **Static / "None"**; output directory: the repo root (where `index.html`
   is). No build command needed.
4. Deploy Now gives a URL + free SSL; you can attach `elevator.dteman.com` as a custom domain.

---

## Alternatives outside IONOS (mentioned for completeness)
A single static file is the easiest thing to host anywhere:
- **Cloudflare Pages / GitHub Pages** — free, push-to-deploy; then CNAME a subdomain of
  dteman.com to it. Lowest-effort if IONOS file management is fussy.
- **Kamatera VPS + Caddy** — only worth it if you want a full server; overkill for one file.

---

## Notes
- It's a static page, so there is nothing to secure server-side and no runtime cost.
- Everything (charts, math, print/PDF) runs in the browser; works offline once loaded.
- The page `<title>`, description, and social-share preview tags are already set, so the link
  previews sensibly when shared.
- **Bilingual (Hebrew/English):** not built yet — when ready it stays a single `index.html`,
  so the deploy steps above don't change.
