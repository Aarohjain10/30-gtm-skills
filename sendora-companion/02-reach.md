# Sendora companion: 3-reach

How to do each reach skill inside Sendora, with Claude connected over MCP. Seven skills: deliverability-plan, spam-check, cold-email, follow-up-sequence, channel-order, text-whatsapp-compliance, inbound-voice-prompt.

## Read this first

- Nothing here is a guarantee. Sendora is built to protect sender reputation. It cannot promise inbox placement, replies or meetings.
- Claude drafts and prepares. You approve every spend, launch and send. Sendora tools that change things only run after you say yes in chat.
- Status labels: **Available**, **Available with setup**, **Rolling out**, **Check first** (a fact that changes or was not verified live).
- Prices below are customer prices from the pricing source. Payments were in test mode when written, so every price is **Check first**. A live quote always wins.
- Calling and texting rules vary by country. Use Sendora's quiet hours, consent and do-not-contact controls, and check the rules where you operate. Results depend on your list, offer, setup and market.
- Wallet facts: one prepaid wallet, 100 credits = $1, credits never expire. An email sent costs 1 credit ($0.01).

---

# 1. deliverability-plan (the full email setup)

## What you want
Email infrastructure that is separate from your main domain, authenticated, warmed up slowly, capped, tested and watched, with a recovery plan ready. Sendora handles the mechanics. You make the decisions and approve the money.

## Who does what

| Sendora handles | You do yourself |
|---|---|
| Checks SPF, DKIM and DMARC before a domain sends | Choose the domain names and approve any purchase |
| Automatic warmup ramps and daily caps per mailbox that ramp on a schedule | Add the DNS records at your registrar if you connected a domain you own |
| Sender rotation across a pool | Connect Gmail, Outlook or SMTP mailboxes (needs your sign in, done in the app) |
| One click unsubscribe and suppression of opted-out contacts | Decide your lawful basis for contacting people |
| Bounce handling, reply capture, health and incident reads | Set your own pause thresholds and review weekly |
| Blacklist and placement tests on request | Clean your list and write honest copy |

Claude can only search, quote and prepare. It does not complete a purchase on its own: you approve each one.

## Do it in Sendora

**Phase A. Look before you build (read only, free)**
1. `read_channels` action `domains`. Lists domains with DNS health. Note which exist.
2. `read_channels` action `accounts` (filter channel email). Lists mailboxes and statuses.
3. `read_channels` action `sender_pools` (query `channel` email). Lists existing pools.
4. `read_platform` action `billing` with `billing.wallet`. Read the wallet balance so nothing surprises you.
5. Claude replies with a plain table: Domain | Purpose | Mailboxes | Authentication status. This is the pack's infrastructure table.

**Phase B. Protect the main domain**
6. Decide in writing that the main domain never sends outreach. Nothing in Sendora sends from it unless you add a mailbox on it. Do not add one.

**Phase C. Sending domains (choose A or B)**
7. Option A, connect a domain you already own: `edit_channels` action `connect_domain`, body `{"domain_name": "getyourbrand.example"}`. The result lists the records to add.
8. Add those records at your registrar yourself. You can also let Claude change records Sendora manages with `edit_channels` action `update_domain`, id of the domain, body `{"dns": {...add, update or delete...}}`.
9. Option B, buy a new domain through Sendora:
   - `edit_channels` action `quote_purchase`, body `{"kind": "domain", "selection": {...}}`. Charges nothing and returns a `quote_id` and a total.
   - Claude reads you the item, the price and the wallet it comes from. You say yes or no.
   - On yes only: `run_channels` action `buy`, body `{"kind": "domain", "quote_id": "...", "pay_with": "wallet"}`.
   - Follow it with `read_platform` action `purchases` using the `operation_id` until done, then read the domain back with `read_channels` `domains`.
   - Quotes expire. If the buy says so, quote again and re-confirm. Read `purchases` before retrying, never buy twice after a timeout.
