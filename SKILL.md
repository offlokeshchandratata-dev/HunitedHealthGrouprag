---
name: knowledge-design
description: Use this skill to generate well-branded interfaces and assets for the Enterprise Knowledge Assistant ("Knowledge"), either for production or throwaway prototypes/mocks. Contains essential design guidelines, colors, type, fonts, assets, and UI kit components for prototyping a RAG-based document Q&A product.
user-invocable: true
---

Read the README.md file within this skill, and explore the other available files.

Key files:
- `README.md` — brand voice, visual foundations, iconography
- `TEACH_ME.md` — beginner walkthrough of the underlying RAG architecture (useful when designing UI for indexing / retrieval / source-citation flows)
- `colors_and_type.css` — drop-in design tokens (colors, type, spacing, radii, shadows, motion)
- `ui_kits/assistant/` — high-fidelity React mock; copy components from here as a starting point
- `preview/` — individual swatches/specimens for every token
- `app/` — the actual Python RAG codebase, useful for grounding any UI in real data shapes

If creating visual artifacts (slides, mocks, throwaway prototypes), copy `colors_and_type.css` and the relevant `ui_kits/assistant/*.jsx` components into the target project, then build static HTML files for the user to view. Lucide icons load from CDN — keep them.

If working on production code, lift the tokens from `colors_and_type.css` and the component patterns from `ui_kits/assistant/` to extend the brand. Match the voice rules in the README's CONTENT FUNDAMENTALS section.

If the user invokes this skill without other guidance, ask them what they want to build (slide deck, marketing page, in-app screen, blog post hero, etc.), ask 3–5 focused questions about audience and surface, then act as an expert designer who outputs HTML artifacts or production code, depending on the need. Default to one option first; offer variations only when asked.
