# Feature Design Spec — Prompt Debugger with Step-Level Transcript Annotation

**Feature:** Prompt Debugger
**Product brief reference:** PRODUCT_BRIEF.md, Feature 1
**Priority:** P0
**Author:** Arad Adler
**Status:** Design spec — ready for Figma

---

## 1. Feature Overview

The Prompt Debugger is a transcript analysis view inside the Retell AI dashboard that annotates every agent turn in a call with the specific prompt section that drove it. Today, when a call goes wrong, a builder has only the raw transcript: they can read what the agent said but have no visibility into why. Debugging is manual archaeology — reading the transcript, re-reading the prompt, forming a hypothesis, running another test, and repeating until the root cause surfaces. The Prompt Debugger closes this loop by making the relationship between prompt and behavior explicit and inspectable. Every agent turn is tagged with its active prompt section, labeled with a confidence signal, and linked back to the exact prompt text that was in play. A Confidence Timeline surfaces failure moments at a glance. A Prompt Diff View lets builders validate a fix before deploying it. The result is a debugging experience that cuts prompt iteration time from hours to minutes.

**Core user problem:** A builder watches a call go wrong — the agent escalated when it should have resolved, or asked a stacked question, or skipped a step — and has no systematic way to identify which part of the prompt caused it. The only tool available today is reading the transcript manually and guessing.

**Primary persona:** A technical PM or developer who has deployed a voice agent to production and is investigating a specific call failure. They know their prompt well, but they need to see the mapping between prompt sections and agent behavior without doing it by hand.

---

## 2. User Journey: Before vs. After

### Scenario
A builder's support agent (Alex) handled a billing inquiry call and escalated it to a human rep. The prompt says billing questions should be deferred — that's correct. But the agent escalated on the *second* turn, before even collecting the caller's account information. The builder needs to understand why the step-ordering guardrail didn't fire.

---

### Before (today)

1. Builder receives an alert or reviews the calls list and spots the short-duration, escalated call.
2. Opens the transcript view. Reads it top to bottom. Sees the agent escalate early.
3. Opens the prompt in a separate tab. Reads through Step 2 (account collection) and Step 3 (billing triage) trying to match the agent's behavior to the instruction text.
4. Forms a hypothesis: the billing triage instruction is too early in the prompt and the model is pattern-matching on "billing" before reaching the step-ordering guardrail.
5. Has no way to confirm this. Edits the prompt, runs a new simulation, reads the new transcript, and compares manually.
6. Repeat 1-3 more times before finding a version that behaves correctly.
7. Total time: 45-90 minutes for a single-cause failure. Multi-cause failures can take days.

**Where they get stuck:** Step 3. There is no tool to tell them which prompt section the agent was "in" when it made the early escalation decision. The hypothesis is educated guessing.

---

### After (with Prompt Debugger)

1. Builder opens the call from the calls list. A "Debug" button appears next to the transcript view. They click it.
2. The Prompt Debugger screen loads. The Confidence Timeline strip at the top of the transcript shows a sharp drop to red (Low confidence) on turn 3 of 11.
3. Builder clicks the red point on the timeline. The transcript scrolls to turn 3. The agent turn is tagged with a yellow left-border labeled "Task Flow: Step 3 (Billing Triage)".
4. Builder hovers the tag. A tooltip expands showing: the prompt section name ("Task Flow — Step 3: Billing Triage"), the exact prompt text ("Acknowledge the question. Explain that billing disputes... require access to financial records..."), and a confidence pill: "Low — step-ordering conflict detected."
5. Builder glances at the Prompt Section Heatmap in the right sidebar. "Task Flow: Step 2" shows zero activations. That confirms: Step 2 (account collection) never fired. The agent jumped from Step 1 directly to Step 3.
6. Builder clicks "Compare with..." and selects a draft prompt they prepared where Step 2 has a stronger ordering instruction. The Prompt Diff View opens. The same transcript replays against both versions side by side. In the right panel (new prompt), turn 3 is now tagged "Task Flow: Step 2" and the agent asks for the account email instead of escalating.
7. Builder deploys the updated prompt. Total time: 8 minutes.

