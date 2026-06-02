---
name: screenshot-safe-widget
version: 2.0
description: >
  Renders HTML widgets using show_widget that look identical when screenshotted,
  exported to Canva, pasted into Figma/Notion, or viewed outside claude.ai.
  Applies to any visual the user will capture or export rather than interact with live.
triggers:
  - user says "screenshot", "export", "canva", "paste into", "save as image"
  - user says "for my report", "for my slides", "for my proposal", "for my presentation"
  - user uploads a document and asks for a matching visual
  - user requests a table, diagram, card, or chart they will use outside this chat
  - user says "make it screenshot safe" or "safe to screenshot"
  - any visualization that will appear in a PDF, Word doc, or slide deck
anti-triggers:
  - user wants a live interactive demo (sliders, calculators, games)
  - user wants dark-mode-aware UI that adapts to their system theme
  - widget is for display inside claude.ai only, never exported
constraints:
  - NEVER use var(--*) CSS variables anywhere in screenshot-safe widgets
  - NEVER use "Anthropic Sans" or var(--font-sans) as font family
  - NEVER use 0.5px borders — they vanish in screenshots, use 1px minimum
  - NEVER hardcode colors outside the approved palette below
  - ALWAYS ask the user for theme preference (white / black / custom) before rendering
    if they have not already specified one
  - ALWAYS use the outer wrapper pattern exactly as specified
  - Chart.js datasets: colors must be hardcoded hex, never CSS variables
  - Mermaid themeVariables: all color fields must be hardcoded hex
escalation:
  - If user requests a brand color not in the palette, ask for the hex value explicitly
  - If user requests a style that cannot be achieved with the safe palette, say so
    before rendering rather than approximating
---

# Screenshot-Safe Widget Skill

## Why This Exists

Claude's `show_widget` tool renders inside an iframe on claude.ai. The host page
injects CSS variables like `var(--color-text-primary)`. These variables **only resolve
inside claude.ai**. Outside that context — screenshots, Canva, Figma, Notion, other
browsers — they fall back to transparent or black, breaking the visual entirely.

The fix is simple: **hardcode every color as hex**.

---

## Step 0 — Ask for Theme (Required)

Before writing any widget code, if the user has not specified a visual style,
ask exactly this:

> "What background style do you want — **white** (clean, minimal, report-friendly),
> **black** (dark, high-contrast, slide-friendly), or describe a custom style in words?"

Map their answer to a theme block from the section below, then proceed.
Never skip this step for screenshot-intended widgets.

---

## Themes

### Theme A — White (default, report/Canva friendly)

```
Outer bg:        #ffffff
Page bg:         #F9F8F5
Surface/card bg: #F1EFE8
Light border:    #D3D1C7
Medium border:   #B4B2A9
Muted text:      #888780
Secondary text:  #5F5E5A
Primary text:    #444441
Heading text:    #2C2C2A
Highlight col bg:#F5F9FE   ← table highlighted column only
```

### Theme B — Black (slide / dark poster friendly)

```
Outer bg:        #0f0f0f
Page bg:         #1a1a1a
Surface/card bg: #242424
Light border:    #333333
Medium border:   #444444
Muted text:      #666666
Secondary text:  #999999
Primary text:    #cccccc
Heading text:    #f0f0f0
Highlight col bg:#1a2535   ← table highlighted column only
```

### Theme C — Custom

User describes style in words (e.g. "warm beige", "navy professional", "soft pastel").
Derive a consistent 9-stop palette from their description using the same role names above.
Show the palette as a small swatch table before rendering and ask for approval.

---

## Semantic Color Palette (applies to both themes)

These accent colors are theme-independent — use the same hex regardless of white/black bg.
Always pair fill + border + text from the same row.

| Meaning              | Fill      | Border    | Text on fill |
|----------------------|-----------|-----------|--------------|
| Our contribution/info| `#E6F1FB` | `#378ADD` | `#185FA5`    |
| Present / success    | `#E1F5EE` | `#1D9E75` | `#0F6E56`    |
| Partial / planned    | `#FAEEDA` | `#EF9F27` | `#854F0B`    |
| Absent / none        | `#F1EFE8` | `#D3D1C7` | `#888780`    |
| Danger / fail        | `#FCEBEB` | `#E24B4A` | `#A32D2D`    |
| Orchestrator/LLM     | `#EEEDFE` | `#7F77DD` | `#534AB7`    |
| Recon / enum         | `#E1F5EE` | `#1D9E75` | `#0F6E56`    |
| CVE / exploit        | `#FAECE7` | `#D85A30` | `#993C1D`    |
| Warning node         | `#FAEEDA` | `#BA7517` | `#854F0B`    |

