# Enterprise Knowledge Assistant

A small, well-explained Python project that builds the **most common LLM application in industry**: a RAG (Retrieval-Augmented Generation) system over your own documents.

> Upload PDFs / Word docs / text files. Ask questions. Get cited answers. No hallucination.

This project doubles as a teaching artifact — a beginner can read every file end-to-end and understand applied LLMs.

---

## What's in this project

```
.
├── README.md                ← you are here
├── TEACH_ME.md              ← entry-level guide. START HERE if you're new to RAG.
├── SKILL.md                 ← Agent Skill manifest — works in Claude Code too
├── colors_and_type.css      ← design tokens (colors, type, spacing, shadows)
│
├── app/                     ← THE PYTHON CODE (this is the real deliverable)
│   ├── main.py              ← FastAPI server: /upload, /ask
│   ├── ingest.py            ← file → chunks → embeddings → FAISS
│   ├── rag.py               ← question → search → LLM → cited answer
│   ├── config.py            ← settings loaded from .env
│   ├── requirements.txt
│   ├── .env.example
│   └── run.sh               ← one-shot: venv, install, start server
│
├── ui_kits/
│   └── assistant/           ← high-fidelity HTML mock of the chat UI
│       ├── index.html       ← interactive prototype
│       └── *.jsx            ← React components for the mock
│
└── preview/                 ← design-system cards rendered for the review pane
    ├── colors-*.html
    ├── type-*.html
    ├── spacing-*.html
    ├── components-*.html
    └── brand-*.html
```

**If you only read one thing, read `TEACH_ME.md`.**
**If you want to run the app, see `app/run.sh`.**

---

## Sources & origin

This project was scaffolded from a single user prompt that described an Enterprise Knowledge Assistant inspired by what Google / Microsoft / Amazon ship internally. **No existing codebase, Figma, or brand assets were provided.** Everything visual here is invented from first principles to fit a "serious enterprise tooling" tone — restrained, ink-on-paper, single accent. If you have a real brand, swap the tokens in `colors_and_type.css` and the look will follow.

---

## Content fundamentals (voice & tone)

The voice is **calm, technical, and direct.** It treats the reader as an intelligent peer and never patronizes.

| | Do | Don't |
|---|---|---|
| Pronoun | "you" — speak to the reader directly | "the user", "we" (except in tutorials) |
| Casing | Sentence case for everything: headings, buttons, menus | Title Case Marketing Speak |
| Jargon | Use the real word (embedding, chunk, FAISS) and define it once | Pretend the field is simpler than it is |
| Length | Short sentences. Plain words. | "Leverage", "synergize", "robust solution" |
| Emoji | Never in product UI. Sparingly OK in `TEACH_ME.md` to mark sections (used at most 1× per heading). | 🚀✨💡 sprinkled as decoration |
| Tone toward errors | Honest: "I don't know based on the provided documents." | "Oops! Something went wrong 😅" |
| Numbers | Always specific: "800-character chunks with 120-character overlap" | "small chunks with some overlap" |

**Examples of voice in this project:**

- *Empty state copy:* "No documents yet. Upload a PDF, Word doc, or text file to get started."
- *Error copy:* "Couldn't read that file. Supported types: .pdf, .docx, .txt, .md."
- *LLM refusal copy (in `rag.py`):* "I don't know based on the provided documents."
- *Section header in TEACH_ME:* "The seven things students get wrong"

---

## Visual foundations

The design language is **"trusted internal tool"** — close to a Bloomberg / Linear / GitHub Enterprise feel, but warmer.

### Colors
- **Ink** (`--ink-50` → `--ink-950`): a blue-tinted neutral. Every neutral in the UI comes from this scale. It is *almost* gray but never sterile.
- **Amber 500** (`#D8911E`) is the **single accent**. Used for primary buttons, focus rings, the active state of the composer. Never used decoratively.
- **Teal 500** (`#14857F`) is a quiet supporting color reserved for **"verified / source"** affordances — citation badges, the green checkmark on a successfully indexed document.
- Three semantic state colors (`--success`, `--warning`, `--danger`) and that's it. **No purple. No pink. No bluish-purple gradient.**

### Type
- **IBM Plex Sans** for all UI and prose. Free on Google Fonts; signals "engineering tool" without being cold.
- **IBM Plex Serif** for display moments — the empty-state hero, big quotes in slides. Used rarely.
- **IBM Plex Mono** for code, file names, IDs, and snippet previews of retrieved chunks.
- Tight letter-spacing on headings (`-0.02em`), normal on body. Caps are reserved for `t-eyebrow` (12px, +0.08em).