---

## 3. Feature Components

---

### Component A — Annotated Transcript View

**What it is:**
The primary content panel. Displays the full call transcript in chronological order, with each agent turn annotated by a colored left-border tag corresponding to its active prompt section. User turns are displayed without annotation. The component is the primary workspace for call inspection.

**Where it lives:**
Center panel of the Prompt Debugger screen. Occupies the main content area between the left Confidence Timeline (which sits above it as a fixed strip) and the right Prompt Section Heatmap sidebar.

**What data it displays:**
- Full call transcript: alternating user and agent turns
- Per agent turn: prompt section tag (color-coded left border + section label), speaker label, turn text, turn timestamp (MM:SS)
- On hover: expanded tooltip (see Component A tooltip spec in Section 5)
- Confidence pill visible inline on each agent turn (not just on hover): small colored dot to the right of the section label

**Interaction behavior:**
- Hover on any agent turn: expands tooltip showing prompt section name, exact prompt text excerpt (truncated to 3 lines with "show more"), and confidence pill (High / Medium / Low)
- Click on a turn: selects it (highlighted background), and the corresponding section in the Prompt Section Heatmap highlights
- Click again to deselect
- Scroll: standard vertical scroll; Confidence Timeline strip stays fixed at top of panel
- "Show prompt text" toggle in the top-right of the panel: expands all tooltips inline (useful for detailed review without hovering)

**Figma design notes:**

- Panel width: 680px. Height: full viewport minus top nav (64px) and top bar (56px) and Confidence Timeline strip (72px). Scrollable.
- Background: `white` (`#FFFFFF`)
- Each transcript turn is a row. Row padding: 16px horizontal, 12px vertical. No row borders — separate turns with 1px `gray-100` dividers.
- User turns: no left border. Speaker label "Caller" in `text-xs` `gray-400` above the turn text. Turn text in `text-sm` `gray-700`.
- Agent turns: 4px solid left border in the section color (see Section 5 color tokens). Speaker label "Alex" in `text-xs` `gray-400` above the turn text. Turn text in `text-sm` `gray-900`.
- Section tag: sits inline to the right of the speaker label on the same line. `text-xs` `font-medium`, background is a 10% opacity version of the section color, text is the full section color. Rounded-full pill shape. Padding: 2px 8px. Example: "Task Flow: Step 3".
- Confidence dot: 8px circle, sits immediately right of the section tag pill. Color per Section 5. No label on the dot — tooltip provides the label.
- Timestamp: `text-xs` `gray-400`, right-aligned on the same row as the speaker label.
- Hover state: row background shifts to `gray-50`. Tooltip appears above the turn (see tooltip spec in Section 5).
- Selected state: row background `blue-50`, left border color intensifies to full opacity (from the section color).
- Tooltip: see Section 5.

---

### Component B — Prompt Section Heatmap

**What it is:**
A right sidebar panel showing the agent's prompt broken into its named sections. Each section has a horizontal activity bar representing how many turns in this call activated that section. It gives the builder a structural "where did my prompt actually fire?" view across the whole call.

**Where it lives:**
Right sidebar of the Prompt Debugger screen. Fixed position, non-scrollable (or internally scrollable if prompt sections exceed panel height).

**What data it displays:**
- List of all named prompt sections in order: Identity, Style Guardrails, Task Flow Step 1, Task Flow Step 2 ... Task Flow Step N, Edge Cases
- Per section: activation count ("4 turns"), horizontal bar proportional to max activations in any single section, section color dot
- Total turns in call shown at top of panel: "11 turns total"