For black theme: these semantic fills stay the same (they are light fills on white).
On dark backgrounds use the **border color as the fill** and **white (#f0f0f0) as text**:
- e.g. info node on dark: `background:#185FA5; border:1px solid #378ADD; color:#ffffff`

---

## Font Stack (always use these — never Anthropic Sans)

```css
/* body / UI */
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;

/* code blocks */
font-family: 'Menlo', 'Monaco', 'Consolas', monospace;
```

---

## Required Outer Wrapper

Every screenshot-safe widget MUST start with this. Swap bg/font colors per chosen theme.

```html
<!-- WHITE theme -->
<div style="background:#ffffff;padding:24px;
            font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;
            max-width:680px">

<!-- BLACK theme -->
<div style="background:#0f0f0f;padding:24px;
            font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;
            max-width:680px">
```

Rules:
- `padding:24px` — breathing room for screenshot cropping
- `max-width:680px` — matches claude.ai widget container width
- No `var(--*)` anywhere inside

---

## Component Patterns

### Section heading

```html
<!-- white theme -->
<div style="font-size:13px;font-weight:500;color:#444441;
            margin-bottom:14px;letter-spacing:.03em">
  Section title
</div>

<!-- black theme -->
<div style="font-size:13px;font-weight:500;color:#f0f0f0;
            margin-bottom:14px;letter-spacing:.03em">
  Section title
</div>
```

### Badge / pill

```html
<!-- absent/none -->
<span style="display:inline-block;padding:2px 8px;border-radius:4px;
             font-size:10px;font-weight:500;
             background:#F1EFE8;color:#888780;border:1px solid #D3D1C7">None</span>

<!-- present/success -->
<span style="display:inline-block;padding:2px 8px;border-radius:4px;
             font-size:10px;font-weight:500;
             background:#E1F5EE;color:#0F6E56;border:1px solid #1D9E75">Yes</span>

<!-- partial/planned -->
<span style="display:inline-block;padding:2px 8px;border-radius:4px;
             font-size:10px;font-weight:500;
             background:#FAEEDA;color:#854F0B;border:1px solid #EF9F27">Partial</span>

<!-- our contribution -->
<span style="display:inline-block;padding:2px 8px;border-radius:4px;
             font-size:10px;font-weight:500;
             background:#E6F1FB;color:#185FA5;border:1px solid #378ADD">Ours</span>

<!-- danger/fail -->
<span style="display:inline-block;padding:2px 8px;border-radius:4px;
             font-size:10px;font-weight:500;
             background:#FCEBEB;color:#A32D2D;border:1px solid #E24B4A">Fail</span>

<!-- novel accent (solid green) -->
<span style="font-size:9px;padding:1px 6px;border-radius:3px;
             background:#1D9E75;color:#ffffff;font-weight:600">NOVEL</span>
```

### Card

```html
<!-- white theme -->
<div style="background:#ffffff;border:1px solid #D3D1C7;border-radius:12px;
            padding:18px;display:flex;flex-direction:column;gap:10px">
  <div style="font-size:10px;font-weight:600;text-transform:uppercase;
              letter-spacing:.08em;color:#B4B2A9">Label</div>
  <div style="font-size:13px;font-weight:600;color:#2C2C2A">Card Title</div>
  <div style="font-size:11.5px;color:#5F5E5A;line-height:1.6">Body text.</div>
  <div style="background:#F9F8F5;border-radius:8px;padding:12px">
    <!-- inner content -->
  </div>
</div>

<!-- black theme — swap colors -->
<div style="background:#1a1a1a;border:1px solid #333333;border-radius:12px;
            padding:18px;display:flex;flex-direction:column;gap:10px">
  <div style="font-size:10px;font-weight:600;text-transform:uppercase;
              letter-spacing:.08em;color:#666666">Label</div>
  <div style="font-size:13px;font-weight:600;color:#f0f0f0">Card Title</div>
  <div style="font-size:11.5px;color:#999999;line-height:1.6">Body text.</div>
  <div style="background:#242424;border-radius:8px;padding:12px">
    <!-- inner content -->
  </div>
</div>
```

### Table

```html
<!-- white theme — swap border/text colors for black theme using palette above -->
<table style="width:100%;border-collapse:collapse;font-size:11.5px">
  <thead>
    <tr style="border-bottom:1.5px solid #B4B2A9">
      <th style="padding:9px 12px;font-weight:600;font-size:10px;letter-spacing:.05em;
                 text-transform:uppercase;color:#888780;text-align:left">
        Column
      </th>
      <!-- highlighted "Ours" column -->
      <th style="padding:9px 12px;font-weight:600;font-size:10px;letter-spacing:.05em;
                 text-transform:uppercase;color:#185FA5;text-align:left;
                 background:#F5F9FE">
        Ours
      </th>
    </tr>
  </thead>
  <tbody>
    <tr style="border-bottom:1px solid #F1EFE8">
      <td style="padding:8px 12px;font-weight:500;color:#2C2C2A;font-size:11.5px">
        Row label
      </td>
      <td style="padding:8px 10px"><!-- badge --></td>
      <td style="padding:8px 10px;background:#F5F9FE"><!-- badge --></td>
    </tr>
  </tbody>
</table>
```

### Code block (syntax highlighted)

```html
<div style="font-family:'Menlo','Monaco','Consolas',monospace;
            font-size:10.5px;background:#ffffff;border:1px solid #D3D1C7;
            border-radius:6px;padding:8px;color:#5F5E5A;line-height:1.9">
  <span style="color:#888780"># comment</span><br>
  <span style="color:#185FA5">key:</span> <span style="color:#3B6D11">value</span><br>
  <span style="color:#993C1D">- list item</span><br>
</div>
```

Syntax roles (same hex in both themes):

| Role            | Hex       |
|-----------------|-----------|
| Comments / `---`| `#888780` |
| Keys / props    | `#185FA5` |
| String values   | `#3B6D11` |
| List items / cmd| `#993C1D` |
| Default body    | `#5F5E5A` |

### macOS window chrome

```html
<div style="background:#F1EFE8;border-bottom:1px solid #D3D1C7;
            padding:9px 14px;display:flex;align-items:center;gap:7px">
  <span style="width:9px;height:9px;border-radius:50%;
               background:#E24B4A;display:inline-block"></span>
  <span style="width:9px;height:9px;border-radius:50%;
               background:#EF9F27;display:inline-block"></span>
  <span style="width:9px;height:9px;border-radius:50%;
               background:#639922;display:inline-block"></span>
  <span style="font-size:11px;font-weight:500;color:#5F5E5A;margin-left:6px">
    filename.ext
  </span>
</div>
```

### Grid layout

```html
<!-- 3 equal columns -->
<div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:12px">
</div>

<!-- 2 columns: code left, explanation right -->
<div style="display:grid;grid-template-columns:1fr 1fr;gap:16px;align-items:start">
</div>
```

---

## Special Rules for Chart.js (screenshot-safe)

Chart.js resolves colors at render time. CSS variables are NOT resolved inside a canvas.

```javascript
// WRONG — colors will be empty strings in canvas
backgroundColor: 'var(--color-background-info)'

// CORRECT — always hardcode hex in Chart.js datasets
backgroundColor: '#E6F1FB'
borderColor: '#378ADD'

// Also hardcode tick/grid/axis colors in scales:
scales: {
  x: {
    ticks: { color: '#888780' },        // NOT var(--)
    grid:  { color: '#F1EFE8' },        // NOT var(--)
    border:{ color: '#D3D1C7' }         // NOT var(--)
  }
}
```

---

## Special Rules for Mermaid (screenshot-safe)

Mermaid's `themeVariables` must also be hardcoded. Additionally, force
`darkMode: false` regardless of system preference, and set `background: '#ffffff'`
(or `'#0f0f0f'` for black theme) explicitly on the rendered SVG after generation.

```javascript
mermaid.initialize({
  theme: 'base',
  themeVariables: {
    darkMode: false,                        // force, do not inherit
    background:       '#ffffff',            // hardcoded
    primaryColor:     '#EEEDFE',
    primaryTextColor: '#26215C',
    primaryBorderColor:'#7F77DD',
    lineColor:        '#888780',
    edgeLabelBackground: '#ffffff',
    // ... all fields hardcoded hex
  }
});

// After render, also force bg on the SVG element:
document.querySelector('#container svg').style.background = '#ffffff';
```

---

## Pre-flight Checklist (mandatory before every show_widget call)

Run through this mentally before writing widget code:

- [ ] Theme confirmed with user (white / black / custom)
- [ ] Outer wrapper uses correct theme bg (`#ffffff` or `#0f0f0f`)
- [ ] Zero instances of `var(--*)` in the HTML
- [ ] Font stack is system fonts — no `"Anthropic Sans"`
- [ ] All borders are `1px` minimum (not `0.5px`)
- [ ] All text colors are from the approved palette
- [ ] Chart.js dataset colors are hardcoded hex (if chart present)
- [ ] Mermaid themeVariables are hardcoded hex + `darkMode:false` (if diagram present)
- [ ] `#F5F9FE` used only for table highlight column bg (white theme only)
- [ ] `max-width:680px` on outer wrapper
- [ ] `padding:24px` on outer wrapper

---

## When CSS Variables Are Still Acceptable

Use CSS variables ONLY when ALL of the following are true:
- The widget is interactive (sliders, live state, calculators, games)
- It will be used inside claude.ai only, never exported
- The user has not mentioned screenshotting, exporting, or pasting elsewhere

If any doubt exists — use hardcoded hex. The cost is zero. The benefit is reliability.
