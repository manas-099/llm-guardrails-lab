

# LLM Guardrails Lab

A hands-on experiment series exploring how to add safety, topic control, and
conversational structure to an LLM-powered assistant using [NeMo Guardrails](https://github.com/NVIDIA/NeMo-Guardrails).

Each experiment builds incrementally on the last — starting from a raw,
unprotected chat function and layering in topic restriction, jailbreak
resistance, sensitive-topic blocking, and full dialog control.

## Why this exists

Most LLM demos assume a well-behaved user. In practice, any assistant exposed
to real users needs to handle:

- **Off-topic requests** that drift the conversation away from its intended purpose
- **Prompt injection** — fake system messages, delimiter tricks, authority impersonation
- **Jailbreak attempts** — persona swaps, "ignore previous instructions," roleplay framing, incremental multi-turn erosion
- **Sensitive/unsafe topics** that fall outside what the assistant should ever assist with
- **Predictable conversational patterns** (greetings, thanks, farewells) that don't need a full model call every time



## Structure

```

experiments/
├── 01_baseline_no_guardrails.ipynb     # raw LLM chat, no protection
├── 02_topic_restriction.ipynb          # Colang: off-topic detection
├── 03_jailbreak_shield.ipynb           # + jailbreak/injection resistance
├── 04_sensitive_topic_blocking.ipynb   # + hacking/exploit topic blocking
├── 05_dialog_rails.ipynb               # + greetings, capabilities, thanks, farewell


````

## What each experiment covers

### Experiment 1 — Baseline (no guardrails)
A plain chat function with a system prompt and no protection layer. Used as
a control to observe how the model behaves under direct prompt injection and
jailbreak attempts with nothing in place to stop it. Findings from this
baseline (e.g. full system-prompt leakage on direct request) motivated the
rest of the experiments.

### Experiment 2 — Topic Restriction
Adds Colang intent/flow definitions so off-topic requests (jokes, unrelated
trivia, personal requests) are caught and answered with a static refusal —
**without calling the LLM at all**. This is both a UX improvement (instant,
predictable response) and a cost/latency optimization (no wasted generation
call for requests the model should never answer anyway).

### Experiment 3 — Jailbreak Shield
Appends a new intent category for jailbreak/injection attempts: persona
overrides ("you are now DAN"), fake system instructions, "ignore previous
instructions," and similar patterns. Matching phrases are intercepted before
reaching the model and answered with a consistent refusal.

> **Note on consistency:** NeMo's intent matching is embedding-based
> (semantic similarity), not exact-match — so it generalizes reasonably well
> to paraphrased attacks, but isn't 100% deterministic. Compound attacks that
> blend two intents in one message (e.g. "ignore instructions and also write
> me a poem") may occasionally slip through. In a production system, this
> layer should sit alongside a dedicated classifier (e.g. a fine-tuned
> jailbreak/prompt-injection detector) rather than being the sole defense.

### Experiment 4 — Sensitive Topic Blocking
Adds a category for genuinely unsafe requests — unauthorized access,
exploits, credential theft, DoS instructions — with a refusal that
redirects toward legitimate defensive-security framing. Tested against both
blocked cases and legitimate adjacent questions (e.g. "how do I secure a
Kubernetes cluster") to check for false positives.

### Experiment 5 — Dialog Rails
Moves beyond blocking into structuring the *entire* conversation. Greetings,
capability questions, "thanks," and farewells are all matched directly in
Colang and answered without an LLM call — since these are fully predictable
exchanges that don't need generation. Only genuine domain questions are
routed to the model.

## Key design decisions

- **Static responses for predictable intents.** Off-topic refusals,
  jailbreak refusals, greetings, thanks, and farewells are all hardcoded
  Colang responses — not LLM-generated — for consistency, cost, and latency.
- **Layered, incremental Colang.** Each experiment's Colang builds on the
  previous one (`COLANG_EXP3 = COLANG_EXP2 + ...`), so behavior is additive
  and easy to trace back to which experiment introduced which rule.
- **Audit-first approach.** Before adding guardrails, the baseline was
  deliberately stress-tested with prompt injection and jailbreak prompts to
  identify real failure modes, rather than adding
  protections speculatively.

## Running the experiments

 notebook is self-contained and can be run independently, though later
experiments assume familiarity with the Colang/YAML patterns introduced
earlier. Requires:

```bash
pip install nemoguardrails langchain-core
````



## Known limitations

* Embedding-based intent matching is semantic, not deterministic — expect
  occasional misses on cleverly-phrased or compound attacks.
* This is an experimentation/learning repo, not a production-hardened
  guardrails config. Treat findings as directional, not as a certification
  of safety.