**Interaction behavior:**
- Click a section row: highlights all corresponding turns in the Annotated Transcript View with a pulsing ring animation (300ms ease). All other turns dim to 40% opacity.
- Click again or click elsewhere: clears the highlight/dim.
- Hover a section row: shows a tooltip with the first 2 lines of that prompt section's text.
- Sections with zero activations: bar is absent, count reads "0 turns", row text in `gray-400` (visually de-emphasized but present — zero activations are diagnostically important).

**Figma design notes:**

- Panel width: 260px. Height: full viewport minus top nav (64px) and top bar (56px). Background: `gray-50`.
- Panel header: "Prompt Sections" in `text-sm` `font-semibold` `gray-900`. Below it: "11 turns total" in `text-xs` `gray-500`. Header padding: 16px. Bottom border: 1px `gray-200`.
- Each section row: 48px tall. Padding: 12px 16px. Stack (top to bottom): section name in `text-xs` `font-medium` `gray-700`, then below it: horizontal bar + activation count on the same line.
- Section color dot: 8px circle, left-aligned, matching the section's color token (Section 5). Sits left of the section name, vertically centered.
- Horizontal bar: height 4px, rounded-full, background `gray-200`, filled portion uses the section color at full opacity. Bar width is proportional: (section activations / max activations) * 148px. Max bar width: 148px.
- Activation count: `text-xs` `gray-500`, right-aligned on the bar row. Format: "4 turns".
- Row hover: background `gray-100`.
- Row selected (after click): background `blue-50`, left edge gets a 3px `blue-500` inset border.
- Zero-activation rows: bar fill absent, count "0 turns" in `gray-300`, section name in `gray-400`.

---

### Component C — Confidence Timeline

**What it is:**
A fixed horizontal strip pinned above the Annotated Transcript View. Plots the confidence level of each agent turn as a connected line graph across the call duration. Confidence is expressed as High (green), Medium (yellow), or Low (red). A sudden drop to Low is the fastest way to find where a call went wrong.

**Where it lives:**
Fixed strip at the top of the center panel, below the top bar and above the Annotated Transcript View. Spans the full 680px width of the center panel.

**What data it displays:**
- X axis: turn index (1 through N), evenly spaced
- Y axis: three positions — High (top), Medium (middle), Low (bottom) — no numeric values
- Line: connected SVG polyline, colored per the confidence of each point (line segments transition color between points using a gradient)
- Each point: filled circle (8px diameter), colored per confidence level
- X axis labels: turn numbers below the axis line, `text-xs` `gray-400`
- Y axis labels: "High", "Med", "Low" on the left, `text-xs` `gray-400`

**Interaction behavior:**
- Hover a point: tooltip shows turn number, speaker turn text preview (first 8 words), and confidence level label
- Click a point: transcript scrolls to that turn and selects it (same as clicking the turn directly in the Annotated Transcript View)
- Active point (currently selected): point circle has a 3px white ring plus outer ring in the point's confidence color (focus indicator)

**Figma design notes:**

- Strip height: 72px. Width: 680px (matches center panel). Background: `white`. Bottom border: 1px `gray-200`.
- Internal padding: 8px top, 12px left/right, 4px bottom.
- Y axis label column: 28px wide. Labels "High" / "Med" / "Low" in `text-xs` `gray-400`, vertically distributed across the 3 positions.
- Chart area: 640px wide. SVG canvas within the strip.
- Three horizontal gridlines (dashed, 1px `gray-100`) at High, Medium, Low positions.
- Polyline: stroke-width 2px, line connects all points. Segment color interpolates between the confidence color of adjacent points.
- Point circles: 8px diameter, filled with confidence color, 1.5px white stroke (to separate from line).
- High: `green-500` (`#22C55E`). Medium: `yellow-400` (`#FACC15`). Low: `red-500` (`#EF4444`). See Section 5.
- Hover tooltip: appears above the hovered point. Background `gray-900`, text `white`, `text-xs`, rounded-md, padding 6px 10px, max-width 200px.

