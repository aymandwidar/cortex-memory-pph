# Cortex Memory

## Pitch Script & Executive Q&A — v4

**Hackathon Track:** Prompt 03 — Drive Growth with Agentic AI
**Builder:** Ayman Dwidar

---

## Part 1: The 2-Minute Pitch Script

### THE HOOK (THE PAIN)

"Hi everyone, I'm Ayman. Over the last year, I've built around 50 to 70 different AI-first applications. Recently, I spent five hours waiting on a coding agent because it kept trying to publish to a blocked port. 47 retries of the same failing command. Roughly $3.20 in Bedrock tokens, by my back-of-the-envelope. Zero learning between attempts. The agent didn't know the port was blocked. It didn't remember it had already tried and failed."

### THE PROBLEM (RETURN ON TOKENS)

"This is what our enterprise customers are facing at scale: **Single-Session Amnesia**. Think about a customer rolling out coding agents to 1,000 developers. If every agent starts from zero and burns tokens rediscovering the same deployment traps, the financial waste is staggering. We aren't maximizing our **Return on Tokens**; we are paying for the exact same LLM inference repeatedly. It is a massive, invisible compute leak."

### THE SOLUTION (NETWORK EFFECTS, NOT BIGGER CONTEXT WINDOWS)

"To solve this, I built Cortex Memory, an enterprise-grade MCP memory server. We can keep increasing context windows and the size of foundational models, but real-world efficiency is the key to scaling. Cortex Memory turns one developer's solution into 99 colleagues' starting point. Instead of brute-forcing large context windows, it proactively intercepts an agent's intentions, searches a vector store of past reasoning, and injects the exact proven solution before the agent executes a bad command."

### THE DIFFERENTIATION

"What makes Cortex different from notes files, rules files, or generic episodic memory like Mem0 or Letta? Cortex stores the reasoning trajectory — including the wrong turns. The Reasoning DAG captures HYPOTHESIS → DEAD_END → SOLUTION as typed nodes with causal edges. That negative knowledge — the paths that didn't work — is exactly what foundation models systematically underlearn from training data biased toward successful outcomes."

### THE OPEN-SOURCE CONNECTION

"To ensure the highest quality extraction, I built this on top of **CMM (Cognitive Memory Manager)**, an MIT-0 licensed open-source project. CMM provided the hardened extraction kernel. I then engineered the proactive MCP interceptor, the Personal Mesh sync over Tailscale, the drift management infrastructure (preconditions, supersession, feedback loops), and the executive dashboard to turn it into a secure, enterprise-ready product."

### THE ROI (RETURN ON INTELLIGENCE)

"The results are measurable. Our controlled A/B test on a complex architectural bug — the exact type of critical-path decision where agents typically burn tokens guessing — Cortex Memory reduced session size by **75%** and cut files written by **71%**. At 100 developers, on conservative unit economics that count only direct Bedrock inference savings, that's roughly **$53.5K saved per year** before counting developer time. Cortex Memory turns AI spend from a sunk cost into a compounding asset. Every solved problem protects your team from paying to solve it twice, maximizing your **Return on Intelligence**."

> **Important:** Be sure to mention that these efficiency gains apply to critical-path debugging sessions (architectural bugs, infra debugging, auth flows), where token burn concentrates — not to boilerplate tasks. The deck and technical doc both make this scope explicit.

---

## Part 2: Anticipated SA Tech Leadership Q&A

### Q: Doesn't running Sonnet 4.5 and Opus 4.7 to distill these memories cost more Bedrock tokens than you are saving?

**A:** The distillation process is asynchronous and runs exactly once per solved problem. If Opus 4.7 spends 5,000 tokens to distill a complex auth strategy, that strategy is then cached in ChromaDB. When a team of 100 developers hits that same auth issue over the next year, Cortex Memory injects the solution locally via semantic search — costing zero Bedrock tokens. We trade a one-time distillation cost (~$0.075 per strategy) for a permanent, team-wide reduction in inference waste. Breakeven is a single reuse.

