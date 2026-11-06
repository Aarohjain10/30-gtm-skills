# Sendora companion 03: reply, measure, agency

This part maps nine pack skills to exact steps inside Sendora, plus an honest note on the three content skills. It assumes Claude is connected to Sendora over MCP.

How to read it:
- Reads are safe. Anything that changes something or reaches a real person is a write. Claude should show you the exact change and wait for your yes before each write.
- Status labels: Available, Available with setup, Rolling out, Check first.
- Results depend on your list, offer, setup and market. Nothing here is a guarantee. Calling and texting rules vary by country, so use quiet hours, consent and do-not-contact controls, and check the rules where you operate.

---

## 1. reply-triage

**What you want**
Every reply sorted, interested people answered first, and nobody who said stop contacted again.

**Do it in Sendora**
1. `read_conversations` action `needs_attention` (with `limit`). This lists everything waiting on a person, most urgent first, with counts.
2. `read_conversations` action `list`. Filter with `channel` (email, sms, whatsapp) and `traffic_light` (GREEN interested, YELLOW neutral, RED not interested).
3. `read_conversations` action `get` with `lead_id` to read the full thread across channels before classifying.
4. Safety first: for anyone who asked to stop, `edit_conversation` action `conversation_step` with body action `pause_outreach` for that lead, then add them to do-not-contact (`edit_leads`, see the leads guide). Confirm the exact lead first.
5. `edit_conversation` action `set_traffic_light` with `lead_id` and `traffic_light` to record your classification. Interested is GREEN, not now or a question is YELLOW, not interested is RED.
6. Open the waiting items: `read_conversations` actions `approvals` (status pending), `escalations` (status), `tasks` (status, lead_id).
7. Decide them with `edit_conversation`: `decide_approval` (body decision approved or rejected), `update_escalation` (body status; resolving needs resolution_type retry, skip or manual), `update_task` (body status open, in_progress, completed, cancelled).
8. For "not now" or out of office: `conversation_step` body action `snooze` with `snoozed_until`. When handled: `mark_read`. To clear: `archive`.
9. If a chat agent is in draft mode, its replies wait as drafts. Read them with `read_conversations` `get`, then send or `discard_draft`.

**What you approve**
Every reply that goes out (show text, recipient, channel), every approval you decide, and any move to do-not-contact. Claude does not approve on your behalf without a clear yes or a standing rule you set.

**Limits and status**
- Inbox, traffic light, approvals, escalations, tasks: Available.
- Chat agents that start conversations: Rolling out. Answering inbound email, SMS and WhatsApp is Available with setup.
- The pack's eight reply types are not eight fields in Sendora. Sendora has the traffic light, status, snooze, tags and pipeline stage. Claude maps the eight types onto those and can note the type as a tag via `edit_leads` `update` (`add_tags`).
- Response-time targets are not enforced by the product. Claude reads `needs_attention` to remind you.

**Prompt to paste**
```
Use my connected Sendora workspace. Read only for now. Run needs_attention, then list unread conversations, and read each thread. Classify every reply into the eight types from the reply-triage skill, with a priority, an owner and a next action. Put anyone who asked to stop at the top. Show me the table and the draft replies. Do not send, approve, pause, tag or change anything until I say yes to each batch.
```

---

## 2. objection-replies

**What you want**
A short, honest written answer to a "too expensive", "send info" or "not now", ready to send or approve.

**Do it in Sendora**
1. `read_conversations` action `get` with `lead_id` to read the objection and the message it answers.
2. `read_leads` actions `get` and `activities` for context on that person (what they were sent, what they did).
3. Claude writes two drafts (softer and more direct), using the 90 word rule in the pack skill.
4. To send as an email or text reply: `send_outreach` action `email` or `message`. Claude shows the exact text, recipient and channel and waits for your yes.
5. If an AI draft is already waiting, send your edited version and clear the old draft with `conversation_step` body action `discard_draft`.
6. Not ready: `conversation_step` `snooze` with `snoozed_until`. Not interested: `set_traffic_light` RED and `pause_outreach`.

**What you approve**
Each send. Claude does not send objection replies automatically.

**Limits and status**
- Reading, drafting, sending a reply: Available. Email needs a connected mailbox, text needs a number: Available with setup.
- Sendora does not have an "objection library" feature. The playbook lives in the pack skill and in your chat agent instructions.
- If you want an AI chat agent to handle objections on its own, put the playbook in its instructions and run it in `draft` mode first. Starting conversations from a chat agent is Rolling out.