---

### Component D — Prompt Diff View

**What it is:**
A split-screen overlay accessed from the top bar's "Compare with..." button. Displays the current prompt version on the left and a selected alternate version on the right. The call transcript replays against both versions simultaneously, showing side-by-side agent responses per turn. Behavioral differences between versions are highlighted, letting the builder see whether a prompt change actually fixes the failure before deploying.

**Where it lives:**
Full-screen overlay that replaces the standard Prompt Debugger view. Triggered by the "Compare with..." button in the top bar. Dismissible via an "X" or "Back to transcript" link in the top-left.

**What data it displays:**
- Left panel header: "Current — v[version number]" + deploy date in `text-xs` `gray-500`
- Right panel header: "Comparing — [selected version name or draft]" + version date
- Per turn: both panels show the agent's response for that turn. User turns are shown once in a shared center column between the two panels.
- Turns where the agent response differs between versions: highlighted with a `yellow-50` background and a "Differs" pill in `yellow-600`
- Turns that are identical: shown at reduced opacity (`gray-400` text) to visually focus attention on differences
- Section annotation tags visible in both panels (same as Component A)

**Interaction behavior:**
- "Compare with..." button opens a modal dropdown listing: available deployed versions (by date), saved drafts, and a "Paste prompt text" option for ad-hoc comparison
- Selecting a version closes the modal and loads the Diff View
- Scroll is synchronized between left and right panels — scrolling one scrolls both
- Clicking a "Differs" turn: expands both panels' annotation tooltips simultaneously for that turn
- "Export diff" button in top-right: downloads a markdown file of the comparison (turn number, left response, right response, section tag, confidence, differs: true/false)

**Figma design notes:**

- Full-screen overlay: 1440px wide, full viewport height. Background: `white`.
- Top bar (diff view): 56px tall, `gray-50` background, 1px `gray-200` bottom border. Contains: back link left-aligned (`text-sm` `blue-600` "Back to transcript"), center title "Prompt Diff" in `text-sm` `font-semibold` `gray-900`, right-aligned "Export diff" button (outlined, `text-sm` `gray-700`).
- Center user turn column: 120px wide, centered. Background: `gray-50`. User turn text in `text-xs` `gray-500`, center-aligned. Turn number above each user turn in `text-xs` `gray-300`.
- Left panel: 648px wide. Right panel: 648px wide. Panels are separated by the 120px center column. Panels have 24px internal horizontal padding.
- Panel headers: 40px tall. Version label `text-sm` `font-semibold` `gray-800`, date in `text-xs` `gray-400` beside it. Bottom border 1px `gray-200`.
- Agent turn rows: same as Component A (16px/12px padding, left-border section tag) with the following additions: "Differs" pill (`text-xs` `font-medium` `yellow-700` on `yellow-100` background, rounded-full, 2px 8px padding) appended after the section tag on differing turns. Row background on differing turns: `yellow-50`.
- Identical turns: agent text rendered at `gray-400`, section tag pill at 50% opacity.
- Synchronized scroll: implement as a note in Figma ("Sync scroll — prototype connection").

---

## 4. Screen Layout Spec

**Canvas:** 1440px wide, full viewport height (design at 900px tall, scrollable).

### Zone breakdown (left to right):

| Zone | Component | Width | Background |
|------|-----------|-------|------------|
| Left sidebar | Prompt Section Heatmap (Component B) | 260px | `gray-50` |
| Center panel | Confidence Timeline (C) + Annotated Transcript View (A) | 680px | `white` |
| Right sidebar | Reserved (call metadata, future: related calls) | 240px | `gray-50` |
| Right gutter | Visual breathing room | 260px | `white` |

> Note: The 260px right column is intentionally left as call metadata in v1 (call date, duration, agent name, phone number, outcome tag). It is not a functional component in this spec but should be mocked with placeholder content so the layout reads as complete.

