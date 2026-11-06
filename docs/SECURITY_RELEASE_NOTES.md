# MetaTruth alpha hardening

## Changes in this branch

- The portal refuses to start without an explicit administrator email and a password of at least 16 characters.
- CORS no longer uses a wildcard. Set `METATRUTH_ALLOWED_ORIGINS` to a comma-separated list of trusted origins, including the actual `chrome-extension://<id>` origin if the extension calls the portal.
- JSON request bodies have a 64 KiB limit. Invalid JSON returns a controlled 400 error.
- Static files have `X-Content-Type-Options: nosniff` and `Referrer-Policy: no-referrer` headers.
- Runtime database files are ignored by Git going forward.

## Known release blockers

1. **Previously tracked database WAL/SHM files remain in Git history.** They must be removed from the index and any sensitive contents investigated. `.gitignore` does not untrack existing files or rewrite history.
2. The extension currently has broad `<all_urls>` permissions and a globally registered content script. Replace with on-demand `chrome.scripting.executeScript` capture and narrow host access after a full capture-flow test.
3. Existing password hashes use a fixed scrypt salt. Migrate to unique per-user salts and verify existing accounts before switching algorithms.
4. Add authentication rate limits, authorization tests, logout/session revocation, and a documented bootstrap/recovery flow.
5. Review and restrict all endpoints, audit event integrity, and user-supplied values before exposing this service beyond loopback.
6. No verified end-to-end tests or deployment checks have been completed by this change alone.

## Local launch

Node 22+ is recommended because the portal imports `node:sqlite` and uses `DatabaseSync`.

```bash
export METATRUTH_ADMIN_EMAIL='admin@example.invalid'
export METATRUTH_ADMIN_PASSWORD='replace-with-a-unique-long-secret'
export METATRUTH_ALLOWED_ORIGINS='chrome-extension://YOUR_EXTENSION_ID'
node portal/server.mjs
```

Do not publish an actual admin password or database to the repository. Keep the server bound to `127.0.0.1` during alpha testing.
