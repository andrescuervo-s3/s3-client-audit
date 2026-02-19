# Data Source Search Strategies

Detailed instructions for pulling data from each source during a client audit.

## Table of Contents
1. [Slack — Client Channel](#1-slack--client-channel)
2. [Slack — Cross-Channel Mentions](#2-slack--cross-channel-mentions)
3. [Gmail](#3-gmail)
4. [Google Calendar](#4-google-calendar)
5. [Notion Meeting Notes](#5-notion-meeting-notes)
6. [Google Drive](#6-google-drive)

---

## 1. Slack — Client Channel

### Finding the Channel
Use `slack_search_channels` with the client name. S3 client channels follow the pattern `#client-name` or `#client-name-project`. The channel purpose/topic field often contains contract details — extract these.

### Reading Full History
Use `slack_read_channel` with the channel ID. Read in batches if needed (use cursor pagination). Continue until you've read every message.

### What to Catalog
For each message, record:
- **Sender** (name and user ID)
- **Timestamp** (for gap analysis and response time calculation)
- **Message type**: categorize as question, status update, asset/link share, reaction/acknowledgment, logistics/scheduling, or system message (join/leave)
- **Thread activity**: if the message has thread replies, note the count and pull with `slack_read_thread`

### Derived Metrics
From the raw message data, compute:
- **Messages per person** — who's talking and who's silent
- **Silent member list** — channel members with zero messages (these are your "ghosts")
- **Silent member percentage** — e.g., "5 of 12 (42%) have never posted"
- **Gap map** — every period of 7+ days with no messages, with start/end dates
- **Question-response pairs** — questions asked, who answered, time to response
- **Activity clusters** — when does the channel come alive? What triggers it?

### Edge Cases
- **Bot messages**: Exclude from message counts but note if integrations (Asana, Jira, etc.) are posting. This tells you whether project management tools are connected.
- **Reactions only**: Some members only react (emoji) but never post. Count these separately from truly silent members.
- **Renamed users**: Slack display names can be confusing ("Rage" = Rachel Howell, "Mommy-Shark" = Lynsey Moore). Cross-reference with `slack_read_user_profile` if a display name is unclear.

---

## 2. Slack — Cross-Channel Mentions

### Search Strategy
Use `slack_search_public` with the client name. Run multiple searches with name variants:
- Full client name: `"Gus Anastopoulo"`
- Short name: `"Anastopoulo"`
- Domain/brand name: `"guslawsc"`
- Common abbreviations the team might use

### Channels to Search
Target these S3 operational channels specifically (use `in:channel-name` modifier):

| Channel | What You'll Find |
|---------|-----------------|
| #artdept | Designer daily stand-ups mentioning client work |
| #site-reliability | Dev tickets assigned for this client |
| #subpage-design-review | Design review threads with feedback rounds |
| #ftp | Domain and hosting setup |
| #qwilr-signatures | Contract signing confirmations |
| #seo | SEO strategy discussions |
| #content | Content planning and handoffs |
| #dev | Development coordination |

Also run a broad search without channel filters to catch mentions in unexpected places.

### Thread Depth
For every cross-channel mention that has thread replies, pull the full thread with `slack_read_thread`. These threads are often the richest source of project collaboration — design feedback, dev discussions, content reviews — and they happen entirely outside the client channel.

### Building the Invisible Contributor Map
Track every person who appears in cross-channel mentions but is NOT a member of the client channel. Build a table:

| Person | Channel(s) Active In | Work Done | In Client Channel? |
|--------|---------------------|-----------|-------------------|
| Walt Gottlieb | #site-reliability | 5 dev tickets | No |
| Jeff Lu | #subpage-design-review | Full subpage design | No |

This table becomes one of the most impactful parts of the briefing.

---

## 3. Gmail

### Search Queries
Run multiple searches using `search_gmail_messages`:

**Client-facing correspondence:**
```
{client name}
```
```
from:{client-domain.com}
```
```
to:{client-domain.com}
```

**Internal discussions about client:**
```
{client name} from:{account-lead@studio3marketing.com}
```
```
{client name} subject:re:
```

### Reading Threads
For each relevant result, use `read_gmail_thread` to get the full conversation. Focus on:
- **Participants** — who's on the thread? Internal only, or client-facing?
- **Content** — what decisions or information were exchanged?
- **Relay check** — did this information appear anywhere in Slack afterward?

### Key Analysis Points
- **Client contact frequency** — how many emails to/from client per week/month?
- **Response times** — how quickly does S3 respond to client emails?
- **Communication breadth** — is it always one person emailing the client, or multiple team members?
- **Silo detection** — are there email threads where the account lead discusses decisions with the client that the team never sees in Slack?

### Privacy Note
Gmail search results may contain sensitive client information. The audit should reference the existence and pattern of emails without quoting client communications verbatim. Focus on metadata (who, when, how often) rather than content.

---

## 4. Google Calendar

### Search Strategy
Use `list_gcal_events` with:
- `query`: client name (try variants)
- `time_min`: contract start date (channel creation as proxy)
- `time_max`: current date

### What to Extract
For each event found:
- **Title** — what kind of meeting? (kick-off, check-in, review, etc.)
- **Date and time**
- **Attendees** — who from S3? Who from the client?
- **Recurrence** — is this a standing meeting or one-off?
- **Description** — does it contain an agenda or notes link?

### Derived Analysis
- **Meeting cadence** — how often are client meetings happening?
- **Attendee consistency** — same person every time, or rotating?
- **Internal vs. external** — are there internal-only meetings about this client?
- **Follow-up audit** — for each meeting, check: did a summary, action items, or any acknowledgment appear in Slack, Notion, or email within 48 hours?

### The Follow-Up Gap
The most valuable finding from calendar analysis is usually the follow-up gap. A meeting happens → no one posts anything → the team doesn't know what was discussed → work continues based on outdated information. Map these gaps explicitly.

---

## 5. Notion Meeting Notes

### Search Strategy
Use `notion-query-meeting-notes` with filters:

**Primary search — by title:**
```json
{
  "filter": {
    "operator": "and",
    "filters": [
      {
        "property": "title",
        "filter": {
          "operator": "string_contains",
          "value": { "type": "exact", "value": "client name" }
        }
      }
    ]
  }
}
```

**Fallback — by attendee + date range:**
If title search returns nothing, search by the account lead as attendee within the project date range. Then scan results for client mentions.

**Broadest fallback — by date range only:**
Search the account lead's meetings within the date range and manually check for relevance.

### What to Extract
For each meeting note found, use `notion-fetch` to read the full content:
- **Meeting title and date**
- **Attendees**
- **Decisions made** — look for explicit decision language
- **Action items** — who's responsible, what's the deliverable, when?
- **Follow-up status** — did the action items get done? (Cross-reference against Slack, Drive, and dev tickets)

### Decision Trail Analysis
Meeting notes are where decisions often originate. The audit should trace 2-3 key decisions from the meeting note → team communication → completed deliverable. Where does the trail go cold?

---

## 6. Google Drive

### Search Strategy
Use `google_drive_search` with:
```
fullText contains '{client name}'
```
And:
```
name contains '{client name}'
```

Run both queries — some documents mention the client in the body but not the title, and vice versa.

### Building the Document Inventory
For each document found, record:
- **Title**
- **Type** (wireframe, creative brief, content page, website notes, etc.)
- **Owner** (who created it)
- **Created date**
- **Last modified date and by whom**
- **Size** (unusually large docs may be de facto project management tools)

### Document Activity Timeline
Plot document creation and modification dates on a timeline alongside Slack activity. This often reveals: "The client channel was silent for 63 days, but during that period, 8 content pages were created and the creative brief was finalized." This is the evidence that work was happening — just not visibly.

### The "Shadow PM" Document
Look for one unusually large document (often titled "Website Notes" or similar) that has been modified frequently by the account lead. At S3, these massive Google Docs often serve as the de facto project management tool — containing decisions, specifications, and status updates that should be in Slack but aren't. If found, flag this in the Anomalies section.

### Content Contributor Mapping
Track everyone who created or edited documents. Cross-reference against the client channel member list. Content writers, editors, and SEO specialists often do significant work in Drive without ever appearing in the client channel.
