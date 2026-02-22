# SpecObserver

A tool that turns complex markdown spec documents into an interactive, navigable React UI.

Built for **spec-driven development** workflows where agents work from detailed markdown plans. Instead of scrolling through raw markdown, SpecObserver generates a self-contained React artifact with collapsible sections, a navigation sidebar, search, Mermaid diagram rendering, and progress tracking.

## The Problem

When doing spec-driven development with AI agents, the spec document is the source of truth — but it's also a wall of text. As specs grow with phases, tasks, diagrams, and checklists, they become hard to navigate. You lose track of where you are, what's done, and how the pieces connect.

## What SpecObserver Does

You ask the agent to visualize your spec, and it generates a single `.jsx` artifact:

- **Collapsible sections** — h2 headings start expanded, deeper levels are collapsed by default
- **Table of Contents sidebar** — dark sidebar with the full heading hierarchy, click to navigate, scroll-tracked active state
- **Search** — `Cmd+K` to search across all sections, highlights matches in content and filters the ToC
- **Mermaid diagrams** — fenced `mermaid` blocks render as SVG diagrams inline
- **Code blocks** — syntax-highlighted with a copy button
- **Tables** — styled with alternating rows and sticky headers
- **Checklists** — visual checkboxes with progress counters ("3/7 complete")
- **Frontmatter** — YAML metadata rendered as a header card with status badges
- **Multi-file support** — tabs to switch between multiple spec documents
- **Expand/Collapse all** — quick toggle for deep specs

## Installation

This is a skill package for AI coding agents. Drop the `spec-observer/` folder into your skills directory:

```
your-project/
├── .claude/
│   └── skills/
│       └── spec-observer/
│           ├── SKILL.md
│           ├── references/
│           │   ├── component-architecture.md
│           │   └── markdown-parser.md
│           └── examples/
│               └── example-spec.md
```

## Usage

Ask the agent:

> "Visualize this spec for me"
> "Render my spec files as a browsable UI"
> "This plan is too long — can you make it easier to navigate?"

The agent reads your markdown files, parses the structure, and generates a React artifact using the SpecObserver component pattern.

## Skill Structure

```
spec-observer/
├── SKILL.md                              # Core skill instructions
├── references/
│   ├── component-architecture.md         # React component blueprint (JSX template)
│   └── markdown-parser.md                # Markdown → JSON parsing algorithm
├── examples/
│   └── example-spec.md                   # Sample spec for testing
├── evals/
│   └── evals.json                        # Test prompts for skill evaluation
└── README.md
```

## Supported Markdown Features

| Feature | Syntax | Rendering |
|---------|--------|-----------|
| Headings | `## Title` | Collapsible sections with hierarchy |
| Mermaid | ` ```mermaid ` | SVG diagrams via CDN mermaid.js |
| Code blocks | ` ```language ` | Styled block with copy button |
| Tables | `\| col \| col \|` | Alternating-row HTML table |
| Checklists | `- [ ] / - [x]` | Visual checkboxes + progress |
| Frontmatter | `--- yaml ---` | Metadata card with status badge |
| Bold | `**text**` | Inline bold |
| Italic | `*text*` | Inline italic |
| Links | `[text](url)` | Clickable links |
| Inline code | `` `code` `` | Styled inline code |

## Design Decisions

- **No npm dependencies at runtime** — the generated artifact only uses React, Tailwind utility classes, and lucide-react (all standard in modern React sandboxes)
- **Mermaid loaded lazily from CDN** — only fetched when a mermaid block exists
- **Manual markdown parsing** — no external libraries; regex-based parsing that handles the structural elements specs actually use
- **Dark sidebar / light content** — high contrast for the navigation, easy on the eyes for reading

## License

MIT