### Spacing & layout
- **4px grid.** All spacing tokens are multiples of 4. No 5s, 7s, 13s.
- Generous vertical rhythm — body line-height `1.55`, paragraphs separated by `--space-6` (24px).
- Layouts use a **two-pane shell**: persistent left rail for navigation/sources, single content column on the right. No three-pane layouts.
- Content is bounded by `max-width: 760px` for prose, `max-width: 1200px` for app shells.

### Backgrounds
- **No gradients.** Surfaces are flat ink-50 (canvas) or paper-white.
- **No background imagery.** Photographic content is reserved for the empty-state hero (one tasteful 16:9 photo of paper documents in soft daylight, if used).
- **No textures, patterns, or noise.**

### Borders & shadows
- Borders are `1px solid var(--border-subtle)` (ink-200) by default. Stronger borders (`ink-300`) only when separation needs to read at a glance.
- Three shadow levels — `--shadow-1` for cards, `--shadow-2` for popovers, `--shadow-3` for modals. All are **ink-tinted** (`rgba(11,18,32, …)`), never neutral gray.
- The focus ring is a 3px amber halo: `--shadow-focus`.

### Corner radii
- `--r-md` (8px) for buttons, inputs, cards.
- `--r-lg` (12px) for the chat composer and modal containers.
- `--r-pill` for tags, badges, citation chips only.
- Never mix radii within a single component.

### Animation
- **Restrained.** All transitions use `--ease-standard` (`cubic-bezier(0.2, 0, 0, 1)`) and one of three durations (120 / 180 / 280ms).
- Hover states fade `background-color` and `border-color`, never scale.
- Press states **darken the background by one step** (e.g. `--accent` → `--accent-press`). They do **not** shrink the element.
- Loading states use a fixed three-dot pulser, never a spinner ring.

### Transparency & blur
- Used **only** for the modal scrim (`--bg-overlay`, 55% ink-950) and the sticky header backdrop blur on scroll. Never on cards, buttons, or panels.

### Cards
A canonical card:
```css
background: var(--bg-surface);
border: 1px solid var(--border-subtle);
border-radius: var(--r-md);
box-shadow: var(--shadow-1);
padding: var(--space-6);
```
Cards never have colored left borders. Cards never have hover-lift transforms.

### Iconography
See **ICONOGRAPHY** below.

---

## Iconography

This project uses **Lucide** (https://lucide.dev) for all UI icons — a free, open-source icon set with consistent 1.5px strokes, rounded line-caps, and a 24×24 grid. It's loaded from the CDN in HTML mocks:

```html
<script src="https://unpkg.com/lucide@latest"></script>
```

**Why Lucide:**
- Stroke-based (matches our restrained, line-art aesthetic)
- 1500+ icons, covering everything we need for an enterprise tool
- Free fork of Feather Icons, actively maintained

**Rules:**
- Default size: **18px** in body, **20px** in headers, **16px** in dense UI (table cells, inline chips).
- Stroke color = `currentColor`. Never paint icons in a brand color directly; let them inherit.
- **Never** use emoji as iconography.
- **Never** use unicode dingbats (✓ ✗ ✦) as iconography.
- For the few brand-specific marks (the assistant's logo, the "verified source" mark), inline SVG lives in `assets/`.

**Substitution flagged:** Lucide is the design-system default since no in-house icon set was provided. If you have one, replace the `<i data-lucide="...">` tags and update this section.

---

## Index — what's where

| File | Purpose |
|---|---|
| `TEACH_ME.md` | Beginner walkthrough of RAG and every line of the Python code. |
| `app/` | The actual Python app (FastAPI + LangChain + FAISS). |
| `colors_and_type.css` | All design tokens. Import into any HTML to inherit the look. |
| `ui_kits/assistant/index.html` | High-fidelity interactive mock of the chat UI. |
| `preview/*.html` | Individual design-system cards (rendered in the Design System tab). |
| `SKILL.md` | Makes this project usable as a portable Agent Skill. |

---

## Caveats (read me)

- **No real brand was provided.** Colors, type, and components are an opinionated invention. Easy to retheme — change `colors_and_type.css` and import the new tokens.
- **Fonts are loaded from Google Fonts CDN**, not bundled as `.ttf`. If you need offline / locked-down deployment, download IBM Plex from https://www.ibm.com/plex/ and place the files in `fonts/`, then swap the `@import` in `colors_and_type.css` for `@font-face` declarations.
- **Icons are Lucide via CDN**, not bundled. Same fix if you need offline.
- The Python app **costs money to run** end-to-end (LLM API calls). Embeddings are free (local model). A typical question costs less than $0.001 with Claude Haiku.
