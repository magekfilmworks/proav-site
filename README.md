# ProAV Podcast — Website

Plain hand-written HTML/CSS/JS. No framework, no build step, no
`node_modules` — what you see in this repo is exactly what gets served.

## Structure

```
index.html      — homepage
contact.html    — contact page
images/         — merch product photos (WebP)
og-image.png    — social share image
robots.txt      — crawl rules
sitemap.xml     — sitemap
amplify.yml     — tells Amplify there's no build step, and explicitly
                  lists which files get served (so README.md,
                  LAUNCH-CHECKLIST.md, etc. stay out of the live site)
```

## Local development

There's nothing to install or run. Open `index.html` directly in a
browser, or serve the folder with any static file server if you want
clean-URL routing (`/contact` instead of `/contact.html`) to work locally
the same way it does in production:

```
python3 -m http.server 8000
```

## Branch strategy (for Amplify)

This repo is meant to be connected to a **single Amplify app** with two
branches, rather than two separate apps:

- **`main`** — the real, full site. Connect this branch in Amplify and
  leave it on its default `*.amplifyapp.com` URL, with **Access
  Control** (App settings → Access control) turned ON — this is your
  private preview link for reviewing changes before they go live.

- **`splash`** — just the "Coming Fall/Winter 2026" splash page. Connect
  this branch too, and assign your **custom domain**
  (`proavpodcast.fm`) to it in Amplify's Domain management. This is
  what the public sees pre-launch.

Every `git push` to either branch triggers an automatic Amplify build
and deploy — no more manual zip uploads.

## Launch day

Once the real site is ready to go public, there's no file swapping
needed — just repoint the domain:

1. In Amplify → **Domain management**, change which branch
   `proavpodcast.fm` points to: from `splash` → `main`.
2. Turn OFF Access Control on `main` (App settings → Access control).
3. Everything else — `robots.txt`, sitemap, canonical URLs — is already
   set up correctly on `main` for a public launch.

See `LAUNCH-CHECKLIST.md` for the full pre-flight list.

## Clean URLs

`contact.html` is requested as `/contact` throughout the site. Amplify
needs one rewrite rule to serve it (App settings → Rewrites and
redirects):

| Source     | Target          | Type              |
|------------|-----------------|-------------------|
| `/contact` | `/contact.html` | `200 (Rewrite)`   |

This is an app-level setting — set it once and it applies across all
branches connected to the app.

## Domains

`proavpodcast.fm` is canonical — it's what `robots.txt`, `sitemap.xml`,
and every canonical/OG URL tag in this site point to.

`proavpodcast.com` is a secondary domain that should 301 redirect to
`https://proavpodcast.fm`. Set this up at the **domain registrar**
level (GoDaddy/Namecheap/Route 53/wherever `.com` is registered) using
its built-in domain forwarding feature — do NOT try to configure this
as a second custom domain + redirect rule inside Amplify. Amplify's
redirect rules operate within a single app/domain; pointing one fully
custom domain at a completely different one is a known-broken pattern
there. `.com` never needs to be connected to Amplify at all.

Both domains are hosted in Route 53, so the redirect is set up as
Route 53 + CloudFront + a CloudFront Function — see
`DOMAIN-REDIRECT-SETUP.md` for the full step-by-step.

**Email is the opposite of the website:** use `.com` for all internal
addresses (e.g. `contact@proavpodcast.com`), not `.fm` — `.fm`
addresses have had deliverability issues. The contact form's
FormSubmit endpoint and the mailto link on the contact page both point
to the `.com` address.
