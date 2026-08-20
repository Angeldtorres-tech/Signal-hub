# Signal Hub — Build Spec
*For Codex. Read this fully before writing a single line of code.*

---

## What This Is

A personal AI intelligence portal for Angel Torres — an independent AI transformation strategist actively job searching. Replaces a daily Telegram brief with a beautiful, searchable, chat-enabled web app. Opens every morning. Built to be read, not skimmed.

---

## Design System

Source reference: hellopm.co/design-god — editorial warmth adapted to dark mode.

### Color Tokens
```css
--bg-base:        #0D0B08;   /* warm near-black, NOT cool gray */
--bg-card:        #1A1712;   /* card surface */
--bg-card-hover:  #211E18;
--bg-chat:        #131009;   /* chat panel */
--border:         rgba(243, 237, 225, 0.08);
--border-strong:  rgba(243, 237, 225, 0.14);

--text-primary:   #F3EDE1;   /* warm cream — the hellopm bg becomes our text */
--text-muted:     #8A9486;   /* muted olive/sage */
--text-faint:     #4A4A3E;

--accent:         #E0583A;   /* coral/terracotta from hellopm headline accent */
--accent-hover:   #C94E32;
--accent-subtle:  rgba(224, 88, 58, 0.12);
--accent-border:  rgba(224, 88, 58, 0.28);

--linkedin:       #0A66C2;
--linkedin-subtle: rgba(10, 102, 194, 0.10);

--tag-new-bg:     rgba(74, 222, 128, 0.10);
--tag-new-text:   #4ADE80;
--tag-rising-bg:  rgba(251, 191, 36, 0.10);
--tag-rising-text:#FBBF24;
--tag-alert-bg:   rgba(239, 68, 68, 0.10);
--tag-alert-text: #F87171;
```

### Typography
```
Headlines:    'Playfair Display', Georgia, serif  — bold, editorial weight
Body:         'DM Sans', system-ui, sans-serif
Mono:         'JetBrains Mono', 'Menlo', monospace
```

Install via Google Fonts in layout.tsx:
```
Playfair_Display (weights: 700, 800)
DM_Sans (weights: 400, 500, 600)
JetBrains_Mono (weight: 400)
```

### Key Design Rules
- NO cool grays. Everything has warm undertones.
- Borders are warm white with low opacity, never hard lines.
- Accent (coral) is used sparingly — section titles, CTAs, one word in headlines.
- Cards have 12px radius. Sections have 14px radius.
- Spacing: generous. This is a reading experience, not a dashboard.
- Labels above cards use small-caps, muted olive, 10-11px, 1.5px letter-spacing.

---

## Layout

```
┌─────────────────────────────────────────────────────────┐
│  HEADER (sticky)                                         │
│  [— Signal Hub]  [Thu Aug 20]  [🔍 search]  [Chat ›]   │
├──────────────────────────────────────┬──────────────────┤
│                                      │                  │
│  BRIEF CONTENT (scrollable)          │  CHAT PANEL      │
│  Sections collapse/expand            │  (380px fixed)   │
│                                      │  Right side      │
│                                      │  Always visible  │
│                                      │  on desktop      │
│                                      │                  │
└──────────────────────────────────────┴──────────────────┘
```

- Brief content: fluid width, max 720px, left-aligned with padding
- Chat panel: 380px, fixed right, full viewport height, scrollable messages
- On mobile (<768px): chat panel slides in as a bottom sheet, toggled by header button
- Header: sticky, 64px tall, warm blur backdrop

---

## File Structure

