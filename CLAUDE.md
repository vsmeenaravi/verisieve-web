# CLAUDE.md — verisieve-web

## What this is

The static site for **verisieve.com** — landing page, privacy policy, and support
page for the VeriSieve iOS app. Plain HTML + CSS, no build step, no framework, no
dependencies. Hosted on **GitHub Pages** from `main` with a custom domain
(`CNAME` → `verisieve.com`).

Pushing to `main` publishes. There is no staging environment.

```
index.html              landing page
privacy.html            privacy policy
support.html            support / contact
assets/style.css        all styling
assets/*.jpg|png        app icon and App Store screenshots
fact_check_config.json  LIVE production config — see below
CNAME                   custom domain
```

Preview locally with `python3 -m http.server 8000` and open
`http://localhost:8000`.

## fact_check_config.json is production configuration, not content

The **shipped iOS app fetches this file at runtime** from
`https://verisieve.com/fact_check_config.json` to tune Fact Check's web-search
budget without an App Store release. Editing and pushing it changes behaviour for
real users on devices already in the field.

```json
{ "max_uses": 2, "daily_search_cap": 20 }
```

- `max_uses` — web-search tool uses allowed per individual fact check
- `daily_search_cap` — total searches allowed per user per day

Rules:

1. Treat any change as a production change. Confirm the intended values with the
   user before pushing; do not adjust them speculatively.
2. The app caches the file for **12 hours**, so a change is not instant and a
   rollback is not instant either.
3. The app falls back to hardcoded defaults (`max_uses: 2`, `daily_search_cap: 20`)
   on any fetch or parse failure, so malformed JSON degrades quietly rather than
   breaking Fact Check. That also means a broken file may go unnoticed — validate
   the JSON before pushing.
4. Temporary test values must be reverted. This has already happened once:
   `28b543a` lowered the cap to 1 for testing, `dce9e72` put it back.
5. Keep the committed values in sync with `FactCheckSearchConfig.defaults` in the
   app repo unless there is a deliberate reason to diverge.

## Copy is owned by the app repo

The source of truth for the published text is in `~/Developer/Veritas`:

- `docs/PRIVACY_POLICY.md` → `privacy.html`
- `docs/APP_STORE_LISTING.md` → landing page feature copy

Update the app repo's markdown first, then mirror it here in a separate commit —
the two repos have separate remotes and never share a commit. Keep the
user-facing claims consistent with what the app actually does; the App Store
review history makes copy/behaviour mismatches expensive.

Public contact address is **support@verisieve.com**, never a personal email
(`da2eeb9`).

Screenshots in `assets/` correspond to a specific App Store screenshot set. When
the app's UI changes materially, refresh them rather than leaving stale imagery.

## Git conventions

Work on `main`. Short descriptive subjects; the app repo's
`type: summary` convention (`feat:`, `fix:`, `docs:`, `test:`) is preferred for
new commits. Trailers:

```
Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
```

Since `main` is live, confirm with the user before pushing anything that changes
published content or `fact_check_config.json`.