10. Verify DNS: `edit_channels` action `domain_step`, id, body `{"action": "verify_dns"}`. If a record Sendora manages is wrong, `{"action": "repair_dns"}`. If you changed nameservers, `check_nameservers` then `verify_nameservers`. `refresh` re-reads the state.
11. Read the proof: `read_channels` `domains` with `id` and `include: "dns"`. Every record should show a passing state before any mailbox is used. Re-check weekly.
12. Optional on the domain: `update_domain` body fields `forwarding_url` (point the sending domain at your main site, the pack asks for a simple site or redirect), `dmarc_reporting_email` (where DMARC reports go), `catch_all_email`, `mask_forwarding`, `tags`.
13. DMARC: start in monitor mode and tighten later, per the pack. Whether Sendora lets you pick the DMARC policy value in its record is **Check first**: read the domain's `dns` records and tell Claude what you see before changing the policy.
14. Custom tracking domain: the pack prefers your own tracking subdomain or tracking off for early sends. A custom tracking domain is **Check first**. The safe route is the campaign email settings `track_opens: false` and `track_clicks: false` (step 24).

**Phase D. Mailboxes**
15. Option A, connect your own Gmail, Outlook (Microsoft 365) or SMTP mailbox. You do this in the app because it needs your sign in. Then Claude reads it with `read_channels` action `accounts`, id, `include: "profile, health, limits, strength, incidents, preferences"`.
16. Option B, managed mailboxes (Available with setup):
   - `edit_channels` `quote_purchase`, body `{"kind": "mailbox", "selection": {...}}`.
   - You approve the quoted total. Then `run_channels` `buy` with the `quote_id`, then `purchases` until done.
17. The pack's starting setting is 2 or 3 domains and 2 to 3 mailboxes per domain. Tune to your data. Start with the lower end.
18. Set the identity: `edit_channels` action `update_account`, id, body with display name, sender name and signature (real person name, real signature, as the pack asks). Set `preferences` for active hours and weekend mode.
19. Set a cold cap per mailbox: `update_account` body `limits` (daily limit per action). Start low, spread volume across mailboxes. Use your own number here, [YOUR DAILY CAP PER MAILBOX], a starting setting. Sendora's automatic ramp also applies, and a soft cap cannot be overridden past.

**Phase E. Warmup**
20. `edit_channels` action `account_step`, id, body `{"action": "warmup_start"}` for each new mailbox. Sendora ramps volume on a schedule.
21. If any warning appears, `warmup_pause`, later `warmup_resume`. `warmup_stop` ends it. Keep warmup running alongside real sends.
22. `resetup` re-runs setup that failed. `ack_reconnect` dismisses a reconnect prompt after you reconnect. `uncancel` undoes a cancelled managed mailbox.
23. Pack ramp as a starting setting: week 1 warmup only, a handful per mailbox, week 2 small real sends to your best verified leads, hold if a warning appears. Read the mailbox `limits` and `strength` each week.

**Phase F. Sender pool**
24. `edit_channels` action `create_sender_pool`, body `{"name": "Outreach pool A", "channel": "email", "strategy": "least_used"}`. Strategies: `least_used` (even load, good default), `round_robin`, `weighted` (more from stronger mailboxes), `sticky_by_lead` (the same lead always hears from the same sender, best for follow-ups).
25. `update_sender_pool`, id, body `add_members` with your warm mailbox ids. `member_settings` sets `weight` and `paused`. Remove weak or flagged mailboxes with `remove_members` rather than deleting them. Keep pools single channel.
26. Pointing a campaign at the right senders is separate: `edit_campaign` action `update_channel_settings`, id, body `{"config": {"channels": {"email": {"account_ids": [...], "signature": "...", "track_opens": false, "track_clicks": false, "include_unsubscribe": true}}}}`. Binding a pool to a campaign does not by itself change who sends, so always set `account_ids`.

**Phase G. Deliverability tests**
27. Before the first send: `edit_channels` action `start_deliverability_test`, body `{"test": "blacklist", "account_ids": [...]}` and again with `{"test": "placement", ...}`.
28. Read results: `read_channels` action `deliverability_tests` (query `test`), or one by `id`. A placement test is a rough signal, never proof.
29. Repeat weekly while scaling. Also send to your own accounts at several providers and look in spam (a manual spot check).

**Phase H. List hygiene before the first send**
30. Verify addresses. Email verification is optional and costs 1 credit ($0.01) for each conclusive answer. Ask Claude to read the leads tools for the verification step in your workspace, then confirm the cost before running it. Remove role addresses and risky catch-alls where flagged, dedupe, and check suppression. Never send to bought lists of unknown origin.

