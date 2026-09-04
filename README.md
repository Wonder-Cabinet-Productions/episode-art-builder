# Wonder Cabinet — Episode Art Builder

A browser-based tool for producers to compose **Wonder Cabinet** podcast episode art.

Three panels on a green-bordered canvas: a guest portrait on the right, two thematic images on the left. Drag-and-drop or Unsplash search to populate; drag within a panel to reposition; export the 3000×3000 composite + a `credits.md` file as downloads.

Original design by [Art + Sons](https://artandsons.com/).

## What this is

This is the public, producer-facing build of the Wonder Cabinet episode-art compositor. All image composition happens in the browser — there's no backend holding your work and no upload destination. The two files you export (composite JPG + credits.md) are downloads to your local machine.

The one server-side piece is a small Cloudflare Pages Function that proxies Unsplash search so the API key stays off the client. The hosted deployment also sits behind Cloudflare Access, so it does require a sign-in — see [Access control](#access-control).

Note that nothing persists: the source pool, the three panels, and Saved Combinations all live in memory, so reloading the page discards them. Export before you refresh.

## What this isn't

A separate, fuller version of this tool lives inside the [`podcast-publishing-suite`](https://github.com/Wonder-Cabinet-Productions/podcast-publishing-suite) repo as part of the agentic episode-publishing pipeline. That version has additional integrations (direct save to canonical episode folders, auto-populated source images from the episode's source-image folder). This public build intentionally omits those integrations — they require a local Python server and access to the episode pipeline.

## Running locally

Which local server you want depends on whether you need the Unsplash search tab.

**Upload, compose, and export only** — any static server will do:

```bash
cd public/
python3 -m http.server 8765
# → http://localhost:8765
```

Unsplash search will fail here. It calls `/api/unsplash`, which a plain static server doesn't
serve, so the tab reports a 404. Everything else works.

**Including Unsplash search** — you need the Pages Function, so run Wrangler from the repo root:

```bash
echo 'UNSPLASH_KEY=your-unsplash-access-key' > .dev.vars   # gitignored; create once
npx wrangler pages dev public
```

`.dev.vars` is how Wrangler supplies `UNSPLASH_KEY` locally. Without it the Function returns
`500 {"error":"UNSPLASH_KEY not configured"}`.

## Deploying

Live at **https://art.wondercabinetproductions.com** — Cloudflare Pages project
`episode-art-builder`.

**Deploys are manual. Pushing to `master` does not deploy anything.** The Pages project is
configured for *direct upload*, with no Git integration, so the repo and production drift apart
silently. After committing, publish explicitly:

```bash
npx wrangler pages deploy public --project-name=episode-art-builder
```

Wrangler uploads `public/` as the static site and `functions/` as Pages Functions. Files at the
repo root (this README, `.impeccable.md`) are not part of the deployed output.

### Access control

The site sits behind **Cloudflare Access**, covering both `art.wondercabinetproductions.com` and
`episode-art-builder.pages.dev` — including the `/api/*` routes, which keeps the Unsplash proxy
from being an open relay for the API key.

The policy is scoped to **`@wondercabinetproductions.com`** identities. Signing in with any other
address (a personal Gmail, a client's own domain) is refused at the login screen.

> **This failure looks like an outage but isn't.** A refused identity produces a login you can
> never get past, with the site itself perfectly healthy behind it. Before debugging the app,
> confirm which email you're presenting to Access. Onboarding an outside collaborator means adding
> them to the Access policy in the Zero Trust dashboard — not changing anything in this repo.

### Unsplash API

Browser code never touches `api.unsplash.com` directly. Requests go to `/api/unsplash`, handled by
`functions/api/unsplash.js`, which attaches the access key server-side so it's never shipped in
client JS. The key lives in the Pages project as the `UNSPLASH_KEY` secret:

```bash
npx wrangler pages secret put UNSPLASH_KEY --project-name=episode-art-builder
```

Attribution is contractual, not optional: the `utm_source`/`utm_medium` parameters on photographer
links and the `action=track-download` ping are both Unsplash API Guidelines requirements. Removing
them puts API access at risk.

## Layout spec

| Element | Size |
|---|---|
| Canvas | 3000×3000 |
| Outer green border | 25px, `#10a544` |
| Black padding around each image | 30px, `#000000` |
| Green gutter between panels | 26px |
| Image area (each panel) | 1402px wide |
| Right panel height | 2890px (full canvas height minus borders) |
| Left panel height | 1402px (square) |

## License

This tool is published as part of the Wonder Cabinet production toolkit. The original collage design is by Art + Sons.
