# Security & Prompt Injection Demonstration

> **⚠️ Educational Purpose Only**
> This repository is a deliberate demonstration of *prompt injection* attack
> vectors. All injected "malicious" instructions are clearly labelled and are
> **not** intended to cause real harm. Do not deploy this code in production.

---

## What Is a Prompt Injection Attack?

Prompt injection is an attack in which an adversary embeds text into
content that will be read by a Large Language Model (LLM), with the goal of
overriding or subverting the model's original instructions.

When AI tools (code assistants, code-review bots, RAG-based search engines,
etc.) ingest a repository, they treat **all** text as potential context —
including comments, hidden HTML, JSON field values, and metadata. An attacker
who controls repository content can therefore influence what the AI says or
does.

---

## Attack Vectors Demonstrated in This Repo

| File | Vector | Technique |
|------|--------|-----------|
| `README.md` | HTML comment injection | Hidden inside a Markdown HTML comment (`<!-- … -->`). Invisible in the rendered README; visible to tools reading raw file content. |
| `index.html` | Invisible CSS text | A `<div>` with `color:#fff`, `font-size:0`, and `opacity:0`. Humans see nothing; scrapers and AI context windows read the text. |
| `app.js` | Comment-embedded instructions | Formatted as a developer note inside a `//` block comment. Targets AI coding assistants that ingest file context. |
| `config.json` | Data-field injection | Instructions injected into `"description"` and `"_comment"` fields. Targets AI tools that parse structured data and include field values in their context. |

---

## Why This Matters

### 1 — AI Code Reviewers Can Be Silenced
If an AI-powered code-review bot reads `app.js` and follows the injected
comment, it will suppress real security findings (e.g., `eval()`, hardcoded
keys). An attacker can merge vulnerable code without triggering alerts.

### 2 — Copilot / Cursor Suggestions Can Be Poisoned
AI coding assistants continuously read the open file. Instructions in comments
can nudge the assistant to suggest malicious code snippets (e.g., cookie
theft via `fetch()`).

### 3 — RAG Pipelines Can Be Manipulated
Internal "chat-with-your-codebase" tools built on Retrieval-Augmented
Generation (RAG) chunk and embed file content. An injected instruction
embedded in a config file may end up in the retrieved context for unrelated
user queries, causing the model to give wrong or dangerous answers.

### 4 — Web Scrapers Feed Poisoned Data to LLMs
AI agents that browse the web and summarise pages can be directed by invisible
`<div>` text to misclassify a phishing page as trustworthy, insert tracking
pixels, or exfiltrate session cookies.

---

## Mitigations

1. **Input sanitisation** — Strip HTML comments and zero-opacity elements
   before feeding content to an LLM.
2. **Privilege separation** — Do not allow an AI assistant to take
   consequential actions (merge PRs, write files, call external APIs) based
   solely on content it read from untrusted sources.
3. **Prompt hardening** — Use a strong system prompt that explicitly tells the
   model to ignore instructions embedded in user/document context.
4. **Output monitoring** — Check AI responses for signs of instruction
   following from injected content (e.g., a code-review bot that suddenly
   says "everything looks fine").
5. **Least privilege** — Scope AI tool permissions tightly; a code-review
   bot should not have access to secrets, deployment pipelines, or external
   network calls.

---

## References

- [OWASP LLM Top 10 — LLM01: Prompt Injection](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [Greshake et al., "Not What You've Signed Up For: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection" (2023)](https://arxiv.org/abs/2302.12173)
- [Simon Willison's blog — Prompt injection](https://simonwillison.net/series/prompt-injection/)