**Phase I. First send and monitoring**
31. First batch small: enrol a tiny set of your best verified leads, as set up in `channel-order`. Preflight with `read_campaigns` `prepare_outreach`.
32. Daily in the first weeks: `read_channels` `accounts` with `include: "health, incidents, strength"`, and `get_analytics` for the campaign (sends, delivered, bounces, replies). Also use the deliverability and mailbox health reads in analytics.
33. Write your own thresholds in the chat ([YOUR BOUNCE THRESHOLD], [YOUR COMPLAINT THRESHOLD]), starting settings, tune to your data. Ask Claude to report against them each day.
34. Watch for: bounces rising by mailbox or domain, complaint signals, a drop in replies versus sends, a low strength score, a lifecycle reason other than active, a reconnect prompt, authentication failures, blacklist hits.

**Phase J. Recovery (available steps)**
35. Pause at once: `edit_campaign` action `pause` (stops future work, does not recall sent mail). For one mailbox, `account_step` `warmup_pause` and remove it from the pool with `update_sender_pool` `remove_members`, or set the member `paused`.
36. Find the cause with `read_channels` `accounts` (`incidents`, `health`), `domains` `include: "dns"`, and a fresh blacklist test.
37. Fix it: repair DNS (`domain_step` `repair_dns`), clean the list, restore earlier copy if a change caused it (`edit_campaign` `restore_version` on a draft or paused campaign).
38. Rest the mailbox. Restart warmup at a low level, then resume campaigns at a fraction of previous volume with only recently verified contacts.
39. If a domain stays listed after fixes, retire it. Add a fresh domain and mailboxes by the steps above. Never move its volume to the main domain.
40. Disconnect a dead mailbox or domain with `delete_resource` (type `account` or `domain`). Add `confirm_unassign` only if it is in use and you agree.

## What you approve
- Every domain or mailbox purchase: item, quoted price, wallet. One yes per quote.
- Each connection of your own mailbox (you sign in yourself).
- Warmup start for each mailbox, and the cold cap per mailbox.
- Email verification spend on the list (1 credit per conclusive answer).
- The first batch, every volume increase, and the campaign launch (`launch_campaign` only after your explicit "launch").
- Any DNS change Claude proposes.

## Limits and status
- Managed domains and mailboxes: **Available with setup**. Quote and approval needed.
- Connect Gmail, Outlook, SMTP: **Available with setup** (you sign in).
- SPF, DKIM, DMARC checks, warmup ramps, caps, rotation, sender pools: **Available**.
- Whether your agent can complete a purchase from chat or hands you to checkout: **Check first**. The product rule is that you approve every purchase, and some purchases finish at checkout.
- Domain, mailbox and verification prices come from the live quote: **Check first**.
- Custom tracking domain and choosing the DMARC policy value: **Check first** (the pack assumes both).
- Placement tests are a rough signal. There is no inbox placement guarantee, no 100 percent deliverability, and daily caps exist on purpose.
- Pools cannot be deleted while a live campaign uses them.

## Prompt to paste
```
Use my Sendora workspace. Set up cold email infrastructure for [MY BRAND], following the deliverability plan. Main domain: [MAIN DOMAIN], never used for outreach. Plan: about [MY MONTHLY VOLUME] emails in month one.

1. Read my domains, email accounts, sender pools and wallet balance. Give me a table: Domain | Purpose | Mailboxes | Authentication status.
2. Propose 2 or 3 sending domains close to my brand, and 2 to 3 mailboxes per domain, as starting settings. Quote each purchase with quote_purchase and show me the price and the wallet it comes from. Do not buy anything until I say yes to that exact quote.
3. For any domain I own, connect it, list the DNS records I must add, then verify DNS and show the result.
4. Set sender names and signatures, start warmup on each mailbox, and set a low daily cold cap per mailbox that I approve.
5. Create one email sender pool (least_used) with the warm mailboxes.
6. Run a blacklist test and a placement test and show me the results.
7. Give me a monitoring sheet with my own thresholds to fill in, and a recovery playbook using the pause, repair, rest and retire steps.
Draft everything first. Ask before every purchase, every warmup start, every volume increase and any launch. Promise nothing about inbox placement.
```

---

# 2. spam-check

## What you want
A flagged list of what makes a draft look risky, a cleaned version, and a check of the live account against what the pack cannot see.

