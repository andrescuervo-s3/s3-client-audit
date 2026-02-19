# Cross-Source Analysis Framework

This document defines the cross-referencing logic that turns raw data from five sources into actionable findings. Each cross-reference follows the same pattern: take a data point from one source, look for its echo in another source, and note whether the echo exists.

The absence of an echo is the finding.

---

## 1. Meeting → Follow-Up Check

**Source:** Google Calendar events
**Echo expected in:** Slack (client channel), Notion (meeting notes), Gmail (summary email)

### Procedure
For each client meeting found in Google Calendar:
1. Note the meeting date, time, and attendees
2. Search the client Slack channel for messages within 48 hours after the meeting
3. Search Notion for meeting notes matching the date and/or attendees
4. Search Gmail for any post-meeting summary or follow-up thread

### Scoring
- **Strong follow-up:** Meeting notes in Notion AND summary posted to Slack or emailed to team
- **Partial follow-up:** Notes exist in one place but not communicated to team
- **No follow-up:** Meeting happened but no record of outcomes found anywhere

### What This Reveals
Meetings without follow-up are where decisions go to die. The team continues working based on outdated information because they don't know what was discussed. If the pattern is consistent (meetings happen but follow-up is rare), it's a systemic issue — not a one-off.

### Example Finding
"5 client meetings occurred between October and January. Notion meeting notes exist for 2 of them. Post-meeting summaries appeared in the client Slack channel for 0 of them. The team has no visibility into what was discussed in 3 of 5 client meetings."

---

## 2. Email → Team Relay Check

**Source:** Gmail threads involving the client
**Echo expected in:** Slack (client channel or relevant functional channel)

### Procedure
For each substantive email thread with the client:
1. Identify the key information or decisions in the thread
2. Search the client Slack channel for messages that relay this information
3. Search other relevant Slack channels for the information
4. Check if the email was forwarded to team members or if they were CC'd

### What Counts as "Substantive"
Not every email needs to be relayed. Focus on:
- Client requests or change orders
- Client feedback on deliverables
- Timeline changes or deadline discussions
- Scope clarifications
- Client dissatisfaction signals

### Scoring
- **Relayed:** Key information appeared in Slack or was forwarded to relevant team members
- **Partially relayed:** Some information made it through but critical details were lost
- **Not relayed:** Team has no visibility into the email exchange

### What This Reveals
If one person is having client conversations that the team never sees, that person becomes a single point of failure. Their absence (vacation, sick day, departure) creates an instant knowledge gap. The audit should quantify this: "Of 12 substantive email threads with the client, 3 were relayed to the team in Slack."

---

## 3. Slack Silence → Activity Check

**Source:** Gaps of 7+ days in the client Slack channel
**Echo expected in:** Cross-channel Slack mentions, Gmail, Calendar, Google Drive

### Procedure
For each gap of 7+ days in the client channel:
1. Define the gap period (start date, end date, duration)
2. Check cross-channel Slack mentions during this period
3. Check Gmail for client-related correspondence during this period
4. Check Google Calendar for meetings during this period
5. Check Google Drive for document creation/modification during this period

### Output Format
For each gap, produce a summary like:

**Gap: Oct 28 – Dec 30 (63 days)**
- Cross-channel Slack: Rose worked on landing page design (Nov 14-17, #artdept)
- Gmail: 3 email threads with client about content direction
- Calendar: 2 client check-in calls (Nov 5, Dec 3)
- Google Drive: Creative brief finalized, 8 content pages drafted
- **Verdict:** Significant work was happening. The channel was dead but the project was not.

### What This Reveals
The gap analysis often produces the most dramatic finding: a channel that looks abandoned is actually backed by significant work happening everywhere except the one place designed to aggregate it. This is the core dysfunction the audit is designed to expose.

---

## 4. Silent Member → Contribution Check

**Source:** Channel members with zero messages
**Echo expected in:** Gmail, Calendar, Drive, cross-channel Slack

### Procedure
For each silent channel member:
1. Search Gmail for their name + client name
2. Check Calendar for their attendance at client meetings
3. Search Google Drive for documents they created or edited for this client
4. Search cross-channel Slack for their name + client name

### Classification
- **Truly silent:** No trace of contribution in any source. May not have a role on this project.
- **Active elsewhere:** Contributing through other channels, email, or Drive but not the client channel. The channel gives a false impression of their involvement.
- **Meeting-only:** Shows up in calendar events but has no other communication trail.

### What This Reveals
The "42% of channel members never posted" finding is powerful on its own. But it becomes more nuanced when you discover that some "silent" members are actually writing content, attending meetings, or working dev tickets — just not in the channel. The audit should distinguish between genuinely uninvolved members and members who are active but invisible.

---

## 5. Decision Traceability

**Source:** Pick 2-3 major project decisions
**Trace through:** All sources

### Selecting Decisions to Trace
Choose decisions that are:
- **Consequential** — affected the project trajectory (design direction, launch date, scope change)
- **Identifiable** — you can find the origin point (a meeting, an email, a Slack message)
- **Recent enough** — still relevant to current project state

Common decision types at S3:
- Homepage design approval
- Subpage design direction
- Content strategy decisions
- Launch date setting
- SEO approach
- Photography/creative direction

### Tracing Procedure
For each decision:
1. **Find the origin.** Where was this decision first articulated? (Meeting note? Email? Slack message?)
2. **Track the relay.** How did the decision flow to the people who needed to act on it? (Posted in Slack? Emailed? Mentioned in a meeting? Added to a doc?)
3. **Verify execution.** Did the people who needed to know actually know? Did they act on it? (Check Drive documents, dev tickets, design files)
4. **Measure the lag.** How long between decision and action? Hours? Days? Weeks?

### Output Format
For each traced decision:

**Decision: Homepage design approved**
- **Origin:** Client email to Whitney (Jan 14)
- **Relay:** Whitney posted in client channel: "homepage was approved last week!" (Jan 19, 5 days later)
- **Execution:** Jeff began subpage design the same week (Jan 30, #subpage-design-review)
- **Gap:** The team had no formal notification for 5 days. The approval was communicated as a casual aside, not a milestone.

### What This Reveals
Decision traceability shows whether the organization has a reliable way to move from "we decided X" to "X is done." If decisions routinely take days to reach the executors, or if they arrive as casual mentions rather than clear directives, the team is operating on delayed information. This compounds over time.

---

## Cross-Reference Summary Table

After completing all five cross-references, build a summary table for the briefing:

| Cross-Reference | Checked | Gaps Found | Severity |
|----------------|---------|------------|----------|
| Meeting → Follow-up | 5 meetings | 3 with no follow-up | High |
| Email → Team relay | 12 threads | 9 not relayed | Critical |
| Slack silence → Activity | 3 gaps (114 days total) | All gaps had active work | High |
| Silent members → Activity | 5 silent members | 2 active elsewhere | Medium |
| Decision traceability | 3 decisions traced | 2 with relay gaps | High |

This table goes in the Summary Assessment section of the briefing, right before the three action items.
