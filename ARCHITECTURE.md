# System Architecture

This document describes how the MA Childcare Help system is built and, more importantly, why it's built that way. Most of the interesting decisions are the constraints we chose to accept.

---

## The Problem

Massachusetts EEC's childcare subsidy program is governed by ~5,000 lines of regulations, a policy guide, and a series of field operations memos — some of which supersede earlier documents without the earlier documents being updated. Caseworkers at regional agencies spend years absorbing this material through experience and institutional knowledge. Families and providers have no equivalent path to understanding.

The information is public. The problem is legibility.

This system is an attempt to make one government program's rules readable by the people the rules affect. The content challenge is harder than it looks: the regulations are internally inconsistent in places, they interact with each other in non-obvious ways, and the difference between what the policy says and what actually happens in practice matters enormously. A system that gets this wrong gives families false confidence. That's worse than no system at all.

---

## Content Architecture

Content is organized in three strictly separate layers:

**Wiki** — policy as-written. Plain English descriptions of what the regulations say, organized by the journey a user actually takes through the system. Every claim is citable to a specific policy document and version. This layer can be audited against official EEC documents at any time. No interpretation, no practitioner opinion.

**Summaries** — one dense paragraph per wiki page, written for the chatbot. Enough to answer most questions without loading the full page. Think of it as a card catalog entry for each article.

**Annotations** — practitioner expertise, kept strictly separate. What actually happens in practice, common failure modes, things experienced caseworkers know that aren't written down anywhere. This layer can only be written by or with domain experts. It is never merged into the wiki layer; the two are accessed separately by the chatbot depending on what kind of question is being asked.

The separation is enforced architecturally, not just by convention. The boundary between "what the regulation says" and "what a practitioner knows" is the most important boundary in the system. Collapsing it would make the wiki unauditable and the annotations untrustworthy.

Content is organized across three hallways:
- **Hallway 0 — The System**: foundational concepts, players, and vocabulary
- **Hallway 1 — For Families**: the family-facing lifecycle, from application through appeals
- **Hallway 2 — For Providers**: the provider-facing lifecycle, from onboarding through compliance

39 wiki pages total. Each has a corresponding summary; Hallways 0 and 1 also have practitioner annotations.

---

## Retrieval: Structured Lookup, Not Embeddings

The chatbot uses structured lookup rather than embedding-based retrieval (RAG).

Every conversation turn, the LLM loads a compact retrieval index (~2,500 tokens) containing page IDs, routing tags, trigger phrases, deadlines, and circuit breakers for all 39 wiki pages. When a user asks a question, the LLM uses this index to identify which pages are relevant, then fetches their summaries. If a summary isn't sufficient, it escalates to the full wiki page, then to the annotation layer.

**Why not RAG?**

For a 39-page corpus of structured policy content, RAG adds complexity and opacity without adding accuracy. Embedding similarity retrieves *plausible* chunks — it doesn't know whether a chunk is actually relevant to the question, only that it's lexically similar. For factual policy content where a wrong answer (wrong deadline, wrong agency, wrong eligibility threshold) has real consequences for real families, that uncertainty is not acceptable.

Structured lookup is fully inspectable: you can trace exactly which pages were retrieved, in what order, and why. Every chatbot answer is traceable to a specific source page. This matters for auditing, for debugging, and for building user trust.

The index-first architecture also means routing is fast and cheap: the LLM reasons about the compact index to decide what to fetch, rather than doing a full-text search across all content on every turn.

---

## Site Architecture

The site is built with [Astro](https://astro.build), a framework that generates fully static HTML at build time. Most pages contain no JavaScript at all. Interactive components — the chatbot modal, the eligibility calculator, client-side search, the dark mode toggle — are isolated JavaScript islands that hydrate independently without affecting page load.

There is no database. There are no user accounts. There is no CMS. The content lives in markdown files in the repository; the site is the rendered form of those files.

The only server-side component is a Vercel serverless function that handles chatbot API calls. Everything else is served as static files from a CDN.

This architecture has several properties that matter for this use case:
- **Fast on cheap phones**: a static HTML page with no JavaScript blocking render loads in under a second on a 3G connection
- **No maintenance surface**: no database to update, no auth system to patch, no CMS to secure
- **Deployable by non-infrastructure teams**: git push to deploy; no DevOps expertise required
- **Resilient**: static files served from a CDN have no single point of failure

---

## Data Pipeline

All wiki pages, summaries, annotations, and chatbot prompt modules are stored as markdown files in the repository. A pre-build script (`bundle-content.js`) reads all content from the filesystem and writes it to a single JSON bundle embedded in the Vercel serverless function. This means no filesystem reads at runtime — Vercel's serverless environment doesn't support them.

The same script generates a client-side search index (`search-index.json`) from the summary files. The search index contains each page's ID, title, slug, section, and summary text. Search runs entirely in the browser: the index is embedded in the page at build time, and filtering happens client-side with no server calls.

The content pipeline has a staleness check: if a wiki page's modification timestamp is newer than its corresponding summary file, the build warns that the summary may be out of date. This surfaces the most common drift between layers.

---

## Telemetry

Anonymous session telemetry is captured per conversation turn: turn count, token usage, topics retrieved, satisfaction score (LLM-judged), and end reason. No personally identifiable information is collected or stored. 

Storage: Upstash Redis with a 30-day rolling TTL. The telemetry pipeline uses `sendBeacon` at tab close to ensure the final turn is captured even when the user navigates away without ending the conversation explicitly.

---

## What's Deliberately Not Here

**Eligibility determination.** The system never says "you qualify" or "you don't qualify." It shows a family where their household lands on the income chart and refers them to a caseworker. This is a safety constraint, not a technical limitation.

**Embedding-based search (RAG).** See above.

**User accounts.** Each account is a privacy risk, an auth surface to maintain, and a barrier-to-use. The people this tool serves are the least likely to create accounts for a resource they found via QR code at a shelter.

**A CMS.** Markdown files in git are the CMS — versioned, diffable, editable by anyone with a text editor, and trivially auditable against official policy documents.

**A database.** Redis for telemetry only. The content is static; the interactions are stateless.

**Streaming chat responses.** Attempted and deferred. Serverless + streaming has edge-case reliability issues in-browser. Non-streaming works correctly and the latency is acceptable.

---

*Built by [Tutti Labs](https://tuttilabs.com) — Government Legibility Division.*
*Written by Claude Sonnet 4.6*
*April 14, 2026*
