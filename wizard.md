---
wizard: 0.2
id: wizard-claude
name: Claude
type: external
category: system
tags: [security]
version: 0.2.21
summary: Connect one or more Claude subscriptions; every agent on the box uses them, and you see how much each has left.
icon: icon.png
license: MIT
source: https://github.com/oscarhenrycollins/claude-connection
runtime:
  kind: runtipi
  config: runtipi/config.json
  compose: runtipi/docker-compose.yml
mcp:
  path: /mcp
  port: 8799
  transport: streamable-http
skill: SKILL.md
view:
  path: /
  port: 8799
provides:
- ai-connection
- claude-subscription
health: /health
browser_contract:
  version: 1
  isolation: opaque
  operations: [ai_accounts.inventory, ai_accounts.change, ai_accounts.progress, ai_accounts.step, claude_code.status,
    claude_code.connect, claude_code.code, claude_code.cancel, claude_code.logout, claude_code.test]
  navigation:
  - {id: signin, url: 'https://claude.ai/oauth/authorize', query: [client_id, code, code_challenge, code_challenge_method, redirect_uri, response_type, scope, state]}
  - {id: signin-claude-com, url: 'https://claude.com/cai/oauth/authorize', query: [client_id, code, code_challenge, code_challenge_method, redirect_uri, response_type, scope, state]}
---

# Claude

Connect your Claude Pro or Max subscriptions. The original sign-in runs Anthropic's official Claude Code login and stays in a private shared volume; **Add account** signs in more subscriptions (any number) inside your agent with Hermes' own Claude sign-in, so your agent rotates across them when one hits its limit. Agents never see a token.

This is a Wizard AI connection app (`wizard.ai-connection/2`): `GET /api/connection` reports the overall status, plan and usage plus one row per subscription, `GET /api/accounts` lists the subscriptions, and the MCP tool `connection_status` gives agents the same answer. Adding, reconnecting and removing subscriptions goes through the agent's dashboard (Wizard OS plugin, stock `hermes auth`); the original Claude Code sign-in keeps its own page at `/claude-code/`. Engine source: `humanitylabs-org/wizard-app-store` `images/wizard-claude`.

**Browser boundary (draft, default-off).** `browser_contract` asks for the reviewed account operations and the provider's own sign-in page; nothing is granted until the owner grants it, and nothing changes unless `WIZARD_BROWSER_BOUNDARY` is on. With it on, the accounts routes answer only the Agent Service adapter (the MCP front's listener gate), account changes run only after the owner confirms them in a Wizard dialog, and only Wizard opens the sign-in page (wizard-os `spec/browser-trust-shell-intents`).