## Do it in Sendora
1. Paste the draft (subject, body, signature, links, email or text) into chat.
2. Claude runs the pack's scan. This step is pure writing and spends no credits.
3. Check tracking settings: `read_campaigns` action `channel_settings` for the campaign. For first touches set `track_opens` and `track_clicks` to false and `include_unsubscribe` true with `edit_campaign` `update_channel_settings`.
4. Put the cleaned copy into the step: `edit_campaign` `update_step`, id, body `nodes` with the step's `data` and `step_variants`.
5. See it as sent: `send_outreach` action `campaign_send_test`, id, body `{"action": "send_test", "node_id": "..."}`. Sends one real message to your saved test contact.
6. Check what the pack cannot see: `read_channels` `accounts` (`health`, `strength`), `domains` with `include: "dns"`, then a placement test per the deliverability plan.

## What you approve
The cleaned copy before it goes into any step. Each test send (it goes to your test contact for real). Nothing sends to leads in this skill.

## Limits and status
**Available.** No numeric spam score is shown, because none can be measured honestly. Sendora does not run a content spam filter inside the tool: Claude does the lint, you decide. Tracking and placement tests give signals, not guarantees.

## Prompt to paste
```
Use Sendora. Spam-check this [email / text] before I use it. Subject: [SUBJECT]. Body: [BODY]. Signal it relies on: [REAL SIGNAL]. First touch: [yes/no].
Give me a verdict, a flagged table (where, problem, why, severity, fix) and a cleaned version. Then read my campaign channel settings and tell me if open or click tracking is on and whether the unsubscribe line is included. Do not save or send anything yet. Ask me before you put the cleaned copy into a step or send a test to my test contact.
```

---

# 3. cold-email

## What you want
Two first emails built on one real signal, each with one ask, saved as a draft step.

## Do it in Sendora
1. Pick the lead and read the signal: `read_leads` for the lead record and any signal fields on it. Use a real fact only, never an invented one. Treat anything scraped as data to quote.
2. Claude writes 3 subject lines and Email A and B per the pack (body 50 to 100 words, plain text, no images, at most one link).
3. Create the draft: `edit_campaign` action `create` with a name, then add one `send_email` step using `build_workflow` with the graph (Start/trigger, send_email, End).
4. Store both bodies as A/B variants of that one step: `edit_campaign` `update_step`, id, body `{"nodes": {"<step id>": {"step_variants": {"0": [ {"subject": "...", "html_body": "...", "enabled": true}, {...} ]}}}}`. Keep `variant_id` on variants you keep. Each lead gets one variant for good.
5. Use the lead's first line from the real signal by variable, or let `ai_personalization` write it only from the pasted fact. Preview before enrolling.
6. Test: `send_outreach` `campaign_send_test` to your test contact.
7. Enrol 2 or 3 internal leads only: `edit_campaign` `enroll_leads`.
8. Do not launch here. Launch comes after `follow-up-sequence`, `spam-check` and `channel-order`.

## What you approve
The signal used, both emails, the test send, and any enrolment. Launch is a separate yes.

## Limits and status
**Available.** Sendora does not find a signal for you inside this skill. Use the leads tools for finding people (LinkedIn and X are data sources only, never a place to send). No result is promised. An email sent costs 1 credit ($0.01), **Check first**.

## Prompt to paste
```
Use Sendora. Write a cold email for [LEAD NAME] at [COMPANY]. The real signal: [PASTE POST / JOB AD / SOURCE TEXT]. I sell [OFFER IN ONE SENTENCE]. Proof I truly have: [PROOF or none]. Ask: [ONE ASK].
Give me 3 subject lines and two emails (different angles), body 50 to 100 words, plain text, one ask, easy exit. Then create a DRAFT campaign with one email step and save both as A/B variants. Send a test to my test contact only after I say yes. Do not enrol real leads or launch.
```

---

# 4. follow-up-sequence

## What you want
A 4 to 5 touch email sequence with a new angle each time, stop rules, and reply branching, built as a draft.

