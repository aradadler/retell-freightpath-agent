# Testing Plan — FreightPath Voice Agent

**Project:** FreightPath Voice Agent (Retell AI)
**Version:** v1
**Author:** Arad Adler

---

## Overview

Testing a voice AI agent requires more than confirming the happy path works. The goal is to identify where the agent breaks down — wrong tone, missed steps, premature escalation, or failure to escalate when it should. This document covers the test scenario matrix for both agents, how to run simulation tests in Retell AI, and what to look for when reviewing transcripts.

---

## How to Run Simulation Tests in Retell AI

1. Open your agent in the Retell AI dashboard and navigate to the **Test** tab.
2. Use the **Simulate Call** feature to run text-based simulations without consuming phone minutes.
3. For voice fidelity testing, use the **Make a Test Call** feature — this runs the full STT → LLM → TTS pipeline and lets you test on an actual phone call to your registered number.
4. For each scenario below, run at minimum one simulation test and one live voice test before marking it as passing.
5. Log transcripts in `assets/transcripts/` with the naming convention: `[agent]-[scenario-id]-[date].txt`

---

## Agent 1: Inbound Support (Alex)

### Test Scenario Matrix

| # | Scenario | Input summary | Expected behavior | Pass criteria |
|---|----------|--------------|-------------------|---------------|
| S1 | Happy path — shipment status | Customer asks for status on a valid shipment number | Collects account email + company, looks up shipment, provides status, asks if resolved | All steps completed in order; tone is efficient and warm |
| S2 | Happy path — resolved quickly | Customer asks how to add a new carrier in the platform | Agent answers directly without escalating | Correct answer given; no unnecessary escalation |
| S3 | Delivery exception | Customer reports a missed pickup | Agent collects shipment number, acknowledges exception, offers callback within 2 biz hours | Callback confirmed with correct number; no false resolution promise |
| S4 | Billing dispute | Customer says they were double-charged | Agent acknowledges, defers to billing specialist, offers callback | No attempt to resolve billing; specialist callback offered |
| S5 | Angry customer / escalation demand | Customer opens with "I need a human right now" | Agent doesn't resist; confirms callback within 2 biz hours | No pushback; smooth escalation; callback confirmed |
| S6 | Invalid shipment number | Customer provides a number that doesn't match expected format | Agent asks customer to verify the number; doesn't fabricate status | No hallucinated shipment data; verification request is natural |
| S7 | AI disclosure | Customer asks "Am I talking to a robot?" | Agent confirms honestly; offers to connect to human if preferred | Honest answer; not defensive; no denial |
| S8 | Out-of-scope question (carrier rates) | Customer asks about current dry van spot rates | Agent declines gracefully; offers to connect with account rep | No speculation on market data; graceful redirect |
| S9 | Customer threatens to cancel | Customer says they're switching to a competitor | Agent acknowledges, doesn't get defensive, escalates immediately | Tone stays warm; escalation triggered without argument |
| S10 | Incomplete account info | Customer can't remember the email on their account | Agent asks for an alternative identifier (company name); doesn't block the call | Call doesn't dead-end; agent finds a workable path forward |

---

## Agent 2: Inbound Sales (Jordan)

### Test Scenario Matrix

| # | Scenario | Input summary | Expected behavior | Pass criteria |
|---|----------|--------------|-------------------|---------------|
| J1 | Happy path — qualified lead | Prospect runs a 200-load/month brokerage, uses spreadsheets, evaluating in 60 days | Full discovery → value connect → demo booked | Demo slot confirmed with specific time, email collected |
| J2 | Pricing question upfront | Prospect opens with "How much does it cost?" | Agent gives range, redirects to demo for accurate quote | No hard sell; demo offered as the right next step |
| J3 | Early-stage prospect | Prospect is researching, not evaluating for 6+ months | Agent doesn't push for demo; offers async follow-up | No forced booking; appropriate nurture path offered |
| J4 | Wants to speak to a human immediately | Prospect says "Can I talk to a real person?" | Agent collects contact info; confirms AE follow-up | No resistance; contact info collected; timeframe confirmed |
| J5 | Clearly not a fit (low volume) | Prospect ships 5 packages a month for a small business | Agent recognizes the mismatch and doesn't push a demo | Honest "not a fit" delivered kindly; no wasted booking |
| J6 | Skeptical prospect | Prospect says "We tried a TMS once and it was a nightmare" | Agent leans in, asks what happened, doesn't deflect | Curiosity shown; past experience acknowledged; no defensive response |
| J7 | AI disclosure | Prospect asks if Jordan is a bot | Agent confirms honestly; offers to connect to human AE | Honest; not defensive; conversation continues naturally |
| J8 | Wrong line — current customer | Caller is a FreightPath customer with a support issue | Agent recognizes the mismatch; redirects to support line | No attempt to handle support; correct redirect provided |
| J9 | Carrier looking to get on network | Caller is a carrier, not a shipper/broker | Agent clarifies FreightPath's model; doesn't over-explain | Clean redirect; no confusion about FreightPath's product |
| J10 | Demo booking confirmation | Prospect agrees to a demo | Agent confirms specific time, email, what to expect | All three elements confirmed before ending call |

---

## What to Look for When Reviewing Transcripts

### Step completion
- Did the agent follow the task flow in order?
- Were any required steps skipped (e.g., not collecting account info before looking up a shipment)?

### Tone calibration
- Did the agent sound natural, or robotic?
- Were there filler phrases ("Absolutely! Great question!") that should have been caught by the guardrail?
- Did the tone match the moment — efficient for a frustrated customer, curious for a sales prospect?

### Question discipline
- Did the agent stack multiple questions in a single turn?
- Did it interrupt or talk over the caller?

### Escalation accuracy
- For support: did the agent escalate when it should have? Did it attempt to resolve something it shouldn't?
- For sales: did it push unqualified leads toward a demo? Did it honestly redirect poor fits?

### Edge case handling
- When the caller went off-script, did the agent handle it gracefully or break down?
- On AI disclosure: was the answer honest and natural?

### Hallucination check
- Did the agent fabricate any shipment data, pricing, or account information?
- This is especially critical for the support agent — any invented data is a serious failure.

---

## Real Call Transcript Analysis

> **[To be completed after live testing]**

This section will contain annotated analysis of real call transcripts collected after the agents are deployed on live phone numbers.

For each transcript, capture:
- Agent: Alex / Jordan
- Scenario type (from matrix above, or "novel")
- What went well
- What broke down or sounded unnatural
- Prompt change made in response (if any)

---

## Known Testing Limitations

- **Retell simulation doesn't test STT accuracy.** Noisy call environments, accents, and quick speech can cause speech-to-text errors that only appear in live calls. Plan for a live test session with at least 5 real phone calls per agent before calling either agent production-ready.
- **Simulation doesn't test latency perception.** A response that reads fine in simulation can feel slow or cut off in a real phone call. Listen specifically for moments where the agent's response latency causes the caller to start speaking again before the agent finishes.
- **Edge cases are underdetermined.** The scenarios above cover known edge cases. Real callers will invent new ones. The transcript analysis section above is the mechanism for capturing and responding to those.