**Prompt to paste**
```
Use my Sendora workspace. Open the thread for [lead name or id] and read the full conversation and lead activity. Using the objection-replies skill, work out what the real concern is, then write a softer and a more direct reply under 90 words each. Mark both as drafts. Do not send anything. If I pick one, show me the exact text, recipient and channel, and only then send with send_outreach.
```

---

## 3. meeting-booking

**What you want**
Interested people turned into booked, confirmed calls on your calendar.

**Do it in Sendora**
1. Check the calendar link first. `read_platform` (integrations) to see whether Cal.com, Calendly, Google Calendar or Outlook Calendar is connected. If not, connect it in the app (see the workspace guide).
2. `read_conversations` action `meetings` with `start` and `end` ISO times to see what is already booked. Add `include_cancelled` if useful.
3. Read the thread with `read_conversations` `get`. Reply on the channel they used, with `send_outreach` `email` or `message`, offering two or three times in their time zone plus your booking link. Wait for your yes before sending.
4. When they accept, record it: `edit_conversation` action `create_meeting` with body `prospect_name`, `scheduled_at` and optional details. **This sends no invite.** Send the invite from your calendar or booking tool.
5. Move the lead: `conversation_step` body action `mark_meeting_booked` (or `set_pipeline_stage` with `pipeline_stage`).
6. Reschedule or cancel a manual record: `edit_conversation` `update_meeting` with body status `rescheduled`, `confirmed` or `cancelled`.
7. No-show: send one friendly note with two new times via `send_outreach`. The template library has a Meeting No-Show Recovery play you can use for a campaign version.

**What you approve**
Each outbound message, the meeting record, and any stage move.

**Limits and status**
- Meeting records, calendar connection: Available with setup. Agents proposing and booking inside real availability is Available with setup via the calendar connection.
- `create_meeting` records only, it sends no invite.
- Sendora does not send your day-before and one-hour reminders as a built-in feature. Use your calendar tool's reminders.
- Voice to confirm interest is for people who replied or asked. Never a cold call.

**Prompt to paste**
```
Use my Sendora workspace. [Name] replied with interest on [channel]. Read the thread and my booked meetings for the next 14 days. Write a booking reply in their time zone with three time options and my booking link, as a draft. Show me the exact text and recipient. Only after I say yes, send it. When they confirm a time, record it with create_meeting and mark the lead as meeting booked, then remind me to send the calendar invite because Sendora does not send one.
```

---

## 4. discovery-call-prep

**What you want**
A one page prep sheet before a call with someone who replied or booked.

**Do it in Sendora**
1. `read_conversations` action `meetings` (window around the call) to find the meeting and the lead.
2. `read_conversations` action `get` with `lead_id` for the full thread across channels.
3. `read_leads` action `get` for profile, enrichment and flags. `read_leads` action `activities` for what they were sent and did. `read_leads` action `context` with `include` of `ai_state, outreach_state, callbacks, campaigns, recordings`.
4. If the person talked to a voice agent: `read_voice` actions `list_calls` and `get_call` for the transcript.
5. Optional web research on a high value lead through the lead research action (`enrich_leads`, research_web). It may use credits, so Claude should tell you before running it.
6. Claude builds the prep sheet from the pack skill, separating facts (with source) from guesses.
7. After the call: `edit_conversation` `conversation_step` `set_pipeline_stage`, and `update_task` to close any task.

**What you approve**
Any paid lookup, and any stage or tag change afterwards. Nothing is sent by this skill.

**Limits and status**
- Reading threads, lead profile, calls: Available. Call transcripts need a voice agent set up: Available with setup. Recording where legally permitted.
- The prep sheet itself is written by Claude, not a Sendora feature, and is not saved in the product unless you paste it into a lead note or your CRM.
- Check first: confirm the exact name of the web research action in your tool list before relying on it.
- Sendora does not join your video calls or take notes.

**Prompt to paste**
```
Use my Sendora workspace, read only. Find my next booked meeting, then read the full thread, the lead profile, activities and context, and any call transcript for that person. Using the discovery-call-prep skill, write a one page prep sheet: facts with sources, guesses marked as guesses, likely pains as hypotheses, up to 7 open questions, signals to listen for, and one next step to propose with a smaller fallback. Do not run paid lookups or change anything.
```

---

## 5. weekly-review

**What you want**
Real numbers, one clear decision per campaign, and one experiment for next week.