### Top nav (global, not feature-specific):
- Height: 64px. Background: `white`. Bottom border: 1px `gray-200`.
- Contains: Retell AI logo (left), primary nav links (center), account avatar (right).
- This nav is unchanged from the standard Retell dashboard.

### Top bar (feature-specific, below global nav):
- Height: 56px. Background: `white`. Bottom border: 1px `gray-200`. Full 1440px width.
- Left side (16px left padding): Agent name in `text-sm` `font-semibold` `gray-900` ("Alex — Support Agent"), then a `gray-300` divider (1px, 20px tall), then call date/time in `text-sm` `gray-500` ("Apr 14, 2026 at 2:34 PM"), then call duration in `text-sm` `gray-500` ("3m 12s").
- Right side (16px right padding): "Export" button (outlined, `text-sm` `gray-600`, icon: download) and "Compare with..." button (filled `blue-600`, `text-sm` `font-medium` `white`, rounded-md, padding 8px 16px). Buttons separated by 8px gap.

### Center panel internal stacking (top to bottom):
1. Confidence Timeline strip (Component C): 72px tall, fixed at top of center panel, does not scroll.
2. Annotated Transcript View (Component A): fills remaining center panel height, scrollable.

### Left sidebar:
- Prompt Section Heatmap (Component B) starts at top of sidebar, below global nav and top bar (64px + 56px = 120px from top of viewport).
- Sidebar is fixed position (does not scroll with transcript).

### Right sidebar (metadata, v1):
- 240px wide. Background `gray-50`. Contains: "Call Details" header in `text-sm` `font-semibold` `gray-900`, then a list of metadata rows (label: value format). Labels in `text-xs` `gray-500`, values in `text-xs` `gray-700`. Rows: Agent, Call date, Duration, Phone number, Outcome (e.g., "Escalated"), Call ID.

---

## 5. Color and Interaction System

### Prompt section color tokens

| Prompt section | Tailwind color | Hex | Usage |
|---------------|---------------|-----|-------|
| Identity | `violet-500` | `#8B5CF6` | Left border, tag background (10% opacity), dot |
| Style Guardrails | `sky-500` | `#0EA5E9` | Left border, tag background, dot |
| Task Flow: Step 1 | `teal-500` | `#14B8A6` | Left border, tag background, dot |
| Task Flow: Step 2 | `teal-600` | `#0D9488` | Left border, tag background, dot |
| Task Flow: Step 3 | `teal-700` | `#0F766E` | Left border, tag background, dot |
| Task Flow: Step 4+ | `teal-800` | `#115E59` | Continue darkening for additional steps |
| Edge Cases | `orange-500` | `#F97316` | Left border, tag background, dot |
| Unclassified | `gray-400` | `#9CA3AF` | Fallback for turns with no clear section match |

> Task Flow steps use a teal ramp intentionally — they are the most common section and a graduated color helps builders distinguish step 3 from step 5 at a glance without introducing too many hues.

### Confidence level colors

| Confidence | Tailwind color | Hex | Usage |
|-----------|---------------|-----|-------|
| High | `green-500` | `#22C55E` | Dot, timeline point, timeline line segment, pill background `green-50` / text `green-700` |
| Medium | `yellow-400` | `#FACC15` | Dot, timeline point, timeline line segment, pill background `yellow-50` / text `yellow-700` |
| Low | `red-500` | `#EF4444` | Dot, timeline point, timeline line segment, pill background `red-50` / text `red-700` |

### Hover state behavior
- Transcript row hover: background `gray-50` (transition: 100ms ease). Tooltip appears 8px above the row, centered on the section tag.
- Heatmap row hover: background `gray-100` (transition: 100ms ease). Tooltip appears to the left of the row (since the heatmap is the rightmost interactive element).
- Timeline point hover: point expands from 8px to 12px diameter (transition: 150ms ease). Tooltip appears above the point.

