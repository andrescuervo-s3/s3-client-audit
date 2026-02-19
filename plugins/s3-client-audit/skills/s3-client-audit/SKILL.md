---
name: s3-client-audit
description: |
  Performs a comprehensive client communication audit across Slack, Gmail, Google Calendar, Notion meeting notes, and Google Drive. Produces an operational briefing (.md + .docx) analyzing team collaboration health, communication gaps, and project risk for a specific client engagement.

  Use this skill whenever someone asks to: audit a client channel, review client communications, assess team collaboration, check project health, look at how a client project is being managed, investigate communication gaps, or prepare an operational briefing on a client. Also trigger when someone asks "what's happening with [client]?" or "how is [client] going?" in a way that implies they want a systemic view rather than a quick status update. Trigger on phrases like "pull a report on [client]", "how's the [client] channel looking?", "team audit", "communication audit", "collaboration audit", or "channel health check."

  This skill is built for Studio 3 Marketing leadership. It assumes familiarity with S3's channel structure, team roles, and operational cadence.
---

# S3 Client Communication Audit

## What This Skill Does

This skill audits how a client project is being communicated across the organization. It answers one question: "If I could only look at our communication tools, would I understand what's happening on this project?"

The answer is usually no — and the audit shows exactly where and why information is getting lost.

## Why This Matters

