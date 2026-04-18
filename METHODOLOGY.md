# Building with AI: Notes on a Working Method

This document describes how this system was built — not the what, but the how. It's a record of a working method that emerged over the course of this project, and that we think is worth describing because we haven't seen it written down elsewhere.

---

## Not Vibe Coding

"Vibe coding" — prompting an AI and accepting whatever it produces — is a useful mode for quick prototypes. It doesn't work for systems where correctness matters: where a wrong answer about a government deadline could cost a family their childcare, where an annotation that leaks into a policy layer makes the whole system unauditable, where an architectural decision made casually in session three breaks session twelve.

What follows is something more disciplined: a collaborative process in which the human is the architect, domain expert, and final decision-maker, and the AI is the planning, implementation, and documentation partner. The AI writes most of the code and most of the content. The human defines what gets built, reviews decisions before they're implemented, and contributes the practitioner knowledge that makes the content worth reading.


---

## The Core Problem: AI Has No Memory

Language models don't have persistent memory. Each conversation starts fresh. For a multi-week build with dozens of sessions, this is an obvious problem: how do you maintain architectural coherence when your implementation partner forgets everything each time?

Our approach: externalize everything that matters into documents, then load only what's relevant to the current session. The AI never needs to remember what happened in previous sessions because the relevant decisions are written down in a place it can read.

---

## Snapshots as Cross-Session Memory

At the end of each work session, we write a snapshot — a plain English briefing document that records what was decided, what was built, what was left open, and what comes next. Snapshots are a temporal chain-of-events that can be used as a map of all the decisions that were made and why, a map that can be looked at tomorrow, or ten years from now.

This is the core novel contribution of this methodology. It inverts the usual relationship between documentation and work: instead of writing documentation as a post-hoc record of what was built, you write it prospectively — as a brief for your collaborator's next shift.

Snapshots solve several problems at once:

- **Continuity**: the next session can be oriented in under a minute
- **Decision logging**: decisions are recorded with their reasoning, not just their outcome. Six sessions later, when someone asks "why did we do it this way?", the answer is in the snapshot.
- **Documentation as byproduct**: comprehensive documentation emerges naturally from the work process rather than as a separate effort after the fact
- **Inspectability**: the build history is human-readable; anyone can understand what was decided and why

Snapshots live in `documentation/snapshots/` with date-prefixed filenames and accumulate over time as a chronological record of the build.

---

## CLAUDE.md: Writing for a Future AI Instance

This project includes a file called `CLAUDE.md` in the repository root. It's loaded into every AI session automatically. It contains:

- The invariants of the system — decisions that cannot be changed without explicit discussion
- Working style and communication preferences
- The system architecture at a high level, with pointers to deeper documentation
- Single sources of truth for every major topic

The key insight behind this file is that it's written *for a future AI instance, not for the human*. This is an unusual thing to optimize for, and it changes how you write: more explicit statement of things that might seem obvious, more careful definition of terms, more deliberate reasoning about why constraints exist rather than just what they are.

A well-written `CLAUDE.md` means each new session starts with a collaborator who understands the system's rules without needing to be re-briefed. A poorly written one means each session spends the first twenty minutes re-establishing context.

---

## The Control Plane: One Question Per Document

`CLAUDE.md` is the first of seven root-level documents that together form a **control plane** for the project. Each answers exactly one durable question that a maintainer — human or AI — might ask on day one or on day one thousand:

| Document | The question it answers |
|----------|-------------------------|
| `CLAUDE.md` | What are the rules? |
| `BLUEPRINT.md` | How does this system work? |
| `PROCEDURE.md` | How do I build another one? |
| `OPERATIONS.md` | Something changed — what do I update? |
| `DIRECTORY.md` | Where does everything live? |
| `GOVERNANCE.md` | Who decides what? |
| `CHANGELOG.md` | What changed and when? |

The constraint is strict: one document per question, one question per document. If a new concern emerges that none of the seven answer, it gets its own document at the root, not a subsection inside an existing one. If two documents start to overlap, one of them is wrong.