### Q: If we inject past memories into every session, won't we eventually bloat the agent's context window with irrelevant data?

**A:** That is exactly why we use Just-In-Time (JIT) interception rather than static prompt injection. Cortex Memory doesn't dump the whole team manual into the system prompt. It waits until the agent explicitly states an intention (like "I am adjusting the pool_size") or hits an error. Only then does it query the vector database and inject that specific, highly-targeted solution. Furthermore, the precondition filter runs *before* semantic ranking — a Next.js 13 strategy will not surface in a Next.js 15 codebase regardless of cosine similarity, so the retrieved set is already context-pruned before it reaches the agent.

### Q: How do you handle memory drift — when a strategy that worked three months ago is now outdated because we updated our framework?

**A:** Drift is solved structurally, not manually. Three mechanisms:

**1. Strategies carry preconditions, not just content.** Every strategy node records the stack version, file glob patterns, and problem signature it applies to. Retrieval filters by current context BEFORE semantic ranking — a Next.js 13 strategy will not surface in a Next.js 15 codebase even on perfect cosine similarity.

**2. Supersession edges in the DAG.** When a new strategy resolves the same problem signature as an older one, the distillation pipeline writes an explicit `supersedes` edge. Retrieval automatically prefers the latest non-superseded node along any supersession chain. No manual archiving required.

**3. Usage feedback loop.** When a strategy is injected and the session subsequently fails or is abandoned, that signal decays the strategy's confidence. Strategies that keep failing in fresh contexts decay below the retrieval threshold; strategies that keep succeeding get reinforced. Darwin runs on the store instead of relying on humans to notice.

The Tech Lead dashboard handles outliers and audit, not the critical path. Day-to-day drift management is automatic.

### Q: What about sensitive code, secrets, and PII? Your example nodes reference DATABASE_URL strings and internal hostnames. How do you prevent that from leaking into the vector store?

**A:** Three layers of redaction:

**1. Capture-time secret filters.** The session capture layer runs a regex pass against known secret patterns (API keys, JWTs, AWS access keys, connection strings, private keys) before any text reaches the extraction pipeline. Matches are replaced with typed placeholders (e.g., `<REDACTED:DATABASE_URL>`) which preserve semantic meaning for the LLM without leaking values.

**2. Customer-controlled allowlists.** Enterprises configure additional patterns (internal hostnames, employee emails, project codenames) via a YAML policy file mounted into the MCP server. Policies are versioned and audit-logged.

**3. Embedding scope discipline.** Only distilled strategy text is embedded into ChromaDB. Raw investigation evidence — file contents, command outputs, stderr dumps — stays on the local machine in the warm-tier capture layer and is never embedded or synced to the team store.

ChromaDB runs inside the customer's VPC. The only external call is the asynchronous distillation to Bedrock, which operates on already-redacted text under standard enterprise data privacy terms. Inputs are not used for model training. Every redaction event is audit-logged for compliance review.

### Q: How accurate is the 75% reduction figure? Is it cherry-picked?

**A:** Honest answer: the 75% is the upper end of the range we measured on critical-path debugging sessions. Across our A/B test cases, session-size reduction ranged from 60% to 75%, with the largest gains on tasks that involve repeated dead ends (auth flows, infrastructure scaling, deployment traps). We make this scope explicit in the deck and technical doc rather than burying it in a footnote. Boilerplate tasks see negligible benefit and are out of scope. The system is designed to win where token burn concentrates, not everywhere.

### Q: Our customers are highly sensitive about sending proprietary code to third-party databases. How is this secured?

**A:** The beauty of the MCP architecture is that the memory store (ChromaDB) is hosted entirely within the customer's secure perimeter, VPC, or even locally on the developer's machine. The code never leaves their AWS environment. We only use Bedrock for the asynchronous distillation, which operates on already-redacted text and is covered under standard enterprise data privacy agreements where inputs are not used for model training. See the security Q&A above for the redaction layers in detail.