**Do it in Sendora**
1. Totals this week: `get_analytics` action `overview` with `days` 7. Last week for comparison: `overview` with `start_date` and `end_date` set to the previous 7 days (inclusive, YYYY-MM-DD).
2. Rank campaigns: `get_analytics` action `top_campaigns` with `metric` meetings, then replies; `window` 7d; `limit` 10.
3. Each live campaign: `get_analytics` `campaign` with `campaign_id`, then `funnel` (sent, delivered, opened, clicked, replied, meeting booked) for 7 days.
4. A/B reads: `get_analytics` `email_variants` with `campaign_id` (sends, opens, replies per variant for each email step). For one step: `read_campaigns` action `variants` with the campaign `id` and the step's `node_id` (variant_key, weight, is_control, assignments, outcomes). Empty until the campaign has sent. Claude should write the winner rule before reading the result.
5. Reply quality: `get_analytics` `outcomes` with `outcome_type` reply (and call if voice is used).
6. Health, in diagnosis order: `deliverability` with `window` 7d (bounce rate and band, DNS health), `mailboxes` (warmup, sent today, cap), `accounts_health`, `send_cap`.
7. Spend and AI: `ai_spend` with `dimension` feature, `window` 7d. Credit use: `read_platform` action `billing`.
8. People follow ups: `get_analytics` `action_queue` (limit up to 100) and `read_conversations` `needs_attention`.
9. Calls, if used: `get_analytics` `calls` and `agents`.

**What you approve**
Any change Claude proposes after the review (pausing a campaign, editing a step, launching a variant). The review itself is read only.

**Limits and status**
- All analytics reads: Available. The pack's formulas (positive reply rate, meeting rate per positive reply) are computed by Claude from these counts. Sendora's traffic light is the closest thing to "positive".
- Use the smallest window that answers the question and say the window in the answer. A report returning zero or empty usually means nothing happened in the window.
- If a report returns 503 or times out, Claude should say it is unavailable and not guess.
- A variant stays on one lead for the whole journey. Small samples do not prove a winner.
- Check first: no benchmarks are built in, so compare to your own best week only.

**Prompt to paste**
```
Use my Sendora workspace, read only. Run the weekly-review skill. Pull overview for the last 7 days and the 7 days before that, top_campaigns by meetings and replies for 7d, funnel and email_variants for each live campaign, deliverability for 7d, mailboxes, send_cap and action_queue. Compute reply and meeting rates from the real counts, compare only to last week and my best week, and give a keep, fix, scale or cut verdict per campaign. Name the one change to make first using the diagnosis order, and one A/B experiment with the winner rule written before it runs. Do not change anything.
```

---

## 6. reactivation-play

**What you want**
Old leads, lost deals and dormant contacts brought back into conversation, safely.

**Do it in Sendora**
1. Find the dormant people. Option A, from a connected CRM: `find_prospects` action `prepare_intent_run` with `list_id`, `source_type` `crm_dormant_reactivation` and `config` containing `connection_id`, `dormant_days` (1 to 730) and `limit` (up to 2000). It checks the config and saves the run, with no leads added and no charge. Then `start_intent_run` with `list_id` and `source_id` (this writes leads and may cost credits).
2. Option B, from a file: `check_lead_import` action `prepare` (validate plus preview), then `edit_leads` action `import`, then `check_lead_import` `verify_import` and `verify_list`.
3. Create a list first with `edit_leads` `create_list`, and tag by segment with `edit_leads` `bulk` action `add_tags`.
4. Suppress first: confirm do-not-contact, bounces and opt-outs, and keep open-deal contacts out. A lead on the do-not-contact list is not contacted even if re-imported.
5. Optional hygiene: `enrich_leads` action `verify_emails` (charged per conclusive answer, 1 credit, $0.01).
6. Build the sequence: use the template library play "Closed-Lost Reactivation" as a start. See `sendora-campaigns-technical`: create, validate with preflight, and use a small test slice first via enrolment of a few leads.
7. Launch only on your yes: `launch_campaign`. Pause or stop with `stop_campaign`.
8. Handle replies with the reply-triage steps above. Add a Human Approval step before sends if you want to read each message.

**What you approve**
The intent run start (credits), the import, the list and suppression, the copy, the test slice, the launch, and each reply.

**Limits and status**
- CRM sync (HubSpot, GoHighLevel, Close): Available with setup. Intent runs are one time runs, not recurring schedules.
- Email: Available with setup (a connected mailbox). Text: Available with setup, US mobile numbers only for lookups, with the consent your market needs. Voice only for people who replied or asked, never a cold list.
- Social sources (LinkedIn post engagement, X replies) are data sources only. Public data is not consent to contact.
- Sendora does not decide your lawful basis. You do.
- Price facts: email sent 1 credit ($0.01), US mobile lookup 8 credits ($0.08), work email found 2 credits ($0.02). Check first: confirm live prices before quoting.