### Active / selected state behavior
- Selected transcript turn: background `blue-50`, left border at full section color opacity (was 4px, stays 4px but visually pops against the tinted background).
- Selected heatmap section row: background `blue-50`, 3px left inset border `blue-500`. All non-corresponding transcript turns dimmed to 40% opacity (`opacity-40`).
- Selected timeline point: 8px diameter point gains a 3px white ring and a 2px outer ring in the confidence color (total visual diameter: ~18px including rings).

### Tooltip style spec
- Background: `gray-900`
- Text: `white`, `text-xs`
- Border radius: `rounded-md` (6px)
- Padding: 8px 12px
- Max width: 280px
- Box shadow: none (flat)
- Structure (transcript turn tooltip): Section name in `text-xs` `font-semibold` `white` on line 1. Prompt text excerpt (up to 3 lines) in `text-xs` `gray-300` on line 2 (with "show more" link in `blue-400` if truncated). Confidence pill on line 3: colored pill with confidence label.
- Arrow: 6px triangle pointing down toward the hovered element, same `gray-900` background
- Appear/disappear: 120ms fade, no delay

---

## 6. Empty and Error States

### Component A — Annotated Transcript View

**No call selected:**
Center panel shows an empty state illustration (simple icon: a transcript page with a magnifying glass, drawn in `gray-300` line style, 64px). Below it: "Select a call to inspect" in `text-sm` `gray-500`, centered. No transcript content.

**Loading:**
Each turn row replaced with a skeleton loader: a 12px tall rounded `gray-200` bar at 80% of row width, animated with a left-to-right shimmer (standard skeleton pattern). Section tag area shows a 60px wide `gray-200` pill skeleton. Show 6 skeleton rows.

**Annotation data unavailable (call predates feature):**
Transcript displays normally (turn text visible) but section tags are replaced with a `gray-200` pill reading "Not available" in `gray-400` `text-xs`. A banner at the top of the transcript panel: `yellow-50` background, `yellow-800` text, 1px `yellow-200` border: "Annotation data is only available for calls after [feature launch date]. This transcript can be read but not debugged." Banner height: 40px, full panel width, dismissible with an X.

---

### Component B — Prompt Section Heatmap

**No call selected:**
Section list renders in skeleton form: each row shows a gray section name skeleton and an empty bar track. Below the header: "Select a call to see prompt activation" in `text-xs` `gray-400`.

**Loading:**
Same skeleton treatment as Component A: gray bars animate with shimmer.

**Annotation data unavailable:**
Section list renders with all bars empty and all activation counts reading "--". Section names are visible (they come from the prompt config, not the annotation data). A note below the list in `text-xs` `gray-400`: "Activation data not available for this call."

---

### Component C — Confidence Timeline

**No call selected:**
Empty chart area with three dashed horizontal gridlines and the Y axis labels. Center of chart: "No call selected" in `text-xs` `gray-400`.

**Loading:**
Animated skeleton: a wavy gray line placeholder across the chart area (sinusoidal path in `gray-200`, no actual data points). Points replaced with `gray-200` circles.

**Annotation data unavailable:**
Timeline area shows a flat dashed line at the Medium position in `gray-300`, spanning the full call duration. A tooltip on hover anywhere on the line: "Confidence data unavailable for this call."

---

### Component D — Prompt Diff View

**No alternate version selected (just opened "Compare with..."):**
Right panel shows the version selector dropdown open over a grayed-out empty state: right panel background `gray-50`, center text "Select a version to compare" in `text-sm` `gray-400`.

**Loading (after version selected):**
Right panel fills with skeleton transcript rows (same as Component A loading state). Left panel remains fully rendered.

**Selected version has no shared turns with current call (edge case: very different prompt structure):**
All right panel turns show a "No match" state: gray turn text and a `gray-200` pill reading "Unmatched". Banner at top of right panel: "The selected version produced a different number of turns — comparison may be incomplete."