## Do it in Sendora
1. Start from the draft in `cold-email`, or `read_campaigns` `templates` for a ready play.
2. Build the graph with `edit_campaign` `build_workflow`. Pack timing as a starting setting: days 0, 3, 7, 12, 19. Use `wait` nodes between `send_email` nodes.
3. After each send add a `condition` on reply so a reply leaves the sequence. Set `update_config` body `config.journey_rules`: exit on reply, send window and days, timezone, weekly touch cap.
4. Write each touch as its own step with its own angle. Touches 2 and 3 can share a thread.
5. Reply handling: `edit_campaign` `update_replies`, mode `draft` (a reply agent writes a reply and waits for you) or `off` (a person answers). Pick `draft` first. `auto` replies right away, not recommended for early runs.
6. Add a `human_approval` step before any reply you want to see first. Out of office and unsubscribe are handled by suppression and pause rules in the journey.
7. Validate: `read_campaigns` `validate_workflow`, then `build_workflow` which saves and preflights. Fix every blocker.
8. Senders: `update_channel_settings` with `account_ids` from the pool's mailboxes.

## What you approve
The whole sequence before any enrolment, each reply the agent drafts, and the launch. Unsubscribes and STOP stop everything on their own and go to suppression.

## Limits and status
**Available.** Opens and clicks are not treated as interest. Cooling period after the last touch is yours to set. 5 emails per lead is 5 credits ($0.05), **Check first**.

## Prompt to paste
```
Use Sendora. Build a 5 touch follow-up sequence as a DRAFT campaign after my first email. Reader: [ROLE]. First email and signal: [PASTE]. Real resources I can add: [RESOURCES or none]. Timing (starting setting): days 0, 3, 7, 12, 19, weekdays, reader's morning.
Each touch needs a new angle and one ask. Add reply conditions so any reply, unsubscribe or bounce ends the sequence. Set replies to draft so I approve. Validate and save as a draft. List what is untested. Do not enrol real leads or launch.
```

---

# 5. channel-order

## What you want
Several separate campaigns, each aimed at one audience, with its own channel mix. The order never changes: email, then text, then WhatsApp. AI voice only after interest or a reply.

## The journey (per campaign)

| Step | Node type | Rule |
|---|---|---|
| 1. Start | `trigger` | Audience enters the campaign |
| 2. Email | `send_email` (touches at days 0, 3, 7) | Always the opening channel |
| 3. Reply check | `condition` | Any reply leaves the journey and goes to the voice or human branch |
| 4. Text | `send_sms` (no earlier than day 4) | Only US mobile numbers with permission |
| 5. WhatsApp | `whatsapp_message` (no earlier than day 8) | Only with permission and where it fits |
| 6. Interest gate | `condition` / `ai_decision`, then `human_approval` | Positive reply, booking request or ask to talk |
| 7. AI voice | `voice_call` | Only for someone who asked or replied. Confirms interest and books the meeting |
| 8. Stop | shared suppression | STOP, unsubscribe, wrong person, complaint, bounce ends every channel |

Never a cold call. LinkedIn and X are data sources only. Do not add any LinkedIn node.

## Do it in Sendora
1. Split into campaigns, one audience and one outcome each: `edit_campaign` `create` per campaign (for example: people engaging with a competitor, recent hires, recently funded companies). Group them with `create_folder` and `move_to_folder`.
2. Keep the channels each audience has permission for. No mobile permission means email only. Remove text and WhatsApp from that campaign entirely.
3. Build each graph with `build_workflow`, using the node order in the table. `read_campaigns` `validate_workflow` first. Edges need real node ids and handles.
4. Channel settings per campaign with `update_channel_settings`:
   - email: `account_ids` (warm mailboxes), `signature`, tracking off for early sends, `include_unsubscribe` true.
   - sms: `account_ids` (numbers that can send texts), `max_length`, `stop_keyword_reply`.
   - whatsapp: `account_ids` (connected accounts). `use_template` true needs an approved `template_name` first.
   - voice: `account_ids` (numbers that can place calls).
5. Timing and hours: `update_config` `journey_rules` (exit on reply, send window, days, timezone, weekly touch cap). No text or WhatsApp outside local daytime.
6. Texts and chat: an SMS or WhatsApp step can name a `chat_agent_id` with an empty message so a chat agent writes the opening. This only works if chat agents are on for your workspace. Chat agents that start conversations are **Rolling out**, especially on WhatsApp and email. Reply routing: `update_replies` mode `draft` with `agent_id`.
7. Voice step: attach a published voice agent (see `inbound-voice-prompt`). Put a `human_approval` node before the `voice_call` so you see who is about to be called.
8. Preflight: `read_campaigns` `prepare_outreach` and `edit_campaign` `step` action `preflight`. Clear every blocker, including unregistered texting.
9. Test with 2 or 3 internal leads: `enroll_leads`, then `send_outreach` `campaign_test_run` (sends real messages to your test contact, ask first).
10. First small real batch only after you say go: `launch_campaign`. Check at 1 hour, 4 hours and next day with `get_analytics` and `read_channels` `accounts` (`health`, `incidents`).
11. Pause rule: `edit_campaign` `pause` if bounces, complaints or opt-outs jump past the thresholds you set. `stop_campaign` is permanent, only on your clear instruction.