At S3, work happens in many places: client Slack channels, functional channels (#artdept, #site-reliability, #subpage-design-review), Gmail threads, Calendar meetings, Notion notes, and Google Drive docs. The client channel is supposed to be the hub, but it often becomes a ghost town while the real work happens elsewhere. This audit maps that reality and surfaces the gaps before they become client-facing problems.

---

## Phase 1: Intake

Before pulling any data, collect these from the user. Most can be inferred from context, but confirm anything ambiguous.

**Required:**
- **Client name** — and spelling variations (e.g., "Anastopoulo" / "Gus Law" / "guslawsc"). Search friction is real — people misspell client names constantly. Collect 2-3 search variants.
- **Client Slack channel** — the #client-name channel

**Usually inferrable — confirm if unclear:**
- **Date range** — contract start to present (check channel creation date as proxy for contract start)
- **Account lead** — who owns this client relationship (usually the channel creator)
- **Client email domain** — if known, dramatically improves Gmail search precision

**Optional but valuable:**
- **Known team members** — designers, devs, content writers assigned to this project
- **Contract scope** — what workstreams are included (website, SEO, photoshoot, paid ads, etc.)
- **Channel purpose field** — S3 channels often contain contract value and scope summary in the purpose/topic field

If the channel purpose field contains contract details (it often does), extract and use those as the scope baseline. This tells you what workstreams *should* be visible in communications.

---

## Phase 2: Data Collection

Pull data from all five sources. Parallelize where possible — these searches are independent. The goal is breadth first: get everything, then analyze.

For the detailed search strategy, tool calls, and edge cases for each source, read `references/data-sources.md`.

### 2.1 Slack — Client Channel (Full History)

Read the full message history of the client channel using `slack_read_channel`. Catalog every message: who posted, when, what type (question, status update, asset share, reaction). Identify all channel members and separate them into active posters vs. silent members. Pull any threads with replies for full context.

Key things to extract:
- Total message count (excluding system join/leave messages)
- Messages per person
- First and last message dates
- Questions asked and whether they got answered (and by whom, and how quickly)
- Gaps of 7+ days between messages

### 2.2 Slack — Cross-Channel Mentions

Search for the client name (and variants) across S3's operational channels using `slack_search_public`. The standard channel list to search:

- **#artdept** — design stand-ups, daily work logs
- **#site-reliability** — dev tickets, technical implementation
- **#subpage-design-review** — design review threads with feedback
- **#ftp** — domain and hosting setup
- **#qwilr-signatures** — contract signing confirmations
- **#seo** — SEO strategy and implementation (if exists)
- **#content** — content writing coordination (if exists)

For each mention found, pull the full thread using `slack_read_thread` if it has replies. Cross-channel threads often contain the most substantive project collaboration — design reviews, dev ticket discussions, content handoffs.

Also identify people who are active on this project in other channels but NOT in the client channel. These are the "invisible contributors" — their work exists but isn't surfaced.

### 2.3 Gmail

Search for emails using `search_gmail_messages` with multiple queries:
- The client name and common variants
- The client's email domain (if known) to find direct correspondence
- The account lead's name + client name to find internal discussions

For each relevant email thread found, read it with `read_gmail_thread` to understand: who's involved, what's being discussed, whether it's client-facing or internal, and whether the substance ever appeared in Slack.

Key things to extract:
- How often is the client being contacted directly?
- Who is emailing the client? Is it just one person?
- Are internal team members CC'd on client emails, or is communication siloed?
- Are there email threads that should have triggered a Slack update but didn't?

### 2.4 Google Calendar

Search for calendar events using `list_gcal_events` with the client name as a query. Also try variant spellings.

Key things to extract:
- Meeting frequency — weekly, biweekly, ad hoc?
- Who attends client meetings? Is it always the same person?
- Are there internal team meetings about this client?
- Is there a standing meeting cadence, or just reactive scheduling?
- For each meeting, check: did a summary or follow-up appear in Slack, Notion, or email within 48 hours?

### 2.5 Notion Meeting Notes

Query meeting notes using `notion-query-meeting-notes` filtered by:
- Title containing the client name
- Attendees including the account lead
- Date range matching the project timeline

If the search by title returns nothing, broaden to search by attendee (account lead) within the date range, then scan for client mentions in the results.

Key things to extract:
- Are meetings being documented?
- Are action items captured and attributed to specific people?
- Do those action items show up as completed work elsewhere?
- Are decisions being recorded explicitly?

### 2.6 Google Drive

Search for documents using `google_drive_search` with the client name. Map the full document ecosystem:
- What documents exist? (wireframes, creative briefs, content pages, website notes, etc.)
- Who created each document and when?
- Who last edited each document and when?
- What's the modification velocity? (Are docs actively being worked on?)

Build a document inventory table. This often reveals the scope of work that's invisible in Slack — content writers, editors, and designers doing significant work that never gets mentioned in the client channel.

---

## Phase 3: Cross-Source Analysis

This is where the audit earns its keep. Raw data from each source is useful, but the real value comes from cross-referencing across sources to find the gaps between them.

Read `references/cross-source-analysis.md` for the detailed framework and examples.

### Meeting → Follow-up Check
For every client meeting on the calendar, check whether a summary appeared in Slack, Notion, or email within 48 hours. Meetings without follow-up are where decisions get lost. If there are Notion meeting notes, check whether the action items from those notes were communicated to the team.

### Email → Team Relay Check
For every email thread with the client, check whether the substance was relayed to the team in Slack or elsewhere. If the account lead is having conversations with the client that the team never sees, that's a bus-factor risk — if that person is unavailable, institutional knowledge disappears.

### Slack Silence → Activity Check
For every gap of 7+ days in the client channel, check whether Gmail, Calendar, Drive, or other Slack channels show active work during that period. Long channel silences often don't mean the project is stalled — they mean the work is happening in places the channel can't see. Quantify this: "During the 63-day channel silence, we found 5 dev tickets, 8 content pages created, and 2 design review threads."

### Silent Member → Contribution Check
For every channel member who has never posted in the client channel, check whether they appear in: email threads about this client, calendar events, Google Drive documents, or other Slack channel mentions. Some "silent" members are actually the most active contributors — they just work in their functional channels rather than the client channel.

### Decision Traceability
Pick 2-3 major project decisions (design approvals, scope changes, launch dates, content direction) and trace them from origin to execution:
- Where was the decision made? (meeting, email, Slack message?)
- Who was informed?
- Did it reach the people who needed to act on it?
- How long did it take to flow from decision to action?

---

## Phase 4: Analysis Framework

The purpose of this audit is to answer one question for S3 leadership: **How well is this team executing this project, and where is it falling apart?**

This is not a communication report. It's a project health diagnosis. Every section must tell a story with names, dates, dollar amounts, and consequences. If a finding doesn't connect to project risk, timeline impact, or client exposure, it's background context — not a headline.

Structure the briefing around these nine sections.

### 1. Communication Volume & Distribution
Tell the story of who is doing the talking and who isn't. Connect volume to responsibility: if the account lead posted 80% of messages, that's a bus-factor problem. If 5 of 12 channel members have never posted, name them, check if they're contributing elsewhere, and say whether the project is being run by a skeleton crew.

Always tie volume to contract value. A $126K engagement with 3 messages per month is a fundamentally different risk profile than a $5K engagement with the same cadence. State the ratio explicitly.

### 2. Information Flow — Where the Work Actually Happens
This is usually the most important finding. Map the gap between where work is happening and where it's visible. Name every person doing work on this project in other channels who is not posting in the client channel. Name the channels where the real collaboration is happening.

Build the picture concretely: "Rose completed 4 design rounds in #subpage-design-review. Walt closed 5 dev tickets in #site-reliability. Rohan wrote 8 content pages tracked in Google Drive. None of this work was ever mentioned in the client channel. If the account lead were hit by a bus, the replacement would see an empty channel and have no idea the project was 70% complete."

### 3. Responsiveness
Find every question asked in the client channel and track what happened. Who answered? How long did it take? Did it get answered at all? If one person answers every question, name them and call it the single point of failure it is.

Look for the worst cases — questions that sat unanswered for days, questions that got answered in email but not Slack, questions from the client that the team never saw. These are the moments where project trust erodes.

### 4. Cadence & Gaps
Map the project timeline with specific dates. Identify every gap of 7+ days and tell the story of what was actually happening during each silence. This is where the cross-source analysis pays off.

Example of what this should look like: "The client channel went silent from December 18 to February 11 — 55 days. During that period, Rose posted 4 design updates in #subpage-design-review, Rohan created 8 content documents in Google Drive, Walt worked 5 dev tickets in #site-reliability, and Whitney had 3 email exchanges with the client. The project was active. The channel just couldn't see it."

### 5. Cross-Functional Visibility
Build a workstream-by-workstream table: for each scope item in the contract (website, SEO, content, paid ads, photoshoot, etc.), show what's visible from inside the client channel vs. what's happening elsewhere. Be specific about what's missing.

This answers: "If I'm a new team member added to this channel, could I understand where this project stands?" The answer should include exactly what's invisible and where you'd have to go to find it.

### 6. Timing & Sequencing Risks
**This is where you find the red flags.** For every specialized role (SEO, dev, content, design), find when they first appeared in any communication about this project and compare that to when they should have been involved.

Tell the story the way a project post-mortem would: "The contract includes $10,550 in SEO services. Justin Reese, the SEO specialist, joined the client channel on February 19 — the same day the team was told launch is Monday February 23. That's 2 business days for on-page SEO, meta data, schema markup, redirect mapping, and indexing strategy on a new-build website in a competitive legal market. This work typically needs to be integrated during development, not bolted on at the end."

Also flag: people who joined the channel with no message (what are they doing there?), handoff timing gaps (content finished but dev didn't start for 3 weeks), and scope items with no communication trail at all.

### 7. Client-Facing Communication Health
*(Draws primarily from Gmail and Calendar data.)*
Answer: Is S3 proactively communicating with the client, or only reaching out when they need something? How often are meetings happening? Is one person handling all client contact (bus-factor risk)?

Find the gaps: weeks where the client heard nothing, emails where the client asked a question and the response took 5+ days, meetings with no follow-up summary sent. These are the moments that erode client confidence.

### 8. Decision Traceability
Pick 2-3 major project decisions and trace them from origin to execution. Show the full chain: where the decision was made (meeting? email? Slack?), who was informed, whether it reached the people who needed to act on it, and how long it took to flow from decision to deliverable.

The point is to show where decisions get lost. Example: "In the January 8 client call, the client approved the homepage wireframe. The approval was noted in Notion meeting notes but never posted to the client channel or communicated to the dev team. Walt didn't start homepage development until January 29 — 21 days after approval — because no one told him the wireframe was approved."

### 9. Anomalies & Red Flags
Flag anything that doesn't fit the expected pattern. This includes: team members who should be in the channel but aren't, scope items with zero communication trail (paid for but no evidence of work), people added to the channel on launch week with no explanation, naming convention issues that create search friction, and documents being used as de facto project management tools because Slack coordination failed.

**Rank anomalies by severity.** A missing SEO workstream on a $10K SEO contract is a red flag. A misspelled channel name is a footnote.

---

## Phase 5: Output

**Always produce both outputs.**

### Markdown Briefing
Save as `[Client-Name]-Client-Audit.md` in the workspace folder. This is the working document — easy to scan, copy, and share in Slack.

### Word Document (.docx)

Read `references/docx-template.md` for the exact document structure, styling code, and template. The .docx should be a polished, professional document suitable for sharing in a leadership meeting.

Key formatting elements:
- **Cover page** with client details table (channel, contract value, date range, project status)
- **Headers/footers** with client name and page numbers
- **Data tables** with alternating row shading and dark header rows
- **Red accent color (#C0392B)** for key findings and critical callouts
- **Callout box** for the summary assessment
- **Numbered action items** in the conclusion

Save as `[Client-Name]-Client-Audit.docx` in the workspace folder.

---

## Tone and Style

This is an operational briefing, not a consultant's report. Write like you're briefing a CEO who has 10 minutes and needs to know what's actually happening — not what should be happening.

**The standard:** Every finding must include a specific person, a specific date, a specific number, and a specific consequence. If you can't name who, when, and what it means for the project, the finding isn't ready.

**Patterns that work:**
- "Whitney is the single point of failure for institutional knowledge in this channel. She posted 23 of 30 messages (77%). If she's unavailable for a week, no one else in this channel has enough context to answer the client's questions."
- "The contract includes $7,550 in SEO services plus $3,000 in Interim SEO — $10,550 total. Justin Reese joined the channel on February 19, with launch planned for February 23. He has 2 business days to complete work that should have been running alongside development for months."
- "The entire design review cycle — 11 messages over 7 days between Rose, Sydney, and two rounds of client feedback — happened completely outside the client channel in #subpage-design-review. Anyone looking at the client channel would have no idea the homepage design was finalized."
- "During the 55-day channel silence (Dec 18 – Feb 11), Rose was designing the landing page, Rohan was writing 8 content pages, and Walt was executing 5 dev tickets. The project was 70% complete. The client channel showed nothing."

**Patterns that are unacceptable — never use these:**
- "Communication could be improved across the team."
- "It is recommended that stakeholders consider enhancing visibility."
- "There may be opportunities for better cross-functional alignment."
- "The team could benefit from more regular check-ins."
- Any sentence that could apply to any project at any company. Every sentence must be specific to THIS client, THIS team, THIS project.

**Structure each section as a narrative, not a list of bullet points.** Tell the story of what happened, then state what it means. Lead with the most important finding, not the chronological first event.

End with exactly three actionable items — the three things that need attention right now, ranked by urgency. Each action item must name who should do what by when.

---

## Data Limitations — Always Acknowledge

This audit can only see what the person running it has access to. Be explicit about blind spots:

- **Gmail visibility is partial.** The audit only searches the inbox of the person running it. If the account lead had email exchanges with the client that this person wasn't CC'd on, those conversations are invisible. State this clearly: "This audit reflects email threads visible to [person's name]. Client-facing emails where [person] was not CC'd are not captured. The actual volume of client communication may be higher."
- **Slack private channels and DMs are not searchable.** If project conversations happened in private channels or direct messages, they won't appear. Note this.
- **Google Drive access depends on sharing permissions.** Documents the auditor doesn't have access to won't show up in search results.
- **Calendar visibility depends on sharing settings.** If the account lead's calendar isn't shared, client meetings may not appear.

**Do not present findings as complete truth.** Present them as "what we can see from the tools we have access to." Where the data suggests something is missing (e.g., a scope item with zero communication trail), flag it as a gap worth investigating — not a definitive conclusion that no work was done.

The goal is to surface what's visible and call out what isn't. Both are valuable.