**Prompt to paste**
```
Use my Sendora workspace. I want to reactivate dormant contacts. Read my connected CRM connections and lead lists first. Then prepare (do not start) a crm_dormant_reactivation run with dormant_days [90 or your number] and a limit of [your limit] into a new list, and show me the config and the cost implications. Draft a 3 email reactivation sequence per segment using the reactivation-play skill. Do not start the run, import, verify, create or launch anything until I confirm each step.
```

---

## 7. referral-outreach

**What you want**
Introductions from happy customers and partners, asked for politely.

**Do it in Sendora**
1. Build the ask list from your leads: `read_leads` action `list` with your customers list (or `segments` and `run_segment` for "customers"). `read_leads` `activities` and `get` for recent contact and positive replies. Positive replies: `read_conversations` `list` with `traffic_light` GREEN.
2. Skip anyone with an open complaint: Claude checks `needs_attention` and tasks for that lead.
3. Claude writes the ask, the forwardable blurb, one follow up and the thank you, from the pack skill.
4. Send one by one with `send_outreach` `email` after your yes, or enrol a small group in a short campaign you built with the campaigns guide, with a Human Approval step.
5. Log it: tag with `edit_leads` `update` (`add_tags`, for example "referral asked") and create a task with `update_task` or the app's task list so the follow up is not forgotten.
6. When an intro arrives, create the new lead with `edit_leads` `create`, and tag the source.
7. Review the log in the weekly review.

**What you approve**
Every ask and every thank you before it goes. Any incentive wording.

**Limits and status**
- Sendora has no referral program tool, referral links or reward tracking. Tags, tasks and stages are the log.
- Email: Available with setup. Do not message customers through a cold campaign tool at volume. This is one favour per person.
- Incentives and gifts are your decision under your own rules. Sendora does not handle them.

**Prompt to paste**
```
Use my Sendora workspace. Find my customers list and rank who to ask for an introduction, using recent positive replies and long tenure, and skip anyone with an open issue. For the top 5, write a short ask with a forwardable two line blurb, as drafts. Show me the exact text, recipient and channel for each. Send nothing until I say yes per person. After sending, tag each as "referral asked" and create a follow up task for 5 days later, only with my confirmation.
```

---

## 8. agency-offer-packaging

**What you want**
A client offer you can sell, priced with your own markup, that makes you recurring revenue. The offer is selling AI-assisted outbound to your clients, with everything approved by them first.

