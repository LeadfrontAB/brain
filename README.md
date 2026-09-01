---
tags: [llm-wiki, setup, guide, obsidian, claude-code, onboarding]
created: 2026-08-31
updated: 2026-08-31
sources: [wiki/llm-wiki.md, CLAUDE.md, hands-on setup of this vault]
---

# LLM-wiki — first-time setup on a new Mac (by the book)

A step-by-step for setting up an LLM-wiki from scratch: the folder layout, Claude Code as the
maintainer, Obsidian as the reader, and the schema that makes Claude link pages into a knowledge
graph. See [[llm-wiki]] for the underlying pattern.

## The one thing to understand first

Three layers, and the graph is simpler than you think:

1. **raw/** — your source files. Immutable. Claude reads them, never edits them.
2. **wiki/** — the markdown pages Claude writes. All the `[[links]]` live here.
3. **the schema (CLAUDE.md)** — the rulebook that turns Claude into a disciplined wiki maintainer.

**The knowledge graph is painted by Obsidian from the `[[wikilinks]]` in the wiki pages. It is not
built from embeddings and there is no parsing or indexing step.** When Claude writes
`[[client-tss]]` into a page, Obsidian draws an edge. That is the whole mechanism. Semantic search
(embeddings) is a separate, optional thing (see the end), and it is not what draws the graph.

---

## Answers to the three questions

**1. Should Claude work only in wiki/, or the whole vault?**
The whole vault root, which sees both `raw/` and `wiki/`. Claude must **read raw** (to ingest) and
**write wiki**. If you point it at `wiki/` alone it cannot see the sources. The discipline "read
raw, never edit it; write only wiki" is enforced by the schema (CLAUDE.md), not by hiding the
folder. So: launch Claude Code at the root.

**2. Open only raw/ as the Obsidian vault, or both?**
Open the **root** (both raw and wiki). If you open only `raw/`, the graph is nearly empty, because
the `[[links]]` live in `wiki/`, not in your source files. Obsidian can only draw edges it can see.
And to be clear: the graph shows up as soon as the links exist. There is no embedding or parsing to
run first.

**3. What is the plugin that draws the graph?**
It is not a community plugin. It is Obsidian's built-in **Graph view** (a core plugin, on by
default). The ribbon icon on the left opens the global graph. **Local graph** shows one note and its
neighbours. Turn them on under Settings, Core plugins.

---

## Folder setup

```
~/llm-wiki/            <- open THIS in both Claude Code and Obsidian
├── CLAUDE.md          <- the schema (the "skill")
├── raw/               <- drop your sources here (immutable)
│   └── assets/        <- images and attachments
└── wiki/              <- Claude writes here; index.md + log.md live here
```

Create it:
```
mkdir -p ~/llm-wiki/raw/assets ~/llm-wiki/wiki
```

---

## Claude Code setup

1. Install Claude Code (see claude.com/claude-code).
2. Launch it at the **root**, so it sees raw and wiki together:
   ```
   cd ~/llm-wiki && claude
   ```
3. Create `CLAUDE.md` at the root. This is the schema, and it is what does the linking and parsing
   into the graph. A minimal by-the-book version:

```markdown
# LLM-wiki schema

## Structure
- `raw/` — immutable source files. Never modify or delete.
- `raw/assets/` — images and attachments.
- `wiki/` — all LLM-maintained markdown pages.

## Conventions
- Page names: lowercase, hyphenated (e.g. `client-acme.md`).
- Every page starts with YAML frontmatter: `tags`, `created`, `updated`, `sources`.
- Cross-reference related pages with `[[page-name]]`. Link generously — the links ARE the graph.
- `wiki/index.md` — catalog of all pages, updated on every ingest.
- `wiki/log.md` — append-only record of every ingest and query.

## Bootstrap (first ingest only)
If `wiki/index.md` does not exist, create it (a table of all pages with one-line
descriptions) and create `wiki/log.md` (a header only) before writing any page.

## Workflows
### Ingest
1. Read the source in `raw/`.
2. Extract the entities, decisions and facts worth keeping.
3. Write or update the relevant wiki pages, adding `[[links]]` to related pages.
4. Update `index.md` and append to `log.md`.

### Query
1. Read `index.md` to find relevant pages, read them, synthesise an answer.
2. If the answer has lasting value, file it as a new wiki page and link it.

### Lint
- Flag contradictions between pages, orphan pages (no inbound links), and stale claims.

## Rule
Read from `raw/`, write only to `wiki/`. Never edit a source file.
```

4. First run: drop a file into `raw/`, then tell Claude: **"Ingest raw/<file> into the wiki."** It
   reads the source, writes a page with `[[links]]`, and updates the index and log. Ingest one
   source at a time and read what it wrote.

---

## Obsidian setup

1. Install Obsidian.
2. "Open folder as vault" and choose the **root** `~/llm-wiki` (not `raw/`).
3. Settings, Core plugins: make sure **Graph view** is on (it is by default).
4. Click the graph icon in the left ribbon. As Claude adds `[[links]]`, the graph fills in live.
5. Optional: in the graph filter, hide the sources with `-path:raw` so you see only the wiki, or
   keep them to see which pages came from which sources.

That is it. The graph is live: no build, no embedding, no export.

---

## Best practice, in short

- **One home, two roles.** Open the same root in Claude Code and Obsidian. Obsidian is the reader,
  Claude is the writer, the wiki is the artifact.
- **Sources are sacred.** Everything goes into `raw/` untouched. Claude only writes `wiki/`.
- **Links are the graph.** The value is in the `[[cross-links]]`, so tell Claude to link generously.
  A page with no inbound links is an orphan and should be linked from the index or a parent page.
- **Ingest one source at a time and stay involved**, at least early on. Read the summaries, correct
  the emphasis, and let the schema improve as you learn what works.
- **Keep index.md and log.md current.** The index is how Claude finds pages at query time; the log
  is the history.

## Optional — semantic search (a different thing)
If you want "find related by meaning" rather than "find related by link", add a community plugin
such as Smart Connections, which builds embeddings over your notes. That is genuinely embedding-based
and separate from the Graph view. You do not need it for the knowledge graph.
