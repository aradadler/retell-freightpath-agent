# Design Document — FreightPath Voice Agent

**Project:** FreightPath Voice Agent (Retell AI)
**Version:** v1
**Author:** Arad Adler
**Last updated:** April 2026

---

## Problem Statement

Freight brokers operate in a high-communication, low-margin business. A mid-size brokerage moving 200–500 loads per month fields dozens of inbound calls daily — customers checking on shipments, asking about invoices, or inquiring about onboarding. Most of these calls are repetitive and resolvable without a human, but staffing a phone team to handle them at scale is expensive and doesn't improve as the business grows.

At the same time, inbound leads — prospects calling after seeing an ad or visiting the website — are high-intent and time-sensitive. A prospect who calls and hits voicemail or a long hold queue is very likely to call a competitor next. Routing every inbound lead to a human SDR isn't scalable, and chatbots have low adoption on inbound phone traffic.

Voice AI addresses both problems: it handles repetitive support volume without adding headcount, and it captures inbound sales interest instantly with a qualified, conversational agent — 24/7, at scale.

---

## Agent Architecture: Why Two Agents?

The most common question in designing this system: why build two separate agents instead of one agent that handles both support and sales?

**The case for a single agent** is operational simplicity — one phone number, one prompt to maintain, fewer routing decisions. It's attractive in theory.

**The case for two agents** is stronger in practice:

| Dimension | Support Agent (Alex) | Sales Agent (Jordan) |
|-----------|---------------------|---------------------|
| Caller intent | Known customer, specific problem | Unknown prospect, exploration mode |
| Tone | Efficient, reassuring, solution-focused | Curious, consultative, forward-looking |
| Success metric | Issue resolved or escalated cleanly | Demo booked |
| Risk of failure | Customer frustration, churn signal | Missed pipeline, poor first impression |
| Failure mode to avoid | Over-promising, under-escalating | Hard-selling, wasting unqualified time |

These two agents need fundamentally different conversation logic, tone calibration, and escalation triggers. A single prompt that tries to serve both purposes ends up doing neither well — it either sounds like a salesperson to a frustrated customer, or like a support rep to an excited prospect. Separation also enables independent iteration: you can test changes to the sales agent without touching the support agent, and vice versa.

In v1, the two agents live on separate phone numbers. A more sophisticated routing layer (IVR, intent detection at call start) is a v2 consideration.

---

## Prompt Design Philosophy

### Task sequencing over branching

Both agents use a linear task flow with labeled steps, rather than a multi-node conversation graph (Retell's "conversation flow" mode). This was a deliberate choice for v1.

**Why:** Single-prompt agents with explicit step ordering are easier to reason about, debug, and modify. When a call goes wrong, you can identify exactly which step broke down by reading the transcript. Conversation graphs are powerful for highly structured flows (IVR trees, form collection) but add significant maintenance overhead for nuanced, conversational interactions.

The tradeoff: a single prompt can be harder to enforce strict step gating (e.g., ensuring the agent always collects account info before looking up a shipment). This is mitigated with explicit ordering instructions ("Follow these steps in order. Do not skip steps.") and will be addressed in v2 if transcript analysis reveals step-skipping as a failure mode.

### One question at a time

Both prompts include an explicit "one question at a time" guardrail. This is one of the highest-leverage instructions for voice AI. In text-based interfaces, users can read a list of questions and answer them in sequence. On a phone call, stacked questions cause confusion — callers don't know which to answer first, often answer only the last one, and feel interrogated rather than helped.

### Escalation by default for anything requiring action

The support agent (Alex) is designed to triage, not resolve anything that requires system access, financial decisions, or carrier coordination. The escalation threshold is intentionally low in v1. The cost of a false escalation (a human rep gets a call they didn't need to take) is much lower than the cost of an agent attempting to resolve something it can't and leaving the customer with no answer.

### Soft guardrails on AI disclosure

Both agents avoid proactively announcing they are AI, but answer honestly if asked. This is a deliberate middle position — not deceptive, but not unnecessarily friction-inducing. Many callers have a strong preconception that AI = bad experience and will disengage before giving the agent a chance. The right moment to disclose is when asked, not as an opening disclaimer.

---

## Key Tradeoffs

**Single prompt vs. conversation flow agent**
Chosen: single prompt. Reason: lower maintenance, easier debugging in v1. Revisit if transcript analysis shows consistent step-ordering failures.

**Liberal escalation vs. high agent resolution rate**
Chosen: liberal escalation for support. Reason: trust-building is more important than deflection rate in v1. A support agent that escalates 60% of calls but gets it right every time is better than one that resolves 80% but occasionally leaves customers with wrong information.

**BANT-full vs. BANT-lite for sales qualification**
Chosen: BANT-lite. Reason: aggressive budget and authority probing in a first inbound call damages rapport. The primary qualification signal in v1 is need + timeline. Budget and authority are surfaced softly and passed to the AE.

**Demo booking via agent vs. human SDR handoff**
Chosen: agent books demo slot directly. Reason: conversion drops sharply when there is a gap between interest and commitment. "Someone will reach out" is a promise that often doesn't land. A confirmed calendar slot with a specific time has much higher show rates.

---

## What Was Intentionally Left Out of v1

**Live CRM/TMS integration:** Both agents operate without live account lookup in v1. The support agent acknowledges account information and proceeds; it doesn't actually query a database. This was excluded because it requires building and securing a Retell AI webhook integration, which would double the scope of v1 without changing the design or prompt logic being demonstrated.

**IVR / call routing layer:** Callers self-select into support or sales via separate phone numbers. A real deployment would benefit from a routing layer that detects intent and routes accordingly. Excluded from v1 because routing logic is infrastructure, not agent design.

**Post-call webhooks and CRM sync:** In production, every call should post structured data (issue type, escalation flag, callback number, lead qualification score) to a CRM via Retell's post-call webhook. Excluded from v1 as an integration concern rather than a design concern.

**Multi-language support:** FreightPath's fictional customer base is English-speaking in v1. Spanish-language support is the most obvious v2 addition for a realistic brokerage.

**Voicemail handling:** What happens when the agent reaches a voicemail? Not designed for v1. Retell AI has some native handling here, but the prompt logic doesn't account for it explicitly.