**Do it in Sendora**
1. See what exists: `read_agency` actions `packages` (with `include` pricing), `pricing` (published markup, any draft, each rate's retail price) and `list_clients`.
2. Your cost base: the white label plans are White Label Basic at $297 a month for 10 client workspaces and White Label SaaS at $497 a month for unlimited. Check first: confirm live prices before you put them in a proposal.
3. Set your markup: `edit_agency` action `save_pricing` with body `default_markup_bps` (basis points, 100 bps is 1 percent). Review the draft retail prices with `read_agency` `pricing`. Then `edit_agency` action `publish_pricing` (body publish true and `policy_id`).
4. Create tiers: `edit_agency` action `create_package` with body `name`, `monthly_price_cents` and `included_credits`. Do Starter, Growth and Scale. Package creation needs published pricing first. Change a package later with `create_package_version` (do not edit a live version in place).
5. Each client funds their own wallet. Your markup sits on top of their usage, and you see margin per client in `read_agency` `client_details` (30 day usage and revenue).
6. Brand it so it looks like yours: `edit_agency` `client_branding` (name, tagline, logos, colours, `custom_domain`) and `update_experience_draft`, then `publish_experience_draft` on a yes. `rollback_experience` with `version_id` if something is off.
7. Onboarding path: `edit_agency` `signup_onboarding` (`signup_mode` invite_only or self_serve, `onboarding_welcome_message`, `onboarding_questions`, `portal_slug`, `signup_trial_days`).
8. Protect your reputation across clients: `update_client_limits` with body `usage_caps` and `access`, per client.

**What you approve**
Publishing pricing, creating or versioning a package, publishing the client experience, and any limit change. Say how many clients a change touches before it runs.

**Limits and status**
- Client workspaces, packages, markup, wallets, branding: Available. Custom domain serving: Available with setup (DNS verification). A client portal with welcome message and slug: Available.
- Price build: Claude uses your real costs. Sendora does not give you a margin target.
- Sendora does not write your pitch or take payment from your clients on your behalf beyond the package and wallet setup. Check first: confirm how your clients pay before promising a billing flow.
- Do not promise guaranteed meetings, replies or revenue to your clients. Do not offer LinkedIn or X outreach (data sources only) or cold calling.
- Agency tools need agency scopes and client clearance. Viewers and client roles are refused.

**Prompt to paste**
```
Use my Sendora agency account. Read only first: list my packages, pricing and clients. Then, using the agency-offer-packaging skill and my real costs [fill in], draft three tiers (Starter, Growth, Scale) and a markup in basis points, show me the retail price per tier and my margin per client. Do not save pricing, create packages, edit branding or publish anything until I approve each step, and tell me how many clients each change would touch.
```

---

## 9. client-report

**What you want**
A monthly report your client can read in two minutes, with real numbers only, that shows why they keep paying you.

**Do it in Sendora**
1. Pick the client: `read_agency` `list_clients`, then `client_details` (status, plan, health, 30 day usage and revenue).
2. Pull the month's numbers for that client. Use `get_analytics` with the client's `workspace_id` (cleared direct workspace) and `start_date` and `end_date` covering the month: `overview`, `top_campaigns` (window 30d), `funnel` and `outcomes` per `campaign_id`, `meetings`, `deliverability` (window 30d).
3. Last month for the change column: the same calls with last month's dates.
4. What changed during the month: `read_agency` `client_limits` with `include` of `limits, audit` shows your own changes.
5. Booked meetings with detail: `read_conversations` `meetings` with `start` and `end`.
6. Claude marks any missing figure "Not tracked this month" and writes the report from the pack template.
7. Deliver it yourself by email: `send_outreach` `email` to the client contact, only after you read and approve it.

**What you approve**
The report text and every send. You also decide what to share about problems.

**Limits and status**
- Analytics reads: Available. Reading another workspace through `workspace_id` needs agency clearance. Check first: if your connection is not cleared for that client, Claude should say so and ask you to paste the numbers.
- Sendora does not generate a PDF or a branded report file through MCP, and does not email reports to clients on a schedule. The client portal shows usage and views to the client. Claude writes the text and you send it.
- Sendora does not track revenue your client earns. Do not report client revenue unless they gave it to you.

**Prompt to paste**
```
Use my Sendora agency account. For the client [name], read client_details, then pull the real numbers for [month] and the month before: overview, top_campaigns, funnel and outcomes per campaign, meetings and deliverability. Using the client-report skill, write the report with only figures from those reads, mark gaps as "Not tracked this month", and lead with the honest one line summary. If you cannot read that client's workspace, tell me and stop. Do not send anything.
```

---

## 10. Content skills: linkedin-post-writer, lead-magnet-creator, case-study-writer

**What Sendora does for these**
Nothing directly. Sendora is not a publishing, scheduling or content tool.

**What Sendora does not do**
- It does not post to LinkedIn or any social network, schedule posts, or watch your comments to reply for you.
- It does not message people on LinkedIn. LinkedIn is only a place data can come from.
- It does not host your lead magnet, build landing pages or send files from a comment keyword.
- It does not store customer testimonials or write approved case studies. No customer result or case study is approved to quote from Sendora itself. Use only your own real results, with permission.

**What can help, honestly**
- Claude with Sendora connected can read your real numbers to feed these skills. For a real proof opener: `get_analytics` `overview` or `meetings` for a real window, or `read_conversations` `meetings`. Use the exact figure, state the window, and keep any "[REAL ...]" placeholder until you have it. For case study numbers, `get_analytics` `funnel` for the campaign.
- Delivery after someone comments: you send the link yourself in the platform. If they gave you an email, add them as a lead with `edit_leads` `create`, tag the source, and send one email with `send_outreach` `email` after your yes. Only to people who asked.
- Intent sources can find people who commented on a post: `find_prospects` `prepare_intent_run` with `source_type` `linkedin_post_engagers` (profile or post URLs) is a data source for building a list, not a way to post or reply. Public engagement is not consent to contact, so treat it with care.
- Reuse real replies (with permission) for proof: `read_conversations` `get`.

**Status**
Lead magnet delivery by email: Available with setup. Everything else here is outside the product. The case study permission checklist stays with you.

**Prompt to paste**
```
Use my Sendora workspace, read only. Pull my real overview for the last 30 days and my booked meetings for that window, and list every number with the window and where it came from. Then use the linkedin-post-writer skill with those numbers only (use [REAL ...] placeholders for anything missing) and write a draft post. Do not post, send or change anything in Sendora. Remind me that Sendora cannot publish the post or deliver the file for me.
```
