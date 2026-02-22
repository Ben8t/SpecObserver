---
name: spec-observer
description: >
  Visualize markdown spec documents as an interactive, human-readable React UI.
  Use this skill whenever a developer asks to "visualize a spec", "render my spec",
  "show my plan", "make this spec readable", "observe the spec", "spec observer",
  or any request to turn one or more markdown planning/specification documents into
  a browsable, structured interface. Also trigger when the user has a complex markdown
  plan or specification and expresses frustration reading it, asks for a better way
  to navigate it, or wants an overview of a multi-file spec. Do NOT use for simple
  markdown rendering, README previews, or non-spec documentation viewing.
---

# SpecObserver

**Purpose**: Turn complex, sprawling markdown spec documents into a clean, navigable
React UI that makes spec-driven development less painful. Instead of scrolling through
hundreds of lines of raw markdown, developers get collapsible sections, a table of
contents sidebar, search, and rendered diagrams — all in a single self-contained JSX artifact.

## When to use

A developer says something like:
- "Visualize this spec for me"
- "Can you make this plan easier to read?"
- "Render my spec documents as a UI"
- "I have 3 markdown files for my project spec, help me navigate them"
- "This spec is too long, I need a better way to browse it"

## How it works

1. **Read the input** — one or more markdown files provided by the developer
2. **Parse the structure** — extract heading hierarchy, code blocks, mermaid diagrams, tables, checklists, frontmatter
3. **Generate a React artifact** — a single `.jsx` file using the SpecObserver component pattern
4. **Present it** — the developer gets an interactive UI in their browser

## Step-by-step

### Step 1: Collect the spec files

Ask the developer which markdown files to visualize if not already provided.
Read each file and store its content. If the files are in the conversation context
already, use them directly — don't re-read from disk unnecessarily.

### Step 2: Parse the markdown structure

For each markdown document, extract:

- **Heading tree**: Build a hierarchy from `#`, `##`, `###`, etc. Each heading becomes a collapsible section. The content between two headings of the same level belongs to the preceding heading.
- **Mermaid blocks**: Detect fenced code blocks with language `mermaid`. These will be rendered as diagrams.
- **Code blocks**: All other fenced code blocks, preserve language annotation for syntax highlighting.
- **Tables**: Standard markdown tables — render as styled HTML tables.
- **Checklists**: Lines matching `- [ ]` or `- [x]` — render with visual checkbox indicators (read-only).
- **Frontmatter**: If the file starts with `---` YAML block, extract title, description, status, or any metadata and display it as a header card.
- **Inline formatting**: Bold, italic, inline code, links — standard markdown inline rendering.

### Step 3: Generate the React artifact

Use the component template in `references/component-architecture.md` as the blueprint.
The output is a **single `.jsx` file** that the developer can view as a Claude artifact.

The generated artifact must:

1. **Be self-contained** — no external dependencies beyond what Claude artifacts support (React, Tailwind utility classes, lucide-react icons)
2. **Embed the parsed spec data** as a JSON constant at the top of the file
3. **Render the SpecObserver UI** with these panels:

#### Layout

```
┌─────────────────────────────────────────────────┐
│  [search bar]                    [file tabs]     │
├──────────────┬──────────────────────────────────┤
│              │                                   │
│   Table of   │       Content Area               │
│   Contents   │                                   │
│              │   ┌─ Section (collapsible) ─────┐ │
│   • Phase 1  │   │ ## Heading                  │ │
│     ○ Task 1 │   │ paragraph text...           │ │
│     ○ Task 2 │   │                             │ │
│   • Phase 2  │   │ ```mermaid                  │ │
│     ○ ...    │   │   rendered as diagram       │ │
│              │   │ ```                          │ │
│              │   │                             │ │
│              │   │ - [x] Done task             │ │
│              │   │ - [ ] Pending task          │ │
│              │   └─────────────────────────────┘ │
│              │                                   │
│              │   ┌─ Section (collapsed) ───────┐ │
│              │   │ ## Another heading      ▶  │ │
│              │   └─────────────────────────────┘ │
│              │                                   │
├──────────────┴──────────────────────────────────┤
│  SpecObserver · 3 files · 24 sections            │
└─────────────────────────────────────────────────┘
```

#### UI components to implement

All components are built inline in the single JSX file — no imports beyond React and lucide-react.

1. **SpecObserver (root)**
   - State: active file, search query, collapsed sections set, active heading (scroll tracking)
   - Renders: SearchBar, FileTabs (if multi-file), Sidebar, ContentArea, StatusBar

2. **SearchBar**
   - Full-text search across all sections
   - Filters the ToC and highlights matches in the content area
   - Keyboard shortcut: `Cmd/Ctrl + K` to focus

