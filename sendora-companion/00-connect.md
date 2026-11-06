# 00. Connect Claude to Sendora

This companion maps each skill in the free pack to the exact steps inside Sendora. Everything here assumes Claude is connected to your Sendora workspace over MCP. Status labels used below: Available, Available with setup, Rolling out, Check first.

## What MCP is, in one line

MCP is the open protocol that lets an AI assistant use Sendora's tools. Once connected, Claude can read your workspace and, when you approve, build drafts in it. About 100 tools are exposed in total.

## Connect (Available)

1. In Claude, add Sendora as a connector (custom connector or MCP server) using the connection address from your Sendora workspace settings. Check the exact address in the app, do not guess it.
2. Sign in with OAuth. On the consent screen, choose which workspaces Claude may touch. Pick only the one you want.
3. Grant read access first. Writes need an explicit write permission. Grant it only when you are ready to build something.
4. Start a new chat. Claude should list Sendora tools (names like find_prospects, read_leads, edit_leads, enrich_leads). If it does not, the connection did not finish: reconnect and check the workspace choice.

Your access matches your role. Prospect search needs a manager, admin or owner role. View-only and client roles are refused.

## What to say first

Paste this as your first message in a new chat:

```
You are connected to my Sendora workspace over MCP. Before anything else:
1. Read only. Look at my workspace settings and my lead lists. Tell me what already exists.
2. Do not change, create, send, buy or spend anything until I say yes to that exact action.
3. Before every write, tell me the target and the effect, including any credit cost.
4. Anything you build stays a draft until I approve it.
Then wait for my next instruction.
```

## How the skills find the workspace

- The pack skills are neutral. They read a file called business-profile.md if you have one, and otherwise ask you a few questions.
- Inside Sendora, Claude reads the workspace itself: settings and company profile (read_platform), lead lists (read_leads), and anything already built. It should build on what exists.
- If you manage more than one workspace (for example an agency), say which one. Claude should not guess.
- Tools take an action field, for example read_leads with action lists. Claude picks these; you do not need the names. Ids come from tool results and are passed back exactly as returned.

## The safety habit

1. **Read first.** Look at what exists before changing anything.
2. **Preview before commit.** Searches and imports show a preview. A preview writes no leads and, for prospect search, spends nothing.
3. **Confirm before any write.** Anything that creates leads, spends credits, sends, calls, publishes or deletes needs your yes, in chat, right before it runs.
4. **Drafts until approved.** Editing a campaign does not start it. Only a launch action starts one. Nothing in this companion launches or sends.
5. **Report what was seen.** Claude reads the record back and tells you what is there, not what it meant to do.
6. **Blockers are reported, not retried.** An empty wallet, a missing permission or a compliance rule stops the step. Claude tells you the fix.
7. **No secrets in chat.** Never paste API keys, tokens or passwords.

## What an AI agent cannot do

- It cannot complete the purchase of a domain, mailbox or phone number. A person checks out in the app.
- LinkedIn and X are data sources only. Sendora does not send LinkedIn or X messages.
- Your own compliance duties stay yours: lawful basis, consent, targeting and message content. Sendora gives controls such as do-not-contact and quiet hours.

## Other AI agents (for example Grok)

Sendora's MCP is built for MCP-capable agents, and Claude and ChatGPT are the ones documented. Whether any other agent, Grok included, can connect depends on that agent supporting MCP with OAuth. This is unconfirmed. Do not assume it works. Check first, and use Claude if you want a known path.

## Money, in brief (Check first)

Credits: 100 credits = $1, and they never expire. Searching and previewing prospects is free. A work email found costs 2 credits, a US mobile number found costs 8 credits, optional email verification costs 1 credit per conclusive answer. Plan allowances and live prices are Check first: confirm them in the app before relying on them. Results depend on your list, offer, setup and market.
