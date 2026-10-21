# Technical Interview Playbook: AgentTrace & Trajectory Forensics

> **Target Audience:** Engineering Leads & Hiring Managers at AI Agent, Evaluation, and LLM Tooling Startups (CrewAI, Confident AI / DeepEval, Langfuse, Giskard, Anthropic, Cognition/Devin).
> **Candidate:** Krish Joshi (AI Systems & Evaluation Engineer / 3rd-Year CS Undergraduate).

---

## 1. The 2-Minute STAR Elevator Pitch

When the interviewer says: **"Tell me about a challenging project you built or your background in agent evaluation."**

> **[SITUATION]**  
> *"Over the past year, as autonomous coding and personal assistant agents gained mainstream adoption, I noticed a critical failure mode: **agents fail silently**. In benchmark evaluations and real multi-turn workflows, language models frequently hallucinate actions—they tell the user 'I deployed the fix' or 'I sent the email', while behind the scenes, either the tool call was never dispatched (a Ghost Action), or the underlying shell command threw an unhandled 403 or exit code 1 that the model completely masked."*
>
> **[TASK]**  
> *"Drawing on my experience as a Senior Quality Reviewer at Handshake AI, where I audited multi-turn agent execution traces across simulated assistant ecosystems, I set out to build an open-source, production-grade observability and forensic interceptor called **`AgentTrace`**."*
>
> **[ACTION]**  
> *"I designed `AgentTrace` around three core pillars:*  
> *1. **Non-Intrusive Runtime Interception:** A lightweight Python SDK with zero heavy dependencies that records step thoughts, tool arguments, stdout/stderr, and token metrics into structured JSONL.*  
> *2. **Forensic Hallucination Engine:** An automated auditor that parses model natural-language assertions against the ground-truth execution graph. It detects Ghost Actions, Masked Tool Failures, and Circular Retry Loops, and computes a calibrated 0.0 to 5.0 Structured Quality Score (SQS).*  
> *3. **Cryptographic Sandbox State Verification:** Inspired by Terminal-Bench and Harbor benchmark environments, I integrated ephemeral sandboxing with SHA-256 state hashing to mathematically prove whether a claimed file edit actually produced disk-level mutations.*  
> *4. **Interactive 21st UI:** Built a visual dashboard with timeline scrubbers and side-by-side claim vs. ground-truth diff inspectors."*
>
> **[RESULT]**  
> *"The project achieves sub-millisecond audit throughput, detects action hallucinations with 100% precision on benchmark suites, and provides engineering teams with an automated gate to block ungrounded agent responses before they hit end users."*

---

## 2. Navigating the NDA & Experience Verification

If asked about your Handshake / Frontier Lab experience:
* **What you CAN say (Technical Ground Truth):**
  - "I served as an R1 Senior Quality Reviewer auditing multi-turn agent execution traces."
  - "I evaluated assistant tools like calendar insertion, messaging, file manipulation, and shell execution against strict SQS 4.5–5.0 rubrics."
  - "I authored Dockerized evaluation environments using two-tier PyTest harnesses and SHA-256 state assertions to prevent agent false passes."
  - "I documented this exact methodology publicly in my open-source `AgentTrace` repository."
* **What you NEVER mention (Confidentiality Protection):**
  - Never disclose client names (Meta, Hatch, etc.).
  - Never mention private internal staging URLs, persona names, or proprietary API key names.
  - Frame everything around **open standards, public benchmarks (Harbor/Terminal-Bench), and your own code**.

---

## 3. High-Frequency Technical Interview Questions & Senior Answers

### Q1: "Why can't you just use an LLM-as-a-judge to detect action hallucinations?"
**Candidate Answer:**
> *"Using an LLM to evaluate an LLM's own tool execution creates two major failure vectors: **Judge Blindness** and **Non-Deterministic Latency**.  
> If a model already suffered from grounding drift in its main turn, prompting a secondary LLM with the raw transcript often results in the judge model accepting the agent's persuasive text rather than strictly parsing the JSON execution log.  
> Furthermore, LLM-as-a-judge adds 2–5 seconds of latency and substantial API costs per turn. In `AgentTrace`, I built a **deterministic regex & graph-reconciliation engine**. It maps semantic action intent directly to physical tool call IDs in O(1) time with mathematical certainty and zero token overhead."*

---

### Q2: "What is an Action Hallucination vs a standard hallucination?"
**Candidate Answer:**
> *"A standard hallucination is factual error in knowledge retrieval (e.g., hallucinating a fake Python library or historical date).  
> An **Action Hallucination** is far more dangerous: it occurs in tool-calling agents when the model claims an external state transition occurred in the physical or digital environment when it did not.  
> We divide this into two classes:  
> 1. **Ghost Actions:** The agent never dispatches the tool call at all, but summarizes to the user that it did.  
> 2. **Masked Failures:** The agent dispatches the tool, receives an error (like HTTP 401 Unauthorized or SQL syntax error), but ignores the tool result and claims successful completion. `AgentTrace` catches both."*

---

### Q3: "How does your SHA-256 Sandbox Verifier work, and why is it necessary?"
**Candidate Answer:**
> *"Even if an agent executes `write_file('app.py')` and the tool returns exit code 0, that does not prove the file was correctly modified or that the agent didn't write an empty string or corrupt the file.  
> In `AgentTrace`, before the agent executes, we generate a SHA-256 manifest of the entire workspace. After the agent claims completion, we snapshot the post-execution state. If the agent claims it fixed a bug in `auth.py`, but the SHA-256 hash delta is zero, we trigger an `UNVERIFIED_STATE_MUTATION` flag. It provides cryptographic proof of work."*

---

### Q4: "How does `AgentTrace` compare to Langfuse, Phoenix (Arize), or Giskard?"
**Candidate Answer:**
> *"Observability platforms like Langfuse and Phoenix are fantastic for logging spans, latency, and cost telemetry. Giskard is strong at pre-deployment scanning.  
> However, most existing APMs treat tools merely as generic trace spans. They don't perform **semantic claim-to-execution reconciliation**. They won't flag a run where the agent's natural language contradicts the tool log.  
> `AgentTrace` is purpose-built as a **forensic gatekeeper** that calculates an objective SQS reliability score and catches the exact gap between what the agent *said* and what the agent *did*."*

---

## 4. Whiteboard Architecture Quick Reference

When asked to sketch the architecture on a whiteboard:
1. **Draw the Triad:**
   - Left: **Agent Text Output** (Claims)
   - Center: **Forensic Auditor / Grounding Engine** (Reconciler)
   - Right: **Physical Sandbox / Tool Execution Log** (Ground Truth)
2. **Highlight the Diff:**
   - Draw an arrow showing how `Claim - Execution Log = Hallucination Delta`.
3. **Show the SQS Formula:**
   - Show how critical ghost actions immediately drop the score from 5.0 to < 2.5 (Automated Rejection).
