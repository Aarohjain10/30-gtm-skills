# 01. Offer and leads: do it in Sendora

Covers six pack skills. Assumes you did 00-connect.md: Claude is connected, reads first, and confirms before every write. Status labels: Available, Available with setup, Rolling out, Check first.

---

## business-interview

**What you want:** A short, approved business profile (what you sell, who buys, real proof, voice, rules) that every later step reads.

**Do it in Sendora**
1. Read what the workspace already holds: read_platform, action settings, include "org,company_profile,mailing_address". Also read_leads, action lists.
2. Ask the pack's six questions in one message. Mark answers Stated, Inferred or Missing. Do not guess Missing ones.
3. If you have a website, let Claude read it and say back what it understood. Ask only for what it could not find.
4. Write business-profile.md as a draft in the chat or a local file. Mark it APPROVED only after you say yes.
5. Note the setup gaps that matter for later: mailing address (email needs one), connected sending account, calendar. Claude reads these but changes nothing.

**What you approve:** The profile text, especially the Proof I really have list and your rules. Nothing is written to Sendora in this step.

**Limits and status:** Reading the workspace is Available. Sendora has no profile-interview tool, so the profile lives in your chat or file. Saving it as agent knowledge needs a knowledge base, which is Available with setup and varies by workspace (Check first). No customer proof is pre-approved: use only what you can show.

**Prompt to paste**
```
Use the business-interview skill. Read my Sendora workspace first (settings, company profile, lead lists) and tell me what exists. Then ask me the six questions in one message. Write business-profile.md as a DRAFT. Do not change anything in Sendora.
```

---

## icp-builder

**What you want:** A clear ideal customer with tiers (core, adjacent, exclude), filters and signals, plus a small test list before you spend.

**Do it in Sendora**
1. Make sure you have a list to hold the preview. Read read_leads, action lists. If none, ask me before edit_leads, action create_list.
2. Turn your buyer description into filters: read_prospect_previews, action parse_criteria, with list_id and a plain-English description. It never charges and never searches.
3. Review the returned filters with me. Filter groups: people (titles, exclude_titles, levels, functions, countries), company (industries, employee_ranges, keywords, funding_types, countries), hiring (titles, description_keywords, posted_within_days).
4. Preview: find_prospects, action preview_search, mode people, page_size 25, and a list_id. No leads are created and nothing is charged.
5. Check a sample of about 20: name, title, company, location. If more than 2 in 20 are not buyers, tighten titles and add exclude_titles such as assistant, intern, student, consultant. Preview again.
6. Do not commit yet. Hand the approved filters to the next skills.

**What you approve:** The filters, the exclude list, and the sample before any commit.

**Limits and status:** Available. Searching and previewing cost nothing. A search needs at least one filter and a manager, admin or owner role. Allowed filter values are fixed lists, and countries are two-letter codes. Sendora has no filter for "decision maker" beyond titles and levels.

**Prompt to paste**
```
Use the icp-builder skill with my business-profile. In Sendora, run parse_criteria on my buyer description, show me the filters, then run preview_search (people, 25 results) into a list I name. Do not commit anything. Show me the sample, flag titles that do not look like buyers, and propose tighter filters. Ask before creating the list.
```

---

## competitor-engagers

**What you want:** A ranked list of people publicly engaging with your competitors or your topic right now, ready to enrich.

**Do it in Sendora**
1. Create or pick an empty list (edit_leads, action create_list) so a test run cannot mix with other leads. Confirm the name with me.
2. Prepare, do not start: find_prospects, action prepare_intent_run. It checks the config, saves the run and returns a source_id. No leads, no charge.
3. Pick the source_type that fits what you can name:
   - linkedin_post_engagers: profile_urls and/or post_urls (up to 20 each), capture commenters true, likers false, limit 50 to start.
   - x_reply_miner: post_urls and/or creator_handles (up to 10), max_replies.
   - youtube_comment_engagers: video_urls (1 to 20), max_comments.
   - zach (review authors): niche_description plus exact G2, Trustpilot or Capterra product pages, up to 5 each. It cannot guess the product.
4. Turn on ai_qualification with a clear icp_prompt and only_qualified true. Raw engagers include many non-buyers.
5. Show me the config and the expected cost. Cost depends on auto_enrich contact lookups. After my yes: find_prospects, action start_intent_run with list_id and source_id. This writes leads and may spend credits.
6. Follow the job (read_platform, action jobs). Then read_leads, action list for the list. Rank by signal strength then freshness in the chat.

**What you approve:** The source posts or pages, the icp_prompt, the limit, and the start of the run.

**Limits and status:** Available with setup. LinkedIn and X are data sources only: Sendora never sends LinkedIn or X messages, connection requests or comments, and you should not either. Each run is one time: recurring schedules are off, so rerun by hand. A connected LinkedIn account's own connections are switched off. Engagement is interest in a topic, not consent to contact. Run do-not-contact checks (read_leads, action dnc). The pack's signal dates and "asked for an alternative" ranking are done by Claude from the results, not a built-in field.

**Prompt to paste**
```
Use the competitor-engagers skill. In Sendora, make me an empty list called "Competitor engagers test", then prepare (do not start) a linkedin_post_engagers run for these posts: [PASTE POST OR PROFILE URLS]. Use ai_qualification with this ICP: [PASTE ICP]. Limit 50. Show me the config and what it may cost. Only start the run after I say yes. LinkedIn is a data source only, nothing gets sent there.
```

---

## signal-finder

**What you want:** Accounts with a live reason to talk now (hiring, funding, growth), each with a truthful first line.

