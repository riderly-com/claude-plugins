# Riderly plugins for Claude

This folder is a Claude plugin marketplace. It is the editable source in the
Riderly repository and is mirrored as-is to a public GitHub repository so users
can install it. It contains no backend code, data, environment files, or credentials.

```
.claude-plugin/marketplace.json            marketplace "riderly"
plugins/riderly-rental-discovery/
  .claude-plugin/plugin.json               plugin manifest and listing metadata
  .mcp.json                                remote MCP server connection
  README.md                                public listing "about" text
  assets/icon.png                          listing icon
  skills/riderly-rental-discovery/SKILL.md usage rules for Claude
```

## Listing copy

- Title and tagline live in `plugin.json` as `displayName` and `description`. The
  marketplace entry does not set them, so `plugin.json` is the single source.
- `plugins/riderly-rental-discovery/README.md` is the public "about" text shown in
  Anthropic's directory. Keep it at least 40 words (code blocks don't count).
- Directory policy requires listing text to match what the tools actually do. Don't
  claim in-chat booking or payment, best prices, instant confirmation, or inventory sizes.
  Keep the "Good to know" section accurate when the tools or telemetry change.
- `icon`, `supportUrl`, `privacyPolicyUrl`, and `termsOfServiceUrl` in `plugin.json` are
  read only by the directory. Claude Code ignores them.

## What the plugin does

It connects Claude to the existing anonymous, read-only MCP server at
`https://mcp.riderly.com/mcp?client=claude`, which exposes:

- `search_motorbike_rentals`
- `get_motorbike_rental_details`

The `client=claude` query parameter makes the server record telemetry with
`provider=claude` and return Riderly links tagged `utm_source=claude` and
`utm_medium=claude_plugin`. Listing opens and attributed bookings keep that
provider. Requests without the parameter are recorded as ChatGPT.

Every tool result includes the full results as JSON text, and Claude writes the
answer itself as a list or table, as the skill describes. The results widget is
currently turned off for all clients.

## Keep in sync with the OpenAI plugin

`skills/riderly-rental-discovery/SKILL.md` is a copy of the OpenAI skill in
`ai_integrations/plugin/riderly-rental-discovery/` with Claude-specific changes
(results always presented as text, no OpenAI API note). It is a plain copy,
not a symlink, because symlinks break in the mirrored public repository. When
tool inputs, limits, or rules change, update both skills.

## Install

From the public repository:

```text
/plugin marketplace add riderly-com/claude-plugins
/plugin install riderly-rental-discovery@riderly
```

From a local checkout of this repository (for testing):

```text
/plugin marketplace add ./ai_integrations/plugin/claude
/plugin install riderly-rental-discovery@riderly
```

Then run `/mcp` and confirm both tools are listed. Claude names them
`mcp__plugin_riderly-rental-discovery_riderly-rental-discovery__<tool>`.

Users who only use claude.ai chat, without plugins, can add the same URL
(`https://mcp.riderly.com/mcp?client=claude`) as a custom connector with no
authentication.

## Release

1. Bump `version` in `plugins/riderly-rental-discovery/.claude-plugin/plugin.json`
   on every change. Users only receive updates when the version changes.
2. Validate the manifests: `claude plugin validate ./ai_integrations/plugin/claude`
3. Deploy web and MCP changes first when the release depends on server changes.
4. Mirror this folder to the public repository from the Riderly repository root:

```bash
git subtree split --prefix ai_integrations/plugin/claude -b claude-plugins-release && git push <public-remote> claude-plugins-release:main && git branch -D claude-plugins-release
```

Submission to Anthropic's official plugin directory is a separate step. Confirm
the current submission process and requirements at that time.

## License and repository

- Public repository: https://github.com/riderly-com/claude-plugins (set as
  `repository` in `plugin.json`). Create it before the first mirror push.
- The plugin is licensed under Apache-2.0 (`license` in `plugin.json`, full text in
  `plugins/riderly-rental-discovery/LICENSE`). `NOTICE` states that the license covers
  only the plugin files and that the Riderly name and logo are not licensed.

## Open items

- The results widget is turned off because it timed out in the Claude app
  ("This app timed out"). Before turning it back on, check the widget's
  `_meta.ui.domain` value (`https://mcp.riderly.com`), which the MCP Apps spec
  leaves to each host, and the `ui/initialize` handshake in Claude.
- Claude has no per-user request metadata like ChatGPT's `openai/subject`, so
  rate limits fall back to per-IP. Claude Code connects from the user's machine.
  If claude.ai or desktop connections share egress IPs, users may be throttled
  together; watch for this after launch.