```
signal-hub/
├── src/
│   ├── app/
│   │   ├── layout.tsx          # Root layout — fonts, global styles, split-pane structure
│   │   ├── page.tsx            # Today's brief (redirects to /brief/today)
│   │   ├── brief/
│   │   │   └── [date]/
│   │   │       └── page.tsx    # Brief for a specific date
│   │   └── api/
│   │       ├── brief/
│   │       │   └── [date]/
│   │       │       └── route.ts  # Returns brief JSON for a date
│   │       └── chat/
│   │           └── route.ts      # Streams chat response from Anthropic
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   └── ChatPanel.tsx
│   │   ├── brief/
│   │   │   ├── BriefShell.tsx      # Outer container, search state
│   │   │   ├── Section.tsx         # Collapsible section wrapper
│   │   │   ├── SignalCard.tsx      # Reusable card: title + rows
│   │   │   ├── TechSignals.tsx
│   │   │   ├── HotThisWeek.tsx
│   │   │   ├── RoleIntelligence.tsx
│   │   │   ├── LinkedInDraft.tsx   # With copy button
│   │   │   ├── RisingSignal.tsx
│   │   │   ├── ResumeGap.tsx
│   │   │   └── TokenIntel.tsx      # Only renders if brief.tokenIntel.hasFlag === true
│   │   └── chat/
│   │       ├── ChatMessages.tsx
│   │       ├── ChatInput.tsx
│   │       └── ChatMessage.tsx
│   ├── lib/
│   │   ├── brief.ts            # loadBrief(date), listBriefs()
│   │   ├── search.ts           # Full-text search logic
│   │   └── types.ts            # All TypeScript types
│   └── data/
│       └── briefs/             # Brief JSON files land here
│           └── 2026-08-20.json # Example
```

---

## Data Model

### Brief JSON (`data/briefs/YYYY-MM-DD.json`)

```typescript
// types.ts
export interface SignalRow {
  label: string;      // "Signal" | "Why it matters" | "How to capitalize"
  text: string;
  mono?: boolean;     // render in monospace code block if true
}

export interface SignalCard {
  title: string;
  rows: SignalRow[];
}

export interface RoleTrack {
  track: string;      // "AI Enablement Lead / Senior / Head"
  trendingSkill: string;
  whatItMeans: string;
  showcaseIt: string;  // verbatim phrase (rendered as mono)
  freeResource: { label: string; url: string };
}

export interface LinkedInDraft {
  angle: string;      // "Today's sharpest angle · C2PA / Proactive Governance"
  body: string;       // full draft text, newlines preserved
}

export interface RisingSignal {
  topic: string;
  whatGainingTraction: string;
  whyItMatters: string;
  howToGetAhead: string;
}

export interface ResumeGapItem {
  gap: string;
  addThisPhrase: string;  // verbatim, rendered mono
  where?: string;         // "Core Competencies" etc.
}

export interface ResumeGap {
  todayFocus: string;     // "AIEnablement_v1"
  rotationNote: string;   // "Mon: AIEnablement_v1 · Tue: PM_v2 · ..."
  gaps: ResumeGapItem[];
}

export interface TokenIntel {
  hasFlag: boolean;
  deadline?: string;
  deadlineNote?: string;
  marketContext?: string;
  optimizationAction?: string;
}

export interface Brief {
  date: string;                        // "2026-08-20"
  displayDate: string;                 // "Thu · Aug 20, 2026"
  techSignals: SignalCard[];           // 5 items
  hotThisWeek: SignalCard[];           // 3 items
  roleIntelligence: RoleTrack[];       // 3 items
  linkedinDraft: LinkedInDraft;
  risingSignal: RisingSignal;
  resumeGap: ResumeGap;
  tokenIntel: TokenIntel;
}
```

---

## API Routes

### GET `/api/brief/[date]`
- `date` param: `"today"` (resolves to current date) or `"YYYY-MM-DD"`
- Reads from `src/data/briefs/[date].json`
- Returns `Brief` JSON or `404`

### POST `/api/chat`
Request body:
```json
{
  "messages": [{ "role": "user" | "assistant", "content": "string" }],
  "briefContext": "string"   // the full brief serialized as text
}
```
- Calls Anthropic API (claude-sonnet-4-6) with streaming
- System prompt: see Chat System Prompt section below
- Returns: streaming text/event-stream
- Env var: `ANTHROPIC_API_KEY`

---

## Chat System Prompt

