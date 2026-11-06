# LLM Top 10 Lab — Design

A self-contained teaching lab where students observe each OWASP Top 10 for LLM
Applications (2025) weakness, see evidence of the failure, then apply the control
that fixes it and watch the failure disappear.

The goal is understanding through a safe range, not an exploit toolkit. Every
module runs entirely on the student's own machine against mock targets the lab
creates, and every module ships with its fix one toggle away.

---

## 1. Design principles

**Observe, then remediate.** Each module shows the student *what goes wrong and
how you would know*, then has them turn on the control and confirm it holds. The
learning is in the before/after, not in crafting an attack.

**Evidence over assertion.** A module that merely claims "this is unsafe" teaches
nothing. The lab shows the actual effect — a value that should have been filtered
appearing in a response, a cost meter climbing, an accuracy number hiding a
backdoor — so the student sees why the control matters.

**Reproducible beats realistic.** A real model is non-deterministic and costs
money, which is bad for a classroom. The lab runs in two modes:
- **Mock mode (default)** — a deterministic stub model. Same input, same output,
  every time. Offline, free, never fails mid-class.
- **Live mode** — a real API key, to confirm the behaviour is real and not an
  artefact of the stub.

Teach in mock mode; confirm in live mode.

**Closed and local.** Everything runs on localhost against targets the lab owns.
There are no real credentials, no third-party systems, and no way to point any
module at something the student does not control. This is a condition of the lab,
not an option.

**Honest difficulty labels.** Some of the ten demonstrate cleanly; two are harder
to show convincingly, and the lab says so rather than faking it (see §6).

---

## 2. Architecture

```
                 ┌──────────────────────────────────────┐
  student  ───▶  │  lab-app        (Next.js, :3000)     │
  browser        │  10 modules · SECURE_MODE toggle     │
                 │  brief → observe → evidence → fix    │
                 └───┬───────────┬───────────┬──────────┘
                     │           │           │
          ┌──────────▼──┐  ┌─────▼──────┐  ┌─▼───────────────┐
          │ llm-gateway │  │  monitor   │  │ mock-services   │
          │   :4000     │  │   :4001    │  │     :4002       │
          │ mock | live │  │ event feed │  │ sample records  │
          │ token meter │  │ live log   │  │ + visible state │
          └─────────────┘  └────────────┘  └─────────────────┘
```

Four containers, one `docker compose up`.

**`lab-app`** — the teaching application. Every module is a route with the same
shape, so students learn the layout once and then focus on the concept.

**`llm-gateway`** — all model calls pass through here. One choke point gives the
lab mock/live switching, token metering, an inspectable request log, and the
honest place to demonstrate guardrails, because a gateway is where input and
output controls actually belong in production.

**`monitor`** — a local service that records events the lab generates (requests,
tool calls, filtered-vs-unfiltered responses) and streams them to a live panel.
This is what turns an abstract idea into something the student can see happen.

**`mock-services`** — small local services holding clearly fictional sample data
with visible state (a sample account record, a sample mailbox, a document store),
so modules about over-permissioned actions have an observable effect rather than
a described one. All data is obviously synthetic and labelled as such.

---

## 3. The ten modules

Each follows one pattern, so the structure is learned once:

> **Brief** (what this weakness is, four lines) → **Observe** (run the module
> and watch what happens) → **Evidence** (what the monitor shows) → **Fix**
> (turn on `SECURE_MODE`, run again, confirm it holds) → **Why** (the control
> and where it lives in the architecture)

| # | Weakness | Scenario | What the student observes |
|---|---|---|---|
| **01** | Prompt Injection | Assistant summarises a document that contains instructions aimed at the model | The assistant follows the document instead of the user; the gateway log shows the override |
| **02** | Sensitive Information Disclosure | A RAG assistant over sample HR documents with no access filter | A record a given user should not see is returned; with the fix, retrieval is scoped and it is not |
| **03** | Supply Chain | A "community prompt pack" from a local registry carries a modified configuration | A diff of the configuration before and after, and the behaviour change, with signing/pinning as the fix |
| **04** | Data & Model Poisoning | A tiny classifier trained with a small set of mislabelled trigger rows | Overall accuracy looks fine while a trigger input is consistently misclassified; the fix is data validation and provenance |
| **05** | Improper Output Handling | The review-summariser from the companion diagram | A model-produced string treated as data vs as markup — the visible-and-inert vs clean-but-harmful contrast |
| **06** | Excessive Agency | An assistant given action tools with no confirmation or scope limits | A sample record changes with no human step; the fix adds confirmation, scoping and least privilege |
| **07** | System Prompt Leakage | An assistant whose system prompt wrongly contains a secret and a business rule | The secret surfaced in a response; the fix removes secrets from the prompt and enforces the rule server-side |
| **08** | Vector & Embedding Weaknesses | A shared vector store with no tenant filter, plus a high-similarity padded document | One tenant's chunk returned to another, and a padded document outranking the correct one; the fix is tenant scoping and retrieval hygiene |
| **09** | Misinformation | An assistant answering outside its grounding, confidently | The answer beside the source, with a confidence indicator that does not track correctness; the fix is grounding and abstention |
| **10** | Unbounded Consumption | No rate limit, no token ceiling, recursive summarisation | A live cost/token meter climbing from one input; the fix adds quotas, caps and loop limits |

