# AI Security & Governance Lab

Three browser-based tools for assessing, governing and teaching the security of AI and
LLM systems. Every tool is a **single self-contained HTML file** — no build step, no
dependencies, no backend, no data leaves the browser.

Built by **Tosin Aina** — CISSP, CISA, CRISC, ISO/IEC 27001 Lead Auditor, ISACA AAISM.

---

## The three tools

### 1. LLM Top 10 Lab

A hands-on threat lab covering the **OWASP Top 10 for LLM Applications (2025)**.
Each risk is presented as a working exercise rather than a definition, with every
finding mapped to control anchors in:

- **OWASP LLM Top 10 (2025)** — LLM01 Prompt Injection through LLM10
- **NIST AI RMF** — GOVERN, MAP, MEASURE, MANAGE functions
- **ISO/IEC 42001** Annex A and **ISO/IEC 27001** Annex A
- **MITRE ATLAS** adversarial techniques

Two risks ship as expanded standalone modules:

| Module | Covers |
|---|---|
| `module-01-prompt-injection.html` | LLM01 — why a prompt is not a trust boundary |
| `module-05-output-handling.html` | LLM05 — insecure output handling and downstream injection |

### 2. AI Project Risk Advisor

Walks an AI initiative through a structured risk interview and returns a populated
risk position — not an awareness score. Maps answers against ISO/IEC 42001,
ISO/IEC 27005, the NIST AI RMF and the **EU AI Act**, including Article 15 and the
Article 27 fundamental-rights impact assessment.

### 3. AI Risk Lifecycle Navigator

Takes a risk from identification through treatment decision and residual position,
following the ISO/IEC 42001 lifecycle. Designed to turn lab findings into entries
a GRC register will actually accept.

---

## Running it

Open any `.html` file in a browser. That's the whole install.

```
git clone https://github.com/Tozino22/ai-security-governance-lab.git
cd ai-security-governance-lab
start llm-top10-lab/llm-top10-lab.html     # Windows
open  llm-top10-lab/llm-top10-lab.html     # macOS
```

Or use the hosted version via GitHub Pages — see the repository's Pages link.

---

## A note on the payloads

The lab contains deliberately malformed tokens, injection strings and a dummy JWT
(`eyJhbGciOiJIUzI1NiJ9.FAKE`). These are **teaching artifacts**. They are inert,
contain no real credentials, and exist to demonstrate what insecure output handling
looks like in practice.

---

## Why these exist

Most AI governance training stops at definitions. These tools were built to close the
gap between *naming* a risk and *recording a treatment decision against it* — the step
where assurance usually breaks.

## Licence

MIT.
