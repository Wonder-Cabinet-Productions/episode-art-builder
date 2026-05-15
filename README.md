# Wonder Cabinet — Episode Art Builder

A browser-based tool for producers to compose **Wonder Cabinet** podcast episode art.

Three panels on a green-bordered canvas: a guest portrait on the right, two thematic images on the left. Drag-and-drop or Unsplash search to populate; drag within a panel to reposition; export the 3000×3000 composite + a `credits.md` file as downloads.

Original design by [Art + Sons](https://artandsons.com/).

## What this is

This is the public, producer-facing build of the Wonder Cabinet episode-art compositor. It runs entirely in the browser — no server, no accounts, no upload destination. The two files you export (composite JPG + credits.md) are downloads to your local machine.

## What this isn't

A separate, fuller version of this tool lives inside the [`podcast-publishing-suite`](https://github.com/Wonder-Cabinet-Productions/podcast-publishing-suite) repo as part of the agentic episode-publishing pipeline. That version has additional integrations (direct save to canonical episode folders, auto-populated source images from the episode's source-image folder). This public build intentionally omits those integrations — they require a local Python server and access to the episode pipeline.

## Running locally

```bash
# Any static file server works
cd public/
python3 -m http.server 8765
# → http://localhost:8765
```

## Deploying

The `public/` directory is a self-contained static site. Drop it into Cloudflare Pages, GitHub Pages, Netlify, Vercel, or any static host. The only external dependency is the Unsplash API for the search tab — see deployment notes below.

### Unsplash API

The Unsplash search tab calls `api.unsplash.com` directly from the browser. For a production deploy, route those calls through a Cloudflare Worker (or similar serverless proxy) that adds your Unsplash access key from a server-side secret. This avoids shipping the key in client JS.

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
