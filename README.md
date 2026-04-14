# Groundwork

**A plain-language guide to the Massachusetts childcare subsidy system.**

Live at [machildcareassistance.vercel.app](https://machildcareassistance.vercel.app)

---

## Why This Exists

Massachusetts has a childcare subsidy program that helps low-income families pay for childcare. The program is managed by the Department of Early Education and Care (EEC) and administered through a network of regional agencies called CCR&Rs (Child Care Resource and Referral agencies). It's a lifeline for thousands of families.

The problem is that the rules governing this program are spread across 5,000+ lines of regulations, policy guides, and advisory memos. Caseworkers at CCR&Rs spend years absorbing this knowledge through training, experience, and hallway conversation. Families and childcare providers — the people the system is built to serve — have almost no way to access it on their own.

When a parent gets a letter saying their voucher is being terminated, they don't know they have 14 days to file an appeal that keeps their childcare running while the appeal is reviewed. When a provider has a question about attendance reporting, they can't look it up anywhere. When a family's income changes, they don't know what they're required to report versus what they can choose to report. The information exists, but it's locked inside a system that was never designed to be read by the people it affects most.

This project exists to change that.

## Who Built This

This project was created by a former caseworker at Childcare Choices of Boston, one of the CCR&Rs that administers the subsidy program in the Metro Boston area. After years of answering the same questions, watching families lose benefits they were entitled to because they didn't know the rules, and seeing how much institutional knowledge lived only in the heads of experienced staff, the idea became clear: this information needs to be written down in plain language and made available to everyone.

This is not a replacement for caseworkers or the agencies that serve families. It's a tool that helps both sides. When families arrive at their CCR&R already understanding the basics — what documents to bring, what "service need" means, what happens at reauthorization — the conversation can start further along. The caseworker spends less time explaining the system and more time helping the family navigate it.

## What It Is

**A wiki** — 39 articles across three sections: *The System* (foundational concepts), *For Families* (the full family lifecycle from application through appeals), and *For Providers* (the provider-facing side: voucher agreements, reimbursement, attendance, compliance). Written in plain English, organized by the journey a user actually takes through the system.

**A chatbot** — an AI assistant embedded in the site that can answer questions about the subsidy system. It pulls its answers directly from the wiki content, so what the chatbot says and what the wiki says are always the same. It's designed to be warm, direct, and honest about the limits of what it can help with.

**An annotation layer** — behind the wiki content, there's a separate layer of practitioner notes drawn from real caseworker experience. These capture what actually happens versus what the regulations say: common mistakes families make, things caseworkers wish families knew, timing that matters in practice. The chatbot can access these notes when a question calls for practical guidance, not just policy.

**Search** — a full-text search across all 39 articles, running entirely in the browser with no server calls. Searches the article summaries, ranks results by title match, surfaces the relevant section for each result.

## What It Is Not

This tool does not determine eligibility, submit applications, or give legal advice. It does not replace a caseworker, an attorney, or any part of the existing system. Every answer points the user toward the human resource that can actually help — their CCR&R, Mass 211, legal aid, or the appropriate agency.

It's a ladder, not an elevator. It helps people understand where they are, what's next, and who to call.

## How It's Built

The wiki articles are written in plain markdown and compiled into a static website. They load fast, work on cheap phones, and don't require any account or login. The site is designed to be distributed via QR code at CCR&R offices, WIC clinics, family shelters, and other places where families already go.

The chatbot uses structured lookup — not a general-purpose AI that guesses at answers. When a user asks a question, the system identifies which wiki articles are relevant, retrieves them, and composes a response grounded in that specific content. Every answer is traceable back to a source article. If the system doesn't have an answer, it says so and redirects the user to a human.

The practitioner annotations are kept separate from the policy content by design. The wiki can be audited against official EEC documents at any time. The annotations add a layer of real-world guidance without compromising the accuracy of the policy layer.

## Who This Helps

**Families** navigating the subsidy system — applying for the first time, managing an active voucher, dealing with a termination or denial, or just trying to understand what a letter from their CCR&R means.

**Childcare providers** trying to understand how the subsidy system works from their side — voucher agreements, attendance rules, payment timelines, and what to do when a family's authorization is about to expire.

**CCR&R agencies and caseworkers** who field the same questions hundreds of times a year. When families arrive with a baseline understanding of the system, the intake process is faster and the conversations are more productive. This tool doesn't replace the caseworker — it gives them a head start.

**Community organizations** — legal aid offices, family shelters, immigrant services, DTA offices, and anyone else who encounters families with childcare questions outside the CCR&R system. Instead of saying "call your CCR&R," they can point to a resource that explains the basics first.

## The Bigger Picture

Government programs are often well-intentioned and poorly explained. The people who need them most — low-income families, people with limited English, people without reliable internet — are the least likely to understand how they work. The information exists, but it's written for administrators, not for the public.

This project is an attempt to bridge that gap for one specific program in one specific state. But the approach — translating dense policy into plain-language content, layering practitioner expertise on top, and making it accessible through both a browsable site and a conversational interface — is not specific to childcare subsidies. Any government program with complex rules and a vulnerable user base could benefit from the same treatment.

## For Developers

If you're interested in how this is built — the content architecture, the retrieval design, the decision to use structured lookup over embeddings, or the static site pipeline — see [ARCHITECTURE.md](./ARCHITECTURE.md).

If you're interested in the collaborative process used to build it — how we maintain coherence across AI sessions, how documentation functions as cross-session memory, and how this differs from "vibe coding" — see [METHODOLOGY.md](./METHODOLOGY.md).

---

Built by [Tutti Labs](https://tuttilabs.com).

This is a public repository for a private project.

---

*This is an informational resource, not an official government service. For eligibility determinations, application assistance, or legal advice, contact your local CCR&R or call Mass 211.*