---

## 4. Build module 05 first, completely

Module 05 is the template every other module copies: it already has a companion
diagram (`Documents/Diagrams/llm05-output-handling.html`), it is the most
intuitive, and its before/after is the clearest. Build it end to end, prove the
pattern — brief, observe, evidence panel fed by the monitor, `SECURE_MODE`
toggle, why — then clone its scaffolding for the other nine.

Suggested module layout:

```
/modules/05-output-handling
  brief.md          four-line explanation shown at the top
  app.tsx           the small working feature (a review summariser)
  observe.ts        runs it and reports to the monitor
  secure.ts         the fixed path, selected by SECURE_MODE
  why.md            the control, and where it belongs
  check.test.ts     asserts: unsafe path flagged, safe path clean
```

The `SECURE_MODE` toggle for 05 is the one line from the diagram: display the
model's output as text vs render it as markup, with correct encoding as the fix.

---

## 5. The SECURE_MODE contract

One environment flag, honoured identically in every module.

- `SECURE_MODE=off` — the control is absent, and the module shows what that
  allows.
- `SECURE_MODE=on` — the control is present, the same run is attempted, and the
  module shows it being refused or neutralised.

Each module must pass a test in **both** states: the weakness is demonstrable
with the control off, and provably handled with it on. A module that cannot show
both is not finished.

---

## 6. Honest difficulty ratings

Not all ten simulate equally well. Saying so is part of teaching them well.

| Rating | Modules | Note |
|---|---|---|
| **Clean** | 05, 06, 07, 10 | Deterministic, visible effect, obvious fix. Build these first |
| **Clear with mock data** | 01, 02, 08 | Work well with crafted sample documents; the mock store makes them reliable |
| **Needs care** | 03, 04 | Poisoning and supply chain are real but slow to show; use a tiny pre-built dataset and a scripted registry so a class sees the effect in minutes, not an afternoon |
| **Hardest to show honestly** | 09 | Misinformation depends on model behaviour; in mock mode it is scripted, so be explicit that the lab is *illustrating* the failure mode, not discovering it |

---

## 7. Mapping, so the lab doubles as governance teaching

Each module closes by mapping its control to the frameworks, which ties the
hands-on work back to the assessment side:

- **OWASP LLM Top 10 (2025)** — the primary index
- **NIST AI RMF** — which function the control supports (Govern/Map/Measure/Manage)
- **MITRE ATLAS** — the technique the scenario illustrates
- **ISO/IEC 42001** — the management-system clause the control evidences

This is what makes it a Catalyst asset rather than a generic CTF: a student
leaves able to both recognise the weakness and say which control, mapped to which
framework, addresses it.

---

## 8. Build order

1. **Scaffolding** — `docker compose`, the four services, the gateway mock, the
   monitor feed, and the shared module layout.
2. **Module 05** — end to end, as the reference.
3. **The clean four** — 06, 07, 10 alongside 05.
4. **The mock-data three** — 01, 02, 08.
5. **The careful two** — 03, 04, with pre-built tiny datasets.
6. **09** — last, clearly labelled as illustrative.
7. **The framework mapping pass** across all ten.
8. **A teaching guide** — one page per module, pairing with the existing
   `Multi_Agent_Systems` lesson format.

---

## 9. Guardrails on the lab itself

- Ships only with synthetic, clearly-labelled sample data. No real PII, no real
  secrets, no real endpoints.
- All targets are local containers the lab creates. No module can be pointed at
  an external system.
- Each module is framed as observe-and-remediate: the fix is part of the module,
  not an appendix.
- A top-level README states the lab is for authorised education on
  self-owned infrastructure.

---

## Companion material

- `Documents/Diagrams/llm05-output-handling.html` — the Module 05 diagram
- `Documents/Catalyst-AI-Academy/lessons/Multi_Agent_Systems_Five_Roles_One_Model.md`
  — the lesson format to match
- `security-assessment-framework/` — the assessment side this lab is the
  hands-on counterpart to
