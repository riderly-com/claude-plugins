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

Hosts that support MCP Apps (for example Claude desktop and claude.ai) can show
the `ui://riderly/rental-results-v4.html` results widget. Claude Code shows text
results, formatted by the skill.

## Keep in sync with the OpenAI plugin

`skills/riderly-rental-discovery/SKILL.md` is a copy of the OpenAI skill in
`ai_integrations/plugin/riderly-rental-discovery/` with Claude-specific changes
(text fallback when no widget is shown, no OpenAI API note). It is a plain copy,
not a symlink, because symlinks break in the mirrored public repository. When
tool inputs, limits, or rules change, update both skills.

## Install

From the public repository:

```text
/plugin marketplace add <org>/<public-repo>
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

## Open items

- Public repository name and organization. Add its URL as `repository` in
  `plugin.json` once it exists.
- Whether to add a `license` field to `plugin.json`.
- Confirm Claude accepts the widget's `_meta.ui.domain` value
  (`https://mcp.riderly.com`). The MCP Apps spec leaves the domain format to
  each host. If Claude rejects it or the widget does not render, emit
  `ui.domain` for ChatGPT only (`openai/widgetDomain` already covers ChatGPT).
- Claude has no per-user request metadata like ChatGPT's `openai/subject`, so
  rate limits fall back to per-IP. Claude Code connects from the user's machine.
  If claude.ai or desktop connections share egress IPs, users may be throttled
  together; watch for this after launch.