The most important of the seven — and the one we had to discover the hard way — is `OPERATIONS.md`. It is a **routing table for change propagation**: when the homepage adds a new UI string, what else needs to change? When a wiki page gets a new source citation, what else? When an environment variable rotates? The document enumerates change types and, for each, the downstream artifacts that must be updated in the same session.

This is a structural answer to documentation drift, not a cultural one. "Keep docs in sync" is a wish. A named routing table that names every change type and its downstream effects is a mechanism. When we noticed a silent drift today — a tool had been added to the chatbot months ago and `CLAUDE.md` still didn't list it — the fix wasn't just updating `CLAUDE.md`; it was adding that category of change to `OPERATIONS.md` so the next instance catches itself.

The control plane sits above everything else in the repository. Operational documents (the `documentation/` tree) answer narrower, more time-bounded questions. The superdocs answer the ones that outlive any single session.

---

## Tiered Documentation

Under the control plane, operational documentation falls into three tiers. Not all documentation serves the same purpose; conflating them creates noise.

**Living documents** (`documentation/active/`): The current authoritative state of specific concerns — deferred-work lists, status-note indexes, staleness tracking. Updated in place; always reflect the present.

**Reference documents** (`documentation/reference/`): Stable lookup tables and specifications. The chatbot retrieval index. The wiki taxonomy. Page metadata with sources and citations. Consulted frequently, updated rarely.

**Snapshots** (`documentation/snapshots/`): Point-in-time records of sessions. Never edited after the fact. Accumulate as a log of the build history.

The discipline of asking "where does this information live?" before writing it down — rather than writing it in whatever document is currently open — prevents the drift where the same decision is recorded in three places and they gradually become inconsistent with each other. For control-plane questions the answer is always one of the seven root documents; for everything else, one of the three operational tiers.

---

## Architecture Before Code

Every significant decision in this project was made in conversation before being implemented in code. The three-layer content model (wiki, summaries, annotations) was designed and debated before any pages were written. The retrieval architecture (structured lookup over embeddings) was justified in prose before any retrieval code existed. The site architecture (Astro, no database, build-time bundling) was reasoned through before any components were built.

The format is: the human describes the problem, the AI proposes approaches with trade-offs, the human makes a decision, and then — and only then — the AI writes code.

This prevents a failure mode common in AI-assisted development: the AI implements something plausible-looking that doesn't fit the system architecture, because the architecture was never made explicit. When the AI knows what it's building toward, it makes better local decisions without requiring the human to review every line.

---

## Single Source of Truth

Every piece of information in this system lives in exactly one place. The chatbot and the wiki share the same content. Policy metadata lives in `WIKI_PAGE_METADATA.md` and nowhere else. The site's page registry lives in `wiki.ts` and nowhere else.

This is an engineering principle applied to AI collaboration. When the AI can find the authoritative answer to any question by reading the right document, it doesn't have to guess — and it doesn't hallucinate. When there are two sources of truth and they're inconsistent, the AI has to choose between them, and it will sometimes choose wrong.

Single source of truth is also a forcing function for documentation hygiene: you can't have three places where a thing is documented if you commit to one.

---

## What's Genuinely Novel Here

We want to be honest about what's new and what's just good engineering practice applied to a new context.

**Probably not new:** architecture-before-code, single source of truth, tiered documentation. These are standard software engineering disciplines.

**Probably new:** the snapshot-as-cross-session-memory pattern, specifically applied to AI collaboration. The practice of writing documentation *for a future AI instance* rather than for a human reader. The three-layer content architecture (policy / retrieval summary / practitioner annotation) for government legibility systems. The superdoc control plane — a fixed set of root-level documents each answering one durable question, with a named routing table (`OPERATIONS.md`) that enumerates change types and their downstream effects.

**Worth naming:** the distinction between this approach and "vibe coding" is not yet well-articulated in the literature. Most discussion of AI-assisted development assumes either full autonomy (the AI does everything) or pure acceleration (the AI autocompletes your code faster). The collaborative model — where the human architect and the AI implementer work together on an explicit plan with written documentation — is a third mode that produces different results.

---

*Built by [Tutti Labs](https://tuttilabs.com) — Government Legibility Division.*

*Written by: Warren Zhao and Claude Sonnet 4.6*

*April 14, 2026 — last updated April 17, 2026*
