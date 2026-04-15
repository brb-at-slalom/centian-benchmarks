# "Done!" — But Did Your Agent Actually Do the Work?

**How 9 AI agents performed when we stopped trusting their self-assessment**

---

Your AI agent just finished a coding task. The tests pass. The commit message is clean. "Task complete," it says.

But did the agent write the tests first and then make them pass — or did it write the code, realize the tests didn't match, and quietly adjust the tests to fit? Did it follow the plan it committed to, or did it drift halfway through and improvise? You don't know. The agent says it's done, and you take its word for it.

This is the gap between **capability** and **compliance**. Most coding benchmarks test whether an agent *can* produce correct code. That's a solved problem for flagship models — they're very good at it. But producing correct code and producing it *through a correct process* are not the same thing. In production, the process matters: regulated industries need audit trails, teams need reproducibility, and anyone deploying agents at scale needs to know that "tests pass" means the tests were real, not retrofitted.

We wanted to measure this gap directly. So we built a benchmark that doesn't just check the output — it governs the entire workflow.

## What we tested

We took 9 agent/model combinations — spanning Claude (Haiku, Opus, Sonnet), OpenAI Codex (gpt-5.4, gpt-5.4-mini), local open-source models via Ollama (gemma4, qwen3.5), and Gemini (Flash, Pro) — and ran each through the same task 10 times. 90 total runs.

The task itself is straightforward: implement a small feature using test-driven development. What makes this benchmark different is *how* the agent has to do it.

Every agent action flows through [Centian](https://github.com/T4cceptor/centian), an open-source MCP proxy that enforces a structured workflow. The agent cannot touch the code directly — it can only act through Centian's governed tool surface. The process looks like this:

1. **Onboarding** — understand the task and the environment
2. **Planning** — declare the approach, including specific artifacts like test file names and expected test outputs. These declarations are *frozen* into an execution contract.
3. **Scaffolding** — set up the test file and initial test structure
4. **Execution** — the core TDD cycle:
   - Write a failing test
   - Confirm the test actually fails (enforced by Centian)
   - Implement the code to make it pass
   - Confirm the test passes — **and that the test file itself was not modified** since scaffolding

That last constraint is the key. It prevents the most common agent cheat: modifying the test to match the code instead of fixing the code to pass the test. Centian freezes the test file after scaffolding and verifies its integrity at execution time. If the agent touched the test, the run fails.

## What we measured

**Success Rate** — did the task eventually complete with all checks passing? This includes runs where the agent needed a restart or recovery.

**First Pass** — did the agent complete the task on the first attempt, without any restarts, failures, or timeouts? This is the stricter metric — it measures how well the model follows the governed process without stumbling.

**MCP Events** — the total number of tool calls, split between Centian governance events and actual code-level MCP actions. The ratio reveals how much overhead the governance layer adds versus how many actions the agent takes on the code itself.

**Errors** — tool call failures, again split between Centian-level and MCP-level. High Centian errors typically mean the agent is fighting the process. High MCP errors mean it's struggling with the code.

**Median Time** — wall-clock time from task start to completion, measured across successful runs.

## What we found (preview)

The headline: **flagship models have this locked down.** Claude Opus, Sonnet, Codex gpt-5.4, and Gemini Pro all achieved 100% success and 100% first pass. They understood the process, followed it cleanly, and completed the task without needing retries.

But how they got there differs — and the real story is in the models that *didn't* get perfect scores. A model that writes correct code but can't follow a governed workflow. A model that always recovers but never gets it right the first time. A local model that competes on accuracy but not on speed. These patterns matter more than the final ranking, because they reveal what "production-ready" actually requires beyond raw coding ability.

The full metric-by-metric analysis, raw data, and reproduction instructions follow below.

---