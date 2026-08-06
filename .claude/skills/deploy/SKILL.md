---
name: deploy
description: Publish the Episode Art Builder to Cloudflare Pages and verify the deploy landed. Use when asked to deploy, ship, publish, or push this site live.
disable-model-invocation: true
---

# Deploy the Episode Art Builder

This project's Pages deploy is **manual**. The Cloudflare Pages project `episode-art-builder` is
direct-upload with no Git integration, so pushing to `master` publishes nothing. Production only
changes when someone runs the command below — which is exactly why it gets forgotten. Production
once sat two months behind `master` with nothing signalling it.

> If issue #7 lands and Git integration is wired up, this skill becomes obsolete — deploys will
> happen automatically on push. Check whether the project still reports `Git Provider: No` before
> assuming a manual deploy is needed.

## Before deploying

Deploying publishes to a live, client-facing site. Do not run this because a change was merged —
run it because the user asked for a deploy.

1. **Report the current state and confirm.** Show the user what they are about to publish:

   ```bash
   git branch --show-current
   git log -1 --format='%h %s'
   git status --porcelain
   ```

   Wrangler uploads the **working tree**, not a commit. A dirty tree means uncommitted changes go
   live. If `git status` is not clean, say so explicitly and get confirmation before continuing.

2. **Confirm you are on the intended branch.** Deploying from a feature branch publishes that
   branch's content to production. Usually you want `master`, current with `origin`:

   ```bash
   git fetch origin
   git rev-list --left-right --count origin/master...HEAD
   ```

   A non-zero left number means `master` has moved and you are about to publish a stale tree.

## Deploy

```bash
npx wrangler pages deploy public --project-name=episode-art-builder
```

Only `public/` (static site) and `functions/` (Pages Functions) are published. Root files —
`README.md`, `CLAUDE.md`, `.impeccable.md` — are not part of the deployed output, so a commit
touching only those changes nothing user-visible. Say so rather than implying a functional change.

## Verify

Never report success from the deploy command's output alone. Confirm both that the deployment
registered as Production and that the site still answers:

```bash
wrangler pages deployment list --project-name=episode-art-builder | head -8
curl -sS -o /dev/null -m 20 -w 'http=%{http_code}\n' https://art.wondercabinetproductions.com
```

- The top row should show `Production` with the expected commit in the `Source` column.
- **`http=302` is correct, not a failure.** The site is behind Cloudflare Access; an unauthenticated
  request is supposed to be redirected to the login. A `200` here would mean the gate is off.

To verify the app itself rather than the gate, the user has to load it in a browser signed in with
an `@wondercabinetproductions.com` address — that cannot be checked from the CLI.

## If the Unsplash search breaks after a deploy

Check the secret survived; it is project-level and not part of the upload:

```bash
wrangler pages secret list --project-name=episode-art-builder
```

A missing `UNSPLASH_KEY` makes the Function return
`500 {"error":"UNSPLASH_KEY not configured"}`. Restore with:

```bash
wrangler pages secret put UNSPLASH_KEY --project-name=episode-art-builder
```

## If a user reports the site is down

Check which email they are signing in with **before** investigating the deploy or the app. The
Access policy is scoped to `@wondercabinetproductions.com`; any other address is refused at the
login screen, which looks identical to an outage. See `CLAUDE.md` under "Access control".
