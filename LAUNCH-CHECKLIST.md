# ProAV Launch Day Checklist (Git + Amplify branch deploys)

Context: `proavpodcast.fm` currently points to the `splash` branch
(public, indexable). The real site lives on the `main` branch, deployed
to the same Amplify app's default URL, behind Access Control, for
internal review only.

## Before you start

- [ ] Confirm `main` is final — episode links, guest names, merch
      prices, contact info, whatever changed since it went behind the
      preview gate.
- [ ] Confirm `main`'s last deploy succeeded (Amplify console → the
      `main` branch → build history).

## Launch steps

1. **Repoint the custom domain**
   Amplify console → your app → **Domain management** → edit the
   domain association → change `proavpodcast.fm` from the `splash`
   branch to the `main` branch. Amplify handles the cutover; no new
   deploy needed.

2. **Turn off Access Control on `main`**
   App settings → **Access control** → disable it for the `main`
   branch. Otherwise visitors to the live domain will hit a login
   prompt.

3. **Verify indexing is correct on the live domain**
   - View source on `https://proavpodcast.fm/` → confirm
     `<meta name="robots" content="index, follow">`.
   - `https://proavpodcast.fm/robots.txt` → should allow crawling.
   - `https://proavpodcast.fm/sitemap.xml` → loads, lists both pages.

4. **Submit the sitemap to Google Search Console** if not already done.

4a. **Verify the .com redirect is live** — visit `http://proavpodcast.com`
    and confirm it 301s to `https://proavpodcast.fm`, path and all
    (e.g. `/contact` should carry over, not just the bare domain).
    This is set up via Route 53 + CloudFront (see
    `DOMAIN-REDIRECT-SETUP.md`), not Amplify, so it's independent of
    everything else in this checklist — worth a quick check anyway
    since it's easy to forget.

5. **Spot-check the live site** — merch links, contact form
   submission, video embed, desktop + mobile.

6. **Decide what to do with the `splash` branch** — leave it connected
   (unused, harmless) or disconnect it from the app. Either is fine.

7. **Announce** — Discord, socials, wherever.

## If you ever need to go BACK to the splash page

(e.g., pulling the site down temporarily) — just repoint the domain
back to `splash` in Domain management. No redeploy, no file changes.