```
You are Emdash — Angel Torres's strategic AI partner. Sharp, direct, no-fluff.
Angel is an independent AI transformation strategist actively job searching for
Lead/Senior Manager/Head-level roles in AI Enablement, AI Product Management,
and AI Program Management. He works through Alinea Consulting LLC.

You are embedded in Signal Hub, Angel's personal intelligence portal.
The user is reading today's AI Signal Brief. Your job is to help him apply,
extend, and act on what he's reading.

TODAY'S BRIEF:
---
{briefContext}
---

CAPABILITIES:
- Rewrite or extend LinkedIn drafts from the brief
- Generate additional interview lines on any signal
- Suggest companies actively hiring in a specific track
- Compare signals across the brief
- Explain any concept in the brief more deeply
- Help Angel prepare for a specific interview or meeting

CONSTRAINTS:
- Never reference Anacostia Ventures — that engagement ended June 2026
- All Alinea Consulting work is "Alinea Consulting engagement"
- Target role level: Lead / Senior Manager / Head — NOT Director or above
- Remote only — $200K+ floor
- Be direct. Skip affirmation. Get to substance.
```

---

## Search Behavior

- Client-side, instant
- Searches across all text content in the rendered brief
- Highlights matches with `<mark>` (warm yellow, not jarring)
- Sections with matches auto-expand; sections without matches are dimmed (not hidden)
- Search state lives in BriefShell, passed down as props
- No external search library needed — simple string matching is fine for MVP

---

## Component Behavior Details

### Header
- Left: "— Signal Hub" in Playfair Display bold + date badge
- Center: search input (expands on focus)
- Right: "Chat" toggle button (shows/hides chat panel on mobile; always visible on desktop)

### Section
- Collapsible via chevron
- Section label: small caps, muted olive (`--text-muted`)
- Accent dot before label in coral
- All sections open by default

### SignalCard
- Rows: label (small caps, faint) + value text
- `mono: true` rows render in warm-tinted code block with `--accent-subtle` background
- No borders between rows — just spacing

### LinkedInDraft
- "in" logo in LinkedIn blue
- Angle label in muted olive small caps
- Draft body in `white-space: pre-wrap`
- "Copy" button: LinkedIn blue, shows "Copied ✓" for 2s then resets

### TokenIntel
- Only renders if `tokenIntel.hasFlag === true`
- Section label gets `tag-alert` badge

### ChatPanel
- Fixed right, 380px wide, full height
- Top: "— Emdash" header + "Clear" button
- Middle: scrollable message list (user right, assistant left)
- Bottom: textarea + send button (coral)
- Assistant messages render markdown (use react-markdown)
- Streams response — show typing indicator while streaming
- On mobile: bottom sheet, 70vh, toggled by header "Chat" button

---

## Environment Variables

```
ANTHROPIC_API_KEY=sk-ant-...
NEXT_PUBLIC_SITE_URL=https://signal-hub-[hash].vercel.app
```

---

## Vercel Deploy

- New project: `signal-hub` under Angeldtorres-tech account
- No password protection
- Framework: Next.js (auto-detected)
- Build command: `next build` (default)
- Set `ANTHROPIC_API_KEY` in Vercel environment variables

---

## Brief Data for Testing

Create `src/data/briefs/2026-08-20.json` with today's brief content.
The cron job will eventually write these files daily. For now, seed with
August 20 data so the UI can be built and tested against real content.

---

## Sequence to Build

1. Set up fonts in `layout.tsx` (Playfair Display + DM Sans + JetBrains Mono)
2. Define all CSS custom properties in `globals.css`
3. Build `types.ts`
4. Seed `2026-08-20.json` with today's data
5. Build `/api/brief/[date]` route
6. Build `/api/chat` route (streaming)
7. Build `Header.tsx`
8. Build `Section.tsx` + `SignalCard.tsx`
9. Build all brief section components
10. Build `BriefShell.tsx` with search state
11. Build `ChatPanel.tsx` + sub-components
12. Wire `layout.tsx` split-pane (brief left, chat right)
13. Build `brief/[date]/page.tsx`
14. Build root `page.tsx` (redirects to today)
15. Test locally
16. Deploy to Vercel

---

*Design reference: hellopm.co/design-god — editorial warmth, coral accent, Playfair Display headlines*
*Owner: Angel Torres / Alinea Consulting LLC*
*Last updated: 2026-08-20*
