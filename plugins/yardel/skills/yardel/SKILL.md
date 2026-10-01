---
name: yardel
description: Publish HTML pages, Markdown reports, slide decks, dashboards and static web apps as private pages with Yardel, and share them with named people, a company domain, or collaborators who can edit. Use when the user asks to publish, deploy, host, share or "send a link to" something you built, or to manage who can open or change a Yardel page.
---

# Yardel

Yardel turns what you build into a private web page that only the people the user names can open. Viewers confirm their email with a 6-digit code; no account or password. Everything goes through the `yard` CLI. Every command takes `--json` and answers with one JSON object on stdout; errors come back as `{ "ok": false, "error": { code, title, detail, hint } }`, and `hint` is the exact fix.

## Connector or CLI

Yardel is reachable two ways. Use whichever this session has:

- **The Yardel connector** (Claude.ai, ChatGPT, or any MCP client connected to `https://app.yardel.dev/mcp`): tools `publish_page`, `share_page`, `unshare_page`, `set_link_access`, `list_pages`, `who_has_access`, `list_versions`, `rollback_page` and `access_requests`. `publish_page` takes the whole page as `content` with `format: "html"` or `"markdown"`, so put everything in one self-contained document (inline CSS and JS). The rules below (confirm emails before sharing, roles, expiry, reporting back) apply the same way; each tool answers with a sentence plus JSON, and errors carry a fix.
- **The `yard` CLI**, when you can run shell commands (Claude Code, Codex, Cursor). It also publishes folders and built projects, which the connector can't. The rest of this file shows CLI commands; the connector tools take the same arguments.

If neither is available, tell the user they can add the connector (Settings → Connectors → add custom connector → `https://app.yardel.dev/mcp`) or install the CLI with `npm i -g @yardel/cli`.

## Before you start (CLI)

- If `yard` isn't installed, install it with `npm i -g @yardel/cli` (or run any command as `npx -p @yardel/cli yard <command>`).
- Check the CLI is signed in: `yard whoami --json`. If it fails with `E_AUTH_REQUIRED`, run `yard login --json`. The first line it prints has `verification_uri_complete` and `user_code`: give the user that link and code to approve in their browser, then wait for the command to finish.
- `whoami` also says whether the user is the **owner** of the workspace or a **collaborator** on some of its apps. Collaborators can only work on the apps they were added to.

## Publish

```sh
yard deploy report.html --app q3-report -m "First draft" --json
yard deploy notes.md --app churn-findings --json      # Markdown becomes a clean, readable page
yard deploy ./dist --app pricing-proto --json          # a folder with index.html
yard deploy . --json                                    # a Vite/Astro/SvelteKit/CRA/Next-export project (builds first)
```

- The first deploy creates the app and writes `yard.json` with its name; later deploys from the same folder need no `--app`.
- Every deploy is a new version. Only changed files upload.
- `yard doctor --json` explains what a deploy would do (detection, size, secrets, blocked hosts) without uploading. Run it when unsure.
- A deploy is refused if it contains a live secret (`E_SECRET_FOUND`). Never work around this: move the secret out of the client code. Publishable keys only produce a warning.
- External scripts, styles or APIs outside the default allow-list are reported as `csp` warnings with the `yard.json` line to add.
- `yard versions <app> --json` lists versions; `yard rollback <app> v3 --json` makes an older version the latest again (nothing is deleted).

## Share

Sharing sends email, so confirm the exact addresses with the user before running it, unless they already gave them to you in this conversation.

```sh
yard share q3-report maya@pancake.studio dan@pancake.studio --json                # can view, for 30 days
yard share q3-report alex@northwind.io --role commenter --json        # can view and comment
yard share q3-report @pancake.studio --yes --json                           # everyone with a confirmed pancake.studio email
yard share q3-report ben@pancake.studio --role editor --json                # a collaborator (see below)
yard share q3-report maya@pancake.studio --expires 7d -m "Read section 2 first" --json
```

- Roles: `viewer` (default), `commenter`, `editor`. Expiry: `7d`, `30d` (default for viewers), `90d`, a date like `2026-12-31`, or `never`.
- Sharing with a whole domain needs `--yes` (it's a bulk change); public email domains like gmail.com are refused.
- `yard share` prints each person's personal link. They also get an invite email from the user.
- Remove access: `yard unshare q3-report maya@pancake.studio --json` (takes effect within a minute). `yard unshare q3-report --all --yes --json` removes every viewer and commenter at once, but not collaborators.
- `yard audience list <app> --json` shows who has access and whether their invite was sent. `yard audience pin <app> <group> v2 --json` keeps a group on one version while new versions ship.
- `yard requests --json` lists people who asked for access; approve with `yard requests --approve <id> --json` or decline with `--decline <id>`. Ask the user before approving.

## Anyone with the link

Instead of naming people, a page can open for anyone who has its secret link. The plain page address stays private either way.

```sh
yard link q3-report signed-in --json      # they confirm their email first; the owner sees who opened it
yard link q3-report public --yes --json   # no sign-in at all
yard link q3-report off --json            # back to named people only
yard link q3-report --json                # current mode and the link
```

- The JSON has the `url` to send. Give the user that URL, not the plain page address.
- Use `public` only when the user clearly wants it and the content is fine for anyone to see: ask first. Public pages show a "Public page · Report" link in the Yardel badge, and search engines are asked not to index them.
- Only the workspace owner can change link access. Public needs a workspace older than a day (`E_FORBIDDEN` before that); offer `signed-in` meanwhile.
- Turning it off works at once. `yard unshare <app> --all --yes` also turns it off.

## Collaborators

A collaborator is someone who can change the page, not just open it: they can publish new versions, roll back, share it with viewers and commenters, and handle its access requests, from their own coding agent, terminal or the Yardel portal.

**Adding one** (workspace owners only):

```sh
yard share q3-report ben@pancake.studio --role editor --json
```

- Ben gets a "you've been added as a collaborator" email. His access has no end date; it lasts until the owner removes him.
- Collaborators are added one person at a time, never by domain.
- Remove a collaborator with `yard unshare q3-report ben@pancake.studio --json`. They lose access to that app, and to the workspace if it was their last one there.

**If you are working as a collaborator** (the user was added to someone else's app):

```sh
yard login --workspace <owner's workspace> --json     # the workspace name is in the invite email
yard apps --json                                       # the apps you can work on
yard deploy ./dist --app <app> -m "What changed" --json
```

- Collaborators can only deploy to apps they were added to, and can't create new apps in that workspace (`E_FORBIDDEN`). If the user also owns a workspace, use `--workspace` to pick which one a login is for.
- Collaborators can share with viewers and commenters, but only the owner can add or remove collaborators.

## In the browser

The owner portal is at https://app.yardel.dev: apps, versions with rollback, people with access (including collaborators), the share dialog, and access requests. Point the user there for anything they'd rather click than type.

## Reporting back to the user

After a deploy or share, tell the user in a sentence or two: the page's URL, who can open it now (and whether invites went out), and the version number. Quote any `warnings` from the JSON. On an error, relay the `title` and follow the `hint`.
