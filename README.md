# volvo-oauth-bridge

Static HTTPS bridge for the Volvo Companion Android app's OAuth callback. Volvo's developer portal does not accept custom-scheme redirect URIs (`volvo-companion://...`), so we register the HTTPS URL of this page in the portal. The page then forwards `?code=&state=` to the deep link `volvo-companion://oauth/callback?…` which the installed app catches via its intent-filter.

## Deploy to GitHub Pages

1. Create an **empty public** repo on GitHub, e.g. `volvo-oauth-bridge`.
2. Push this folder:
   ```bash
   cd ~/Apps/volvo-oauth-bridge
   git checkout -b main 2>/dev/null || git branch -m main
   git add CNAME README.md redirect/
   git commit -m "Initial bridge"
   git remote add origin git@github.com:<your-user>/volvo-oauth-bridge.git
   git push -u origin main
   ```
3. On GitHub: **Settings → Pages**
   - Source: **Deploy from a branch**
   - Branch: `main` / `(root)`
   - Custom domain: `volvo.roussel-zeter.eu` (the `CNAME` file already does this; the field auto-fills)
   - Tick **Enforce HTTPS** once the cert is provisioned (~5–15 min).
4. DNS at your DNS provider for `roussel-zeter.eu`:
   - `volvo.roussel-zeter.eu  CNAME  <your-github-user>.github.io.`
5. Verify:
   ```bash
   curl -sSL https://volvo.roussel-zeter.eu/redirect/ | head
   # should show the bridge HTML
   ```

The bridge is served at `https://volvo.roussel-zeter.eu/redirect/` (with the trailing slash — the file lives at `redirect/index.html`). Register exactly that URL in the Volvo portal AND paste the same string into the **Redirect URI** field of the Volvo Companion app's sign-in screen. They must be byte-identical.

## How it works

```
Volvo IdP  ──code,state──▶  https://volvo.roussel-zeter.eu/redirect/  ──location.replace──▶  volvo-companion://oauth/callback?code=…&state=…
                              (this page)                                                          (caught by AndroidManifest intent-filter)
```

No JavaScript framework, no analytics, no third-party requests. The page is ~1.5 KB.
