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

Structure the briefing around these nine sections. Each section should include specific examples from the data — not vague observations.

### 1. Communication Volume & Distribution
Who is posting the most? Who is silent? What percentage of channel members have never posted? How does this channel's activity level compare to its contract value and complexity? A $126K engagement with 3 messages per month is a very different signal than a $5K engagement with the same cadence.

### 2. Information Flow — Where the Work Actually Happens
Is project work visible in the client channel, or is it happening elsewhere? Map where the actual work conversations are taking place vs. what's being surfaced. This is usually the most important finding. Build a clear picture: "9 active contributors are doing work on this project in other channels, none of whom are in the client channel."

### 3. Responsiveness
Are questions getting answered? By whom? How quickly? Is there a single point of failure — one person who answers everything? What happens when that person is unavailable?

### 4. Cadence & Gaps
How often is the channel active? Map the timeline: when did activity happen, and how long were the gaps between bursts? Overlay the gaps against activity found in other sources. Is there any regular status update rhythm (weekly check-ins, milestone announcements)?

### 5. Cross-Functional Visibility
Can someone in this channel understand the full project status (design, dev, content, SEO, account) without going elsewhere? Build a workstream-by-workstream visibility table showing what's visible vs. invisible from inside the client channel.

### 6. Timing & Sequencing Risks
Are people or workstreams being brought in too late? Check when specialized roles (SEO, dev, content) first appeared in client communications vs. when they should have been involved given the project timeline. Any signs of last-minute scrambling?

### 7. Client-Facing Communication Health
*(This section draws primarily from Gmail and Calendar data.)*
How often is the client hearing from S3? Is there a regular cadence of client emails or calls? Is one person handling all client communication, or is it distributed? Are client questions getting answered promptly? Is the client being kept informed of progress, or only contacted when S3 needs something?

### 8. Decision Traceability
*(This section draws from cross-source analysis.)*
Can you follow a decision from the meeting where it was made, through team communication, to the deliverable? Show 2-3 traced examples. Where in the chain do decisions get lost?

### 9. Anomalies
Anything unusual in naming conventions, channel structure, missing workstreams, display names, search friction, or activity patterns. Also flag: team members who should be in the channel but aren't, scope items with no communication trail, and documents that seem to be doing the job of Slack (massive Google Docs used as de facto project management tools).

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

This is an operational briefing, not a formal report. Write in direct language with specific examples and clear conclusions. The reader is a senior leader who wants to know what's actually happening, not what should be happening.

**Patterns that work:**
- "Whitney is the single point of failure for institutional knowledge in this channel."
- "5 of 12 original members (42%) have never posted a single message."
- "The entire design review cycle — 11 messages over 7 days — happened completely outside the client channel."
- "During the 63-day channel silence, Rose was designing the landing page, Rohan was writing 8 content pages, and Walt was executing 5 dev tickets. None of this appeared in the client channel."

**Patterns to avoid:**
- "Communication could be improved across the team."
- "It is recommended that stakeholders consider enhancing visibility."
- "There may be opportunities for better cross-functional alignment."
- Generic observations without specific evidence.

Every claim should be backed by a specific message, date, person, or document. If you can't point to evidence, don't make the claim. Quantify everything you can: message counts, gap durations, response times, document counts.

End with exactly three actionable items — the three things that need attention right now, ranked by urgency.
