# .com → .fm Redirect (Route 53 + CloudFront)

Both `proavpodcast.com` and `proavpodcast.fm` are hosted in Route 53, so
this is entirely AWS-native — no third-party registrar involved.

**Why not just an Amplify redirect rule:** Amplify's redirect rules
operate within a single app/domain. Redirecting one fully custom domain
to a *completely different* domain (`.com` → `.fm`) is a known-broken
pattern there. CloudFront + a CloudFront Function is the reliable way
to do a real cross-domain 301.

**Why a CloudFront Function instead of Lambda@Edge:** it's the lighter,
cheaper option for a plain redirect like this — no need for
Lambda@Edge's extra overhead when there's no real logic beyond
"redirect and preserve the path."

---

## 1. Request an ACM certificate (us-east-1)

CloudFront requires the certificate to live in **us-east-1**,
regardless of which region anything else is in.

1. ACM console → **us-east-1** → Request a public certificate.
2. Domain names: `proavpodcast.com` and `www.proavpodcast.com` (add
   the www version too if you want that covered — cheap to include
   now).
3. Validation method: **DNS validation**.
4. Since the hosted zone is already in Route 53, ACM will offer a
   **"Create records in Route 53"** button that adds the validation
   CNAME automatically — click it instead of copying values by hand.
5. Wait for status to flip to **Issued** (usually a few minutes).

## 2. Create the CloudFront Function

1. CloudFront console → **Functions** → Create function.
2. Name: `redirect-com-to-fm`.
3. Paste this as the function code:

   ```js
   function handler(event) {
     var request = event.request;
     return {
       statusCode: 301,
       statusDescription: 'Moved Permanently',
       headers: {
         'location': { value: 'https://proavpodcast.fm' + request.uri }
       }
     };
   }
   ```

   This preserves the path and query string —
   `proavpodcast.com/contact` → `proavpodcast.fm/contact`, not just a
   blanket redirect to the bare domain.

4. **Publish** the function.

## 3. Create the CloudFront distribution

1. CloudFront console → Create distribution.
2. **Origin domain**: doesn't functionally matter — the function
   returns a redirect before CloudFront ever contacts the origin. The
   simplest choice is to point it at the Amplify default domain for
   `main`/`splash` (the `*.amplifyapp.com` URL), just so it's a valid
   reachable origin.
3. **Viewer protocol policy**: Redirect HTTP to HTTPS.
4. **Alternate domain name (CNAME)**: `proavpodcast.com` (and
   `www.proavpodcast.com` if you requested that in the cert).
5. **Custom SSL certificate**: select the cert from step 1.
6. Under **Function associations** → **Viewer request**: attach
   `redirect-com-to-fm` (CloudFront Functions, not Lambda@Edge).
7. Create the distribution. It takes several minutes to deploy fully
   (status will show "Deploying" → "Enabled").

## 4. Point Route 53 at the new distribution

1. Route 53 → Hosted zones → `proavpodcast.com`.
2. Edit (or create) the **A record** for the root domain:
   - Alias: **Yes**
   - Route traffic to: **Alias to CloudFront distribution**
   - Select the distribution from step 3.
3. Repeat for `www.proavpodcast.com` if you're covering that too.

Since this is an alias record (not a manual CNAME), there's no need to
know the distribution's actual `d1234.cloudfront.net` hostname — Route
53 resolves it directly.

## 5. Verify

```
curl -I http://proavpodcast.com
curl -I https://proavpodcast.com
curl -I https://proavpodcast.com/contact
```

Each should return a `301` with a `location:` header pointing at the
matching `https://proavpodcast.fm/...` path. Then check in an actual
browser too — confirm the address bar lands on `.fm`.

DNS itself should resolve almost immediately (Route 53 is fast), but
allow the CloudFront distribution a few extra minutes after creation
before testing if you get unexpected results early on.