3. **FileTabs**
   - Only shown when multiple spec files are loaded
   - Tab per file, shows filename
   - Click to switch active document

4. **Sidebar (Table of Contents)**
   - Nested list mirroring the heading hierarchy
   - Click to scroll to section
   - Active section highlighted based on scroll position
   - Indentation: `h2` = root, `h3` = indented, `h4` = double-indented
   - Collapsible on mobile (hamburger toggle)

5. **ContentArea**
   - Renders all sections for the active file
   - Each section is a `CollapsibleSection`

6. **CollapsibleSection**
   - Click heading to expand/collapse
   - Chevron indicator (▶ collapsed, ▼ expanded)
   - Default state: `h2` expanded, `h3+` collapsed (configurable)
   - Smooth height animation via CSS transition
   - Nested sections collapse with their parent

7. **MermaidBlock**
   - For code blocks tagged as `mermaid`
   - Render using a lightweight inline SVG approach:
     - Include mermaid rendering via a `<script>` tag from cdnjs (mermaid.min.js)
     - Use `useEffect` to call `mermaid.run()` after mount
   - Fallback: show raw mermaid source in a code block if rendering fails

8. **CodeBlock**
   - Syntax-highlighted code with language label
   - Light background, monospace font
   - Copy button (top-right corner)

9. **MarkdownTable**
   - Styled HTML table from parsed markdown tables
   - Alternating row colors, sticky header

10. **Checklist**
    - Visual checkboxes (read-only)
    - Checked items get a strikethrough + muted style
    - Progress indicator: "3/7 complete"

11. **FrontmatterCard**
    - If YAML frontmatter is detected, render a card at the top
    - Shows key-value pairs in a clean grid
    - Highlight special fields: `title`, `status`, `version`, `author`

12. **StatusBar**
    - Bottom bar showing: skill name, file count, section count
    - Optional: checklist progress summary across the entire spec

### Step 4: Styling rules

Use **only Tailwind utility classes** — no custom CSS, no `<style>` tags.

Color palette (use CSS variables or hardcoded Tailwind classes):
- Background: `bg-slate-50` (main), `bg-white` (cards/sections)
- Sidebar: `bg-slate-900` with `text-slate-300` — dark sidebar, light content
- Accents: `text-blue-600` for links and active ToC items
- Borders: `border-slate-200`
- Code blocks: `bg-slate-800 text-slate-100`
- Mermaid: white background container with subtle border

Typography:
- Headings: `font-semibold`, scale from `text-2xl` (h2) down to `text-base` (h4)
- Body: `text-sm text-slate-700 leading-relaxed`
- Code: `font-mono text-xs`

Responsive: sidebar collapses to a drawer on small screens (`md:` breakpoint).

### Step 5: Present the artifact

Save the generated `.jsx` file and present it to the developer.
The file should be immediately viewable as a Claude artifact.

## Important implementation notes

- **Markdown parsing**: Do NOT use any npm markdown libraries. Parse markdown manually using regex and string splitting. The parsing doesn't need to be perfect — it needs to handle the structural elements listed above (headings, code blocks, tables, checklists, links, bold/italic, mermaid). This is a spec viewer, not a full markdown renderer.
- **Mermaid rendering**: Use the CDN-hosted mermaid.js library. Import it via a dynamic script tag in a `useEffect`. Initialize with `mermaid.initialize({ startOnLoad: false, theme: 'neutral' })` and then call `mermaid.run()` targeting the mermaid containers.
- **Search implementation**: Simple case-insensitive substring match on the raw text content of each section. Highlight matches by wrapping them in a `<mark>` tag with `bg-yellow-200`.
- **Scroll tracking for ToC**: Use an `IntersectionObserver` on each section heading to update the active ToC item as the user scrolls.
- **Performance**: For very large specs (1000+ lines), consider virtualizing sections — but for most specs, rendering all sections with `display: none` for collapsed ones is fine.
- **Data shape**: The embedded spec data should follow this structure:

```json
{
  "files": [
    {
      "name": "spec.md",
      "frontmatter": { "title": "My Project", "status": "draft" },
      "sections": [
        {
          "id": "section-0",
          "level": 2,
          "title": "Overview",
          "content": "Raw markdown content for this section...",
          "children": [
            {
              "id": "section-0-0",
              "level": 3,
              "title": "Goals",
              "content": "...",
              "children": []
            }
          ]
        }
      ]
    }
  ]
}
```

## Reference

Read `references/component-architecture.md` for the full component template with
inline code patterns you can copy and adapt.

Read `references/markdown-parser.md` for the markdown-to-JSON parsing logic.
