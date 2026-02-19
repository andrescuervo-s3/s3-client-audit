# S3 Client Communication Audit

A Claude Cowork plugin that performs comprehensive client communication audits across five data sources, producing operational briefings that reveal where information is getting lost in your organization.

## What It Does

This plugin audits how a client project is being communicated across Studio 3 Marketing. It answers one question: **"If I could only look at our communication tools, would I understand what's happening on this project?"**

The answer is usually no — and the audit shows exactly where and why.

## Data Sources

The audit pulls from five sources and cross-references them against each other:

- **Slack** — Client channel full history + cross-channel mentions (#artdept, #site-reliability, #subpage-design-review, #ftp, #qwilr-signatures, #seo, #content)
- **Gmail** — Client-facing correspondence and internal discussions
- **Google Calendar** — Meeting frequency, attendees, and follow-up gaps
- **Notion** — Meeting notes, decisions, and action item tracking
- **Google Drive** — Document inventory, contributor mapping, and activity timelines

## Analysis Framework

The briefing covers nine dimensions:

1. Communication Volume & Distribution
2. Information Flow — Where Work Actually Happens
3. Responsiveness
4. Cadence & Gaps
5. Cross-Functional Visibility
6. Timing & Sequencing Risks
7. Client-Facing Communication Health
8. Decision Traceability
9. Anomalies

Plus five cross-source analyses: Meeting → Follow-up, Email → Team Relay, Slack Silence → Activity, Silent Member → Contribution, and Decision Traceability.

## Output

Every audit produces two files:

- **Markdown briefing** (`[Client]-Client-Audit.md`) — working document for quick scanning and Slack sharing
- **Word document** (`[Client]-Client-Audit.docx`) — polished, formatted briefing for leadership meetings

## How to Trigger

Say any of the following:

- "Audit the [client] channel"
- "Pull a report on [client]"
- "How's the [client] channel looking?"
- "Run a communication audit on [client]"
- "Check project health for [client]"

## Required Connectors

This plugin requires the following MCP connectors to be enabled:

| Connector | Used For |
|-----------|----------|
| Slack | Channel history, cross-channel search, member profiles |
| Gmail | Client correspondence, internal email threads |
| Google Calendar | Meeting events, attendee lists |
| Notion | Meeting notes, action items |
| Google Drive | Document inventory, contributor mapping |

## Built For

Studio 3 Marketing leadership. Assumes familiarity with S3's channel structure, team roles, and operational cadence.