**Do it in Sendora**
1. Pick the rungs with me: competitor engagement (use competitor-engagers), then hiring, then funding.
2. Hiring signal: find_prospects, action preview_search, mode hiring. Set hiring titles (the role your offer helps), posted_within_days 30 to 45, min_roles_per_company, exclude_agencies true, and description_keywords for a tool or pain named in the posting.
3. Funding signal: preview_search in companies or people mode with company funding_types, last_funding_year_min, lead_investors. Add fit filters (employee_ranges, industries, countries).
4. Growth signal: company web_traffic_min, ad_spend_min or min_linkedin_followers where they fit your offer.
5. Stack signals: run hiring plus funding in one filter set, or compare two previews and mark overlap in the chat.
6. For the best accounts only, ask me before: preview_company_employees with company, style best and a role, then commit only people I pick.
7. For a few top accounts, enrich_leads, action research_web to get a true recent fact for the first line. Draft first lines only from facts seen on the page.

**What you approve:** The filter sets, any commit (find_prospects, action commit_search), and any lookups that cost credits (work email 2, US mobile 8).

**Limits and status:** Available. Funding recency filters work by year, not by exact days, so Claude cannot enforce "within 90 days". Leadership change, tech stack change and general news are not filters: only hiring description_keywords stand in for tech. Do those by hand or via research_web. First lines are drafts. Never claim surveillance or false congratulation.

**Prompt to paste**
```
Use the signal-finder skill. Our offer helps companies that are hiring [ROLE]. In Sendora run preview_search in hiring mode (posted within 30 days, exclude agencies) for [REGION] and [COMPANY SIZE]. Also try a funding variant. Show me samples, no commits. Then draft one honest first line per top account using only facts visible in the posting. Ask before any commit or credit spend.
```

---

## list-quality-check

**What you want:** A keep, fix, drop verdict on a list, a clean list, and a small test batch plan before anyone is contacted.

**Do it in Sendora**
1. Bring the list in safely. For a CSV: check_lead_import, action prepare (validate plus preview in one call). Read each issue and its repair hint. Nothing is written yet.
2. Report to me: rows to insert, update, skip, reject. After my yes: edit_leads, action import into a named list with an idempotency_key. Follow the job (read_platform, action jobs), then check_lead_import, actions verify_import and verify_list.
3. For a Sendora-built list: read_leads, action list with lead_list_id, paginated. Look for role mismatches, missing fields, duplicates.
4. Check suppression: read_leads, action dnc, and statuses unsubscribed, dnc, bounced.
5. Verify emails: enrich_leads, action verify_emails, up to 100 lead_ids per call. It costs 1 credit per conclusive answer. Tell me the count first.
6. Fill gaps only after my yes: enrich_leads, action enrich with fields email or phone_number. Contact lookups cost credits when found.
7. Apply fixes: edit_leads, action bulk with set_status or add_tags (for example tag "test-batch"), edit_leads, action add_to_dnc for opt-outs. Never remove a DNC entry without fresh documented consent.
8. Give me the verdict table and the test batch plan (about 50 rows or 5 percent, mixed clean and uncertain, a stop rule I write first).

**What you approve:** The import, verification spend, enrichment spend, any status or DNC change, and the stop rule.

**Limits and status:** Available. Verification is optional, paid per answer and not a guarantee of delivery. Catch-all and unverified addresses are held back for the test batch only. Text steps only for US mobile numbers with the permission your market requires. Mobile lookup is US only. There is no built-in "country check": Claude compares the country field you provide. You remain responsible for lawful basis and local rules.

**Prompt to paste**
```
Use the list-quality-check skill on this list: [LIST NAME OR PASTE CSV]. In Sendora start read-only: use check_lead_import prepare if it is a CSV, or read_leads list if it is already there, plus the DNC list. Show me the keep, fix, drop table first. Tell me the credit cost before you run verify_emails or any enrichment. Do not change statuses or import until I approve.
```

---

## lead-scoring

**What you want:** An editable score (fit, signal, reachability) and tiers A to D that decide who gets contacted first.

**Do it in Sendora**
1. Agree the rubric in the chat using the pack's starting weights (fit 40, signal 40, reachability 20; Tier A 75 and up, B 55 to 74, C 35 to 54, D lower). These are starting settings, not proven values.
2. Read the leads: read_leads, action list with lead_list_id, then get for a few to see enriched fields and the source signal.
3. Claude computes the score and a one-line why per lead in the chat, with the knobs block (every weight in one place).
4. After my yes, record tiers: edit_leads, action bulk with body.action add_tags and tags such as tier-a, tier-b, tier-c, tier-d, in batches of lead_ids.
5. Optionally save slices: edit_leads, action create_segment with body.name and body.criteria (for example the tag tier-a), then run_segment. Check the segment criteria Claude proposes before saving.
6. Tier D stays out. Tier C goes to enrichment first: enrich_leads, action enrich (credits apply when a contact is found).
7. After the first send, compare reply and positive-reply rates by tier using analytics you read, then adjust the weights.

**What you approve:** The rubric, the tags written to leads, and any enrichment spend.

**Limits and status:** Available with a workaround. Sendora has no built-in score field or scoring engine: the score is computed by Claude from lead data and stored as tags or custom fields. It will not update itself when data changes, so rerun it. Reachability points need verified email, so verify first (see list-quality-check). Pipeline stages (new, contacted, engaged, meeting_booked and so on) track status, not score. Results depend on your list, offer and market.

**Prompt to paste**
```
Use the lead-scoring skill. Read the leads in my Sendora list [LIST NAME]. Use the pack's starting weights and show me the rubric and knobs first. Score each lead with a why column. Do not write anything yet. When I approve, tag leads tier-a, tier-b, tier-c, tier-d in batches via bulk add_tags, and tell me the counts it reports back.
```
