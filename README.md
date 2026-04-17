# FreightPath Voice Agent — Built on Retell AI

FreightPath is a fictional B2B SaaS company providing a transportation management system (TMS) for freight brokers and shippers. This project is a fully designed voice AI demo built on [Retell AI](https://www.retellai.com), featuring two production-quality inbound phone agents: one for customer support, one for sales. Both are backed by Claude as the LLM. The goal was to take a deliberately unglamorous, operationally complex domain and build something that would actually hold up in production, then document every decision the same way I would for a real deployment.

---

## Why I Built This

Retell AI's pitch is that entire contact centers can be replaced by AI workers: not just automated, but actually intelligent. I wanted to test that claim as a builder, not just evaluate it as an observer. Freight logistics is a deliberate choice. It has high call volume, operationally complex edge cases, and customers who pick up the phone instead of opening a ticket. If a voice agent can handle a freight broker's inbound support and sales calls well — angry customers, delivery exceptions, live qualification — it can handle most B2B use cases. This project is the result of building both agents end to end: designing the prompts, stress-testing the edge cases, and writing up the product decisions the same way I would for a real deployment. `PRODUCT_BRIEF.md` documents four gaps I found in the platform along the way, written for a Retell AI PM audience.

---

## What's Inside

```
retell-freightpath-agent/
├── README.md                    # This file
├── agents/
│   ├── inbound-support.md       # Full prompt + design for Alex, the support agent
│   └── inbound-sales.md         # Full prompt + design for Jordan, the sales agent
├── docs/
│   ├── DESIGN.md                # Architecture decisions, tradeoffs, and v1 scope rationale
│   ├── TESTING.md               # Test scenario matrix and transcript review guide
│   └── PRODUCT_BRIEF.md         # PM brief: gaps in the Retell platform + proposed features
└── assets/
    └── transcripts/             # Live call transcripts added after testing
```


| Path                        | What it is                                                                                                                        |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `agents/inbound-support.md` | Complete single-prompt agent for existing customers: handles shipment status, delivery exceptions, billing triage, and escalation |
| `agents/inbound-sales.md`   | Complete single-prompt agent for inbound leads: qualifies prospects and books 30-min demos with AEs                               |
| `docs/DESIGN.md`            | The reasoning behind every major design decision, including what was intentionally left out of v1                                 |
| `docs/TESTING.md`           | Scenario matrix (10 cases per agent), simulation test guide, and transcript review checklist                                      |
| `docs/PRODUCT_BRIEF.md`     | Product feedback written for a Retell AI PM audience: gaps I found while building, with prioritized feature proposals             |


---

## Try It Live

> Call the **support line** and tell Alex you have a delivery exception on shipment #48291.
> Call the **sales line** and tell Jordan you're running a 300-load-a-month brokerage on spreadsheets.
>
> **Support line:** Demo number available upon request — contact me directly.
> **Sales line:** Demo number available upon request — contact me directly.

---

## Tech Stack


| Layer                | Tool                                                    |
| -------------------- | ------------------------------------------------------- |
| Voice agent platform | [Retell AI](https://www.retellai.com)                   |
| LLM                  | Claude (Anthropic)                                      |
| Voice & telephony    | Retell AI hosted telephony                              |
| Prompt design        | Single-prompt agent architecture                        |
| Agent architecture   | Single-prompt (linear task flow, no conversation graph) |


---

## Demo

> FreightPath Voice Agent Architecture
> *Two-agent architecture: inbound support (Alex) and inbound sales (Jordan), both powered by Retell AI and Claude.*

> **Sample transcript placeholder** — *See `assets/transcripts/` after live testing*

---

## Project Status


| Agent                  | Prompt written | Product brief written | Simulated in Retell | Live number assigned | Transcripts collected |
| ---------------------- | -------------- | --------------------- | ------------------- | -------------------- | --------------------- |
| Inbound Support (Alex) | Done           | Done                  | Pending             | Pending              | Pending               |
| Inbound Sales (Jordan) | Done           | Done                  | Pending             | Pending              | Pending               |