### Q: You mentioned this was built on an open-source project. How much of this is net-new, and why didn't you just build it from scratch?

**A:** I evaluate projects using the "Vitamin vs. Painkiller" framework. The open-source project I built on is **CMM (Cognitive Memory Manager)**, MIT-0 licensed. CMM provided a hardened "Vitamin" — a research-quality extraction pipeline I would have spent the entire hackathon reinventing.

I used that open-source kernel to build the "Painkiller". The net-new architecture I built:

- **Proactive MCP server** with three interception tools (`get_architectural_context`, `search_past_dead_ends`, `log_new_discovery`)
- **Strategic Snapshot distillation pipeline** using Claude Opus 4.7 via Amazon Bedrock
- **Multi-machine Personal Mesh sync** over Tailscale (Ubuntu server, MacBook M4, Windows Omen)
- **Push / review / pull governance workflow** with project vs. team scope classification
- **Drift management infrastructure**: preconditions, supersession edges, usage feedback loop
- **Security & redaction layer**: capture-time secret filters, customer-controlled allowlists, embedding scope discipline
- **Liquid Glass Tech Lead dashboard** with live injection feed and decay watch
- **Bedrock-native AWS integration** with Isengard credential vending

Standing on the open-source extraction kernel let me deliver a complete enterprise integration in 3 days instead of a research prototype.

### Q: How does Cortex compare to existing memory approaches like Cursor rules, CLAUDE.md, or episodic memory libraries like Mem0 and Letta?

**A:** Three distinct categories, and Cortex is in the third:

**Notes & rules files (Cursor rules, CLAUDE.md, memory banks).** Static, hand-curated, human-authored. Captures preferences and conventions. Does not capture *why* a path failed, and compounds only with manual effort.

**Episodic memory (Mem0, Letta, Zep).** Stores conversation history. Optimized for chat assistants. Not designed around debugging trajectories, typed reasoning nodes, or negative knowledge.

**Cortex Reasoning DAG.** Typed nodes (HYPOTHESIS → DEAD_END → SOLUTION) with causal edges. Captures negative knowledge — the paths that didn't work — which is what LLMs systematically underlearn from training data biased toward successful outcomes. This is the structural differentiator, and it's why the gains concentrate on critical-path debugging rather than general assistance.

### Q: What's the latency overhead of the interceptor on every agent action?

**A:** Measured on local ChromaDB with 381 nodes: p50 total interception overhead is ~45ms (15ms embedding + 30ms vector search), p99 is ~120ms. This is below the threshold of perceived latency for tool calls and is dwarfed by Bedrock inference time. The interceptor is strictly cheaper than the dead end it prevents. On larger stores (10K+ nodes), HNSW indexing keeps p99 under 200ms.

### Q: What happens if the agent gets "trapped" by past decisions and misses a superior modern approach?

**A:** This is a real risk and we've designed against it explicitly. Three mitigations:

**Strategies are suggestions, not directives.** The agent retains override authority. Cortex injects context; it does not constrain behavior.

**Confidence decay.** When a strategy is injected and the session subsequently struggles or fails, that signal decays the strategy's confidence. The system actively unlearns stale patterns.

**Explicit conflict surfacing.** When two strategies survive precondition filtering and neither supersedes the other, Cortex shows both options with provenance rather than silently picking one. Silent resolution is where bad memory systems gaslight their users.

The deck includes an explicit "What Cortex Doesn't Do" slide naming this risk among others. We'd rather a calibrated tool that earns trust than an over-promising one that erodes it.

---

*Cortex Memory — PPH Hackathon 2026 · Prompt 03*
*Built by Ayman Dwidar · Amazon Web Services*
*Built on top of CMM (Cognitive Memory Manager), MIT-0 licensed.*
