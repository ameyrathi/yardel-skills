# Yardel skill

[Yardel](https://yardel.dev) turns what your agent makes (HTML pages, Markdown reports, slide decks, dashboards, static apps) into private pages that only the people you name can open. They confirm their email with a 6-digit code; you can take access back anytime.

This repo holds the Yardel agent skill. It teaches your agent to publish, share, add collaborators, roll back and handle access requests, through either the Yardel connector or the `yard` CLI.

## Claude Code

```
/plugin marketplace add ameyrathi/yardel-skills
/plugin install yardel@yardel
```

Then install the CLI it uses: `npm i -g @yardel/cli` (your agent can do this for you). Or skip the plugin and run `npm i -g @yardel/cli && yard skill install`.

To use the connector instead of the CLI:

```
claude mcp add --transport http yardel https://app.yardel.dev/mcp
```

## Claude.ai and Claude Desktop

1. **Connector:** Settings → Connectors → Add custom connector. Name `Yardel`, URL `https://app.yardel.dev/mcp`. Sign in with your email code and allow access.
2. **Skill (optional, recommended):** download `yardel-skill.zip` from the [latest release](https://github.com/ameyrathi/yardel-skills/releases/latest) and upload it in Settings → Capabilities → Skills.

## ChatGPT

Settings → Apps & Connectors → Advanced settings → turn on Developer mode. Then Create connector: name `Yardel`, URL `https://app.yardel.dev/mcp`, authentication OAuth. Sign in with your email code and allow access.

## Codex, Cursor and other agents

```
npm i -g @yardel/cli
yard skill install --agent codex     # or cursor, or all
```

Or with the [skills CLI](https://skills.sh): `npx skills add ameyrathi/yardel-skills`.

## Try it

> Publish this as a private page called q3-report and share it with maya@acme.com

The skill source lives in the Yardel monorepo; this repo is a published copy.
