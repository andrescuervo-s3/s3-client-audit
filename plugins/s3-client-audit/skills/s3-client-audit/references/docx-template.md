# .docx Output Template

This document defines the exact structure and styling for the Word document output. Read the docx skill (`/mnt/.skills/skills/docx/SKILL.md`) for the full docx-js API reference before generating the document.

## Design System

### Colors
| Name | Hex | Usage |
|------|-----|-------|
| DARK | #1A1A2E | Headings, bold text |
| ACCENT | #C0392B | Key findings, callout borders, critical data |
| GRAY | #555555 | Body text |
| LIGHT_BG | #F5F5F5 | Meta table label cells |
| TABLE_HEADER_BG | #2C3E50 | Table header rows (white text) |
| TABLE_ALT_BG | #F9F9F9 | Alternating table rows |
| BORDER_COLOR | #CCCCCC | Table cell borders |
| CALLOUT_BG | #FDF2F2 | Summary callout box background |

### Typography
- **Font:** Arial throughout (universally supported)
- **Body text:** 10pt (size: 20 in docx-js half-points)
- **H1:** 16pt bold
- **H2:** 13pt bold
- **H3:** 11pt bold
- **Header/footer:** 8pt italic gray

### Page Setup
- US Letter (12240 x 15840 DXA)
- 1-inch margins all sides (1440 DXA)
- Content width: 9360 DXA

---

## Document Structure

### Section 1: Cover Page
Top margin set to 2 inches (2880 DXA) for visual breathing room.

Contents in order:
1. **"OPERATIONAL BRIEFING"** — small caps feel, ACCENT color, letter-spacing 120, 11pt bold
2. **Accent divider line** — bottom border on empty paragraph, ACCENT color, 6pt weight
3. **Client name** — 26pt bold, DARK
4. **"Channel Collaboration Audit"** — 16pt, GRAY
5. **Accent divider line**
6. **Meta table** — two columns (label + value), label cells shaded LIGHT_BG:
   - Channel: #client-name
   - Contract Value: $X (scope breakdown)
   - Channel Created: date
   - Date of Audit: date
   - Project Status: current status in ACCENT bold
7. **Data sources footnote** — 9pt italic GRAY at bottom

### Section 2: Main Content
Standard 1-inch margins. Header (right-aligned, italic): "Client Name — Channel Audit". Footer (centered): "Page [N]".

Each of the nine analysis sections follows this pattern:
1. **Section heading** (H1)
2. **Accent divider line** (same style as cover)
3. **Key finding callout** — opening paragraph with the most important finding in ACCENT bold, followed by supporting context in GRAY
4. **Data table(s)** — with header rows and alternating shading
5. **Analysis text** — direct, specific, evidence-backed paragraphs
6. **Sub-sections** (H2/H3) as needed

### Summary Assessment Section (Final)
1. **"Summary Assessment"** heading (H1)
2. **Accent divider line**
3. **Callout box** — built as a single-cell table:
   - Left border: 12pt ACCENT
   - Other borders: 1pt ACCENT
   - Background: CALLOUT_BG (#FDF2F2)
   - Generous padding (200 DXA vertical, 300 DXA horizontal)
   - Contains the one-line summary in 12pt ACCENT bold + supporting paragraph in GRAY
4. **"Three Things That Need Attention Right Now"** — 13pt bold DARK
5. **Accent divider line**
6. **Numbered list** — each item has a bold ACCENT lead phrase + GRAY explanation

---

## Table Styling Rules

All tables in the document follow these rules (critical for consistent rendering):

1. **Always use WidthType.DXA** — never percentages (breaks in Google Docs)
2. **Set both columnWidths AND cell width** — they must match
3. **Table width = sum of columnWidths** — always equals content width (9360) for full-width tables
4. **Header rows:** TABLE_HEADER_BG background, white bold text
5. **Alternating rows:** Every other data row gets TABLE_ALT_BG shading
6. **Cell margins:** `{ top: 80, bottom: 80, left: 120, right: 120 }` on every cell
7. **Cell borders:** 1pt BORDER_COLOR on all sides
8. **Use ShadingType.CLEAR** — never SOLID (SOLID creates black backgrounds)

---

## Accent Divider Pattern

Used after every H1 heading and on the cover page:

```javascript
new Paragraph({
  spacing: { before: 200, after: 200 },
  border: { bottom: { style: BorderStyle.SINGLE, size: 6, color: "C0392B", space: 1 } },
  children: []
})
```

---

## Bullet and Numbered List Setup

Always configure numbering in the Document constructor — never use unicode bullet characters.

```javascript
numbering: {
  config: [
    {
      reference: "bullets",
      levels: [{
        level: 0, format: LevelFormat.BULLET, text: "\u2022",
        alignment: AlignmentType.LEFT,
        style: { paragraph: { indent: { left: 720, hanging: 360 } } }
      }]
    },
    {
      reference: "numbers",
      levels: [{
        level: 0, format: LevelFormat.DECIMAL, text: "%1.",
        alignment: AlignmentType.LEFT,
        style: { paragraph: { indent: { left: 720, hanging: 360 } } }
      }]
    }
  ]
}
```

---

## Validation

After generating the .docx, validate it:

```bash
python /mnt/.skills/skills/docx/scripts/office/validate.py output.docx
```

If validation fails, unpack, fix, and repack per the docx skill instructions.

---

## Cross-Reference Summary Table

The final briefing should include this table in the Summary Assessment section, populated with actual findings:

| Cross-Reference | Checked | Gaps Found | Severity |
|----------------|---------|------------|----------|
| Meeting → Follow-up | N meetings | N with no follow-up | High/Med/Low |
| Email → Team relay | N threads | N not relayed | High/Med/Low |
| Slack silence → Activity | N gaps (N days total) | Description | High/Med/Low |
| Silent members → Activity | N silent members | N active elsewhere | High/Med/Low |
| Decision traceability | N decisions traced | N with relay gaps | High/Med/Low |

Style this table with the same header/alternating row pattern. Use ACCENT color for "Critical" or "High" severity text.