## What you approve
Campaign map, each audience list before enrolment, every message template, the first batch, every non-trivial reply, every `voice_call` target, SMS registration fees, and the launch.

## Limits and status
- Email, text: **Available with setup**. Text needs US 10DLC registration and a number (see `text-whatsapp-compliance`).
- WhatsApp: **Available with setup**. It is account pairing by QR or pairing code, not the official Business API.
- Chat agents that start conversations: **Rolling out**.
- Voice after interest: **Available with setup**. Voice is inbound first. The call goes to someone who asked, never a cold list.
- Costs, **Check first**: email 1 credit ($0.01), US text about $0.017, voice $0.10 a minute, a phone number $3 a month, an extra WhatsApp seat $30 a month.
- A person who is in a running campaign cannot be started in a separate chat by an agent.

## Prompt to paste
```
Use Sendora. Plan my outbound as separate campaigns, one per audience. Audiences and permission:
- [AUDIENCE 1], email only
- [AUDIENCE 2], email plus text consent
Order is fixed: email, then text, then WhatsApp, with AI voice only after a reply showing interest. LinkedIn and X are data sources only.
Build each as a DRAFT campaign (trigger, emails on days 0, 3, 7, then text no earlier than day 4 and WhatsApp no earlier than day 8 only where permitted, reply condition, human_approval, then voice_call only for people who replied). Any reply, STOP, unsubscribe or bounce ends all channels. Validate and preflight. Show me the journey in plain words and what is untested. Do not enrol real leads, buy anything or launch without my yes.
```

---

# 6. text-whatsapp-compliance

## What you want
A go or no-go check for every audience before any text or WhatsApp goes out, with the setup that makes STOP automatic.

This is a founder checklist, not legal advice. Calling and texting rules vary by country. Use Sendora's quiet hours, consent and do-not-contact controls, and check the rules where you operate. You stay responsible for lawful basis and content.

## Do it in Sendora
1. Run the pack's permission check per audience in chat. Verdict: Go, Email only, or No. Bought or scraped numbers are No for text and WhatsApp. Numbers are US mobile only.
2. Readiness: `read_channels` action `sms_readiness`. It says if the workspace can text, what is next, and gives exact form option values.
3. Registration (US 10DLC): `edit_channels` `create_sms_registration` (type brand) with your real legal name, address, contact, website. Then `sms_registration_step` `send_code` and `verify_code`, `update_sms_registration` until complete.
4. Optional paid trust check: `run_channels` `request_sms_vetting`. Quote and approval first.
5. Submit the brand: `run_channels` `submit_sms_registration`. Charges the wallet. Get the fee from `sms_readiness` with `quote`, say it, wait for yes.
6. After approval, create the campaign registration (use case, truthful sample messages with opt out, exact opt in flow) and submit it the same way. A mismatch between registration and real messages is the usual cause of rejection. Plan for days.
7. Number: `quote_purchase` with `kind: "phone_number"`, approve, `run_channels` `buy`. Then `attach_sms_numbers` (charges).
8. Follow review in `read_channels` `sms_registrations` with `include: "events"`. If rejected, `sms_registration_step` `appeal` or `clone`, fix, resubmit.
9. First message wording, opt out in the first text: for example "Reply STOP to opt out." Set `stop_keyword_reply` with `update_channel_settings`.
10. STOP and opt-outs go on the do-not-contact list automatically and stop all channels. Test it with a team phone before launch.
11. WhatsApp: connect the account by QR or pairing code yourself in the app. Check your account type and country rules.
12. Records: keep the consent proof, wording, date and every opt-out outside the chat (a spreadsheet works). Ask a lawyer how long to keep them.

## What you approve
Each registration submission fee, vetting, number purchase and attach, the audience, the first message and batch size, and the quiet hours.

