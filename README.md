# AXIOM — gated preview host

A standalone [GitHub Pages](https://pages.github.com/) host for the pre-release
**AXIOM claim-gate** preview, kept behind a passphrase gate.

It exists because the same preview on the main site (`axiom.inc/preview/`) sits
behind a Cloudflare Worker that overlays the path; serving it here, straight off
`*.github.io`, means the gate is what visitors actually get — over HTTPS, with
no proxy in front (WebCrypto needs a secure context).

## The gate

`preview/index.html` is an AES-256-GCM passphrase gate. The key is derived with
PBKDF2-SHA256 and the page decrypts in the browser via WebCrypto, so the
committed file contains **only ciphertext**. The plaintext source lives in
`preview-src/`, which is **gitignored** — committing it here (a public repo)
would defeat the gate. The passphrase is held out-of-band, never in this repo.

- Rebuild the gate / rotate the passphrase:
  `pip install cryptography && python3 scripts/gate-build.py --password 'new-passphrase'`
- Recover `preview-src/` on a fresh clone (needs the passphrase):
  `python3 scripts/gate-build.py --decrypt --password '...'`
- The gate needs `https://` or `localhost` (WebCrypto secure-context rule); it is
  `noindex` and disallowed in `robots.txt`.

Anyone with the passphrase can reshare what they see — rotation is re-running the
build with a new passphrase. Old ciphertext remains in git history, so treat a
leaked passphrase as burned.

## Deploy

Served directly from the `main` branch by GitHub Pages — no build step, no
workflow. One-time setup (owner only, in the GitHub UI):

**Settings → Pages → Build and deployment → Source: Deploy from a branch →
Branch: `main` / `/ (root)` → Save.**

GitHub then serves every file at the repo root, so the gate publishes at
`/preview/` on this repo's Pages URL. `.nojekyll` keeps the files un-processed.

(An Actions-based deploy workflow was removed: enabling Pages via the workflow
token isn't permitted on this repo — "Resource not accessible by integration" —
so the simpler branch source is used instead.)

## Note

The canonical claim-gate source and the wider marketing site live in
`dookdog/AXIOM-com`; this repo only re-hosts the gated preview. The demo's
"ask the search engine" handoff points at the main site, so that one link is
inert when the preview is served standalone here.
