# Repository Copilot Instructions

## Repository Overview

**System-Design** is a documentation repository. Its only content today is `system-design-curriculum.md` (about 870 lines): a "Fresher to Senior" system-design curriculum of 24 modules and case studies. Every concept is layered into three tiers so growth can be tracked:

- **Fresher** (0-2 years): explain and implement with guidance.
- **Mid** (2-5 years): own end to end, with trade-offs, failure modes and when to reach for the pattern.
- **Senior/Staff** (5+ years): cross-cutting trade-offs, scale limits, org and cost implications, and knowing when *not* to use the pattern.

There is no application code, build system or CI.

## Technology Stack

- Markdown only. No code, dependencies, diagrams tooling or build files exist yet. Do not add tooling (static site generators, diagram compilers) unless asked.

## Repository Structure

```
system-design-curriculum.md   the whole curriculum
```

Modules in order: 1 Thinking & Requirements, 2 Scalability, 3 Database Fundamentals, 4 Database Scaling, 5 Caching, 6 Distributed Caching & CDN, 7 Messaging, 8 Event-Driven & CQRS, 9 API Architectures (REST/GraphQL/gRPC), 10 Real-Time & Async APIs, 11 Microservices, 12 Advanced Microservices, 13 Fault Tolerance, 14 Observability, 15 Security Architecture, 16 Compliance & Protection, 17 Cloud Architecture, 18 Containers & Kubernetes, 19 CI/CD & Platform Engineering, 20 Serverless/Edge/AI-Integrated, 21-22 URL Shortener (build and productionize), 23-24 Real-Time Chat (build and scale).

## Architecture (of the document)

Each module is a `## Module N: Title` heading. Each concept is a bold title line followed by exactly three bullets: `- Fresher:`, `- Mid:`, `- Senior/Staff:`. Case studies are written as concepts named `**Case Study: <name>**` with the same three tiers. Keep this shape; it is what makes the curriculum scannable and trackable.

## Development Commands

None. Preview Markdown in the editor. Useful checks:

```bash
grep -n "^## Module" system-design-curriculum.md     # verify module numbering and order
```

## Coding Guidelines (writing guidelines)

- Prioritize architecture reasoning: requirements, capacity estimation, bottlenecks, scalability, reliability, consistency trade-offs, observability, security, and cost.
- Each tier must be a distinct depth level: Fresher = define and apply; Mid = choose between options and name failure modes; Senior/Staff = decide, defend, and consider org/cost/long-term consequences.
- Be concrete: use numbers with units (p99 < 200ms, DAU x actions / 86400), name real technologies where helpful, and show the trade-off, not just the definition.
- State when *not* to use a pattern. Avoid buzzword lists without a decision rule.
- Diagrams, when added, should be text-based (Mermaid or ASCII inside fenced blocks) so they render in GitHub and diff cleanly; state the key flow and the bottleneck in words next to it.
- Keep terminology consistent across modules (for example "p99", "RPO/RTO", "idempotency key").

## Testing

No automated tests. Review changes by reading: renumbered modules, consistent three-tier structure, no contradicting statements across modules, valid Markdown, working relative links.

## Security

Public documentation; no secrets or proprietary employer information. Case studies must use generic or public-knowledge designs.

## Infrastructure / Deployment

None.

## Change Guidelines

1. Read the surrounding modules first so new content does not duplicate or contradict them.
2. Make the smallest coherent change; do not restructure the curriculum unless asked.
3. Preserve the module order and numbering (other notes may reference "Module N").
4. When adding a concept, add all three tiers.
5. Do not add tooling or new file formats without a reason.
6. Do not leave commented-out text.
7. Do not leave TODO placeholders unless explicitly requested.
8. Do not fabricate facts, statistics or benchmark numbers; if a number is an illustrative estimate, label it as such.
9. Re-read the edited section for accuracy and consistency before finishing.

## Code Quality Rules (content quality)

- Prefer clear, precise prose over clever phrasing; avoid duplication across modules.
- Follow the existing heading, bold-title and bullet conventions.
- Cover failure modes and edge cases (partial failure, hot partitions, thundering herd, retries).
- Preserve backward compatibility of anchors and headings other docs may link to.
- Avoid unrelated rewrites during focused edits.

## Git Commit Rules

- Never add a `Co-Authored-By` trailer unless I explicitly request it.
- Never add Claude, Anthropic, GitHub Copilot, OpenAI, ChatGPT, Codex, Cursor, or any AI tool as an author or co-author.
- Use only the configured Git `user.name` and `user.email`.
- Do not mention AI assistance in commit messages.
- Keep commit messages concise and professional.
- Do not commit automatically unless I explicitly ask.
- Do not push automatically unless I explicitly ask.
- Never force-push unless I explicitly request it.
- Never rewrite Git history unless I explicitly request it.

## AI Assistant Working Rules

When working in this repository:

- Inspect existing code before proposing architecture changes.
- Do not assume a feature exists without verifying it.
- Do not create fake implementations to make UI or tests appear complete.
- Do not generate random metrics, scores, or placeholder business data unless explicitly requested as test/demo data.
- Clearly separate verified behavior from assumptions.
- Prefer completing working vertical slices over creating many unfinished placeholders.
- Preserve repository conventions.
- Avoid massive rewrites unless explicitly requested.
- When fixing a bug, identify the underlying cause where practical.
- When adding functionality, consider error handling and tests.
- Never expose secrets, API keys, tokens, or credentials.
- Never hardcode secrets.

## Repository-Specific Rules (system-design content)

- Always tie a design choice to a requirement and a trade-off: scalability, reliability/availability, consistency (CAP/PACELC), latency, cost, operability and observability.
- Identify bottlenecks and single points of failure explicitly, and say how the design degrades under failure.
- Case studies (URL shortener, chat) should progress from a working baseline (Modules 21, 23) to a production-hardened design (Modules 22, 24); keep that split.
- Do not present an architecture as "the" answer; present the reasoning and the alternatives rejected.
- Real-world numbers (traffic, latency, capacity) are illustrative unless sourced; say so.