## Limits and status
- 10DLC registration and texting: **Available with setup**. Country and carrier coverage for SMS: **Check first**.
- Texts and WhatsApp respect quiet hours, consent and do-not-contact checks. Chat agents starting a chat refuse outside 8 AM to 9 PM for the contact.
- WhatsApp: **Available with setup**, not the Business API. Approved templates apply only if you set `use_template`.
- Not covered: the legal answer. Sendora does not give legal advice and the checklist does not make a number lawful.

## Prompt to paste
```
Use Sendora. Run the text and WhatsApp compliance checklist for [AUDIENCE]. How I got the numbers and what each person agreed to: [DESCRIBE]. US mobile numbers only. First give me a verdict per audience (Go / Email only / No) and a list of questions for a lawyer. Then read sms_readiness and tell me what registration steps remain, with the fee from a quote. Draft the first message with who I am and STOP wording. Do not submit, buy or send anything until I say yes to each fee.
```

---

# 7. inbound-voice-prompt

## What you want
A voice agent that only talks to people who asked: answering calls into your number and following up with those who replied or requested a call. It confirms interest and books a meeting.

## Do it in Sendora
1. Read what exists: `read_voice` `list_agents`, `agent_templates`. Phone numbers: `read_platform` `phone_numbers`.
2. Claude fills the pack's prompt template (role, AI disclosure, who it speaks to, opening per situation, 3 to 5 questions, booking, facts it may state, objections, opt out, handoff, voicemail, never list).
3. Validate then draft: `check_voice_agent` `validate`, then `edit_voice` `save_draft` with `system_prompt`, `first_message`, `direction`. Settings to start from, change one at a time: `first_speaker` agent for inbound and widget, `user` for a follow-up call so the person says hello first. `temperature` 0.3 to 0.5, `voice_speed` about 1.0, `recording_enabled` only where lawful with a disclosure line, `voicemail_enabled` with a short script.
4. Facts go in a knowledge base: `create_knowledge_base`, `add_kb_text`, `assign_kb`. Behaviour stays in the prompt.
5. Booking: `create_tool` for the booking tool, with `read_voice` `meeting_types` for the calendar. Attach with `update_agent` `tool_ids` (send the full list).
6. Number: if you need one, `quote_purchase` kind `phone_number`, your yes, `run_channels` `buy`, `purchases` to follow. Verify the caller ID: `start_caller_id_verification` then `confirm_caller_id`. Assign: `edit_platform` `update_number_settings` with `assigned_agent_id`, then `save_channel_assignments`.
7. Inbound settings: `update_inbound_config` (`capture_leads`, `lead_defaults`, `after_hours`).
8. Test, cheapest first: `test_config`, `run_voice` `browser_test`, then `phone_test` to your own phone. Run the pack's 3 test calls (says "stop", asks "are you real", asks for an unlisted price).
9. Publish after your yes: `run_voice` `publish` with a `confirmation_id`, then `check_voice_agent` `verify`.
10. Follow-up use: a `voice_call` step in a campaign behind the interest gate (see `channel-order`), or `run_voice` `call` for one named lead who asked. Check caller ID, quiet hours and do-not-contact first.
11. Review: `read_voice` `list_calls` and `get_call` for the transcript. Treat recordings as sensitive.

## What you approve
The prompt, the facts list, the 3 test calls, any number purchase, the publish, and every real call (lead and agent named). The agent says it is an AI assistant if asked.

## Limits and status
- Voice agents and inbound numbers: **Available with setup**. Voice is $0.10 a minute and a number $3 a month, **Check first**.
- Voice languages depend on the chosen voice and model: **Check first**.
- Call recording: available where enabled and legally permitted. You are responsible for any consent that applies.
- No cold calling. The agent should end the call if it cannot confirm the person asked.
- Not legal advice: check calling, recording and disclosure rules where you operate.

## Prompt to paste
```
Use Sendora. Create a voice agent as a DRAFT only for people who asked to hear from us. Business: [BUSINESS]. Human for booking: [NAME]. The call is to confirm interest and book a meeting. Who it speaks to: [e.g. people who replied "interested" to my email, plus calls into my number].
Fill the inbound voice prompt: AI disclosure, an opening per situation, 4 qualifying questions: [Q1..Q4], objection handling, opt out, handoff, a one-time voicemail, and a never list (no cold calls, no invented facts or prices, no pushing after a no). Facts it may state: [FACTS]. Validate and save the draft. Then give me 3 test calls to run. Do not buy a number, publish or place any real call until I approve each step.
```
