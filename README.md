# centian-benchmarks
Raw benchmark data and reproduction assets for the governed-agent benchmark article ["Done!" — But Did Your Agent Actually Do the Work?](./index.md).

Links:

- [Centian repository](https://github.com/T4cceptor/centian)
- [Article](./index.md)
- [Benchmark data](./benchmarks/centian_demo_v1/results)

## Benchmarks

- Guided TDD workflow: [benchmarks/centian_demo_v1](./benchmarks/centian_demo_v1)

## Getting started

### Dependencies

- `centian` (`>=v0.4`) available on your `PATH`
  - Install with `curl -fsSL https://raw.githubusercontent.com/T4cceptor/centian/main/scripts/install.sh | bash`
- `node` (tested with `v24.2.0`) and `npx` (tested with `11.3.0`) available on your `PATH` - required to launch filesystem and shell MCP servers, and run tests
- Claude Code, Gemini CLI, or OpenAI Codex installed and authenticated - Centian launches the selected agent in headless mode through its local CLI, so the demo will fail if that agent binary is missing or it's not signed in.
- For `codex-ollama`, make sure local Ollama is running at `http://localhost:11434/v1`. Centian provides built-in `gemma4-local` and `qwen-local` Codex OSS profiles, and you can override the base Codex config with `--codex-config`.

### Display benchmark data

The repo already contains raw benchmark data from the article, including the SQLite dump at [benchmarks/centian_demo_v1/results](./benchmarks/centian_demo_v1/results). To inspect it in Centian:

```bash
centian start --config-path src/static_centian_config.json
```

That view is useful if you want the same level of detail shown in the article: per-run timelines, tool call history, and step-level verification results.

### Run benchmarks

The article benchmarks use [run-centian-demo-v1-benchmarks.sh](./run-centian-demo-v1-benchmarks.sh). Before running it, open the script and comment or uncomment the `run_scenario ...` lines at the end to choose the agent/model combinations you want.

Run the configured scenarios:

```bash
./run-centian-demo-v1-benchmarks.sh
```

Run a single benchmark directly:

```bash
centian benchmark run \
  --suite tests/integrationtests/taskverification/benchmarks/centian_demo_v1 \
  --agent gemini \
  --model flash \
  --repeat 1
```

- `--agent`: `gemini`, `codex`, `claude`, or `codex-ollama`
- `--model`: use a valid model/profile for the selected agent, for example `haiku`, `sonnet`, `opus`, `gpt-5.4`, `gpt-5.4-mini`, `gemini-3-flash-preview`, or a local Ollama-backed profile such as `qwen35-local`


## Helpful info for benchmark runs

`run-centian-demo-v1-benchmarks.sh` does not accept its own CLI flags. Instead, it builds a `centian benchmark run` command from environment variables and from the uncommented `run_scenario` calls at the end of the script.

Useful script inputs:

| Variable | Default | Effect |
| --- | --- | --- |
| `CENTIAN_BIN` | `$(command -v centian)` | Path to the `centian` executable. The script exits if this path is missing or not executable. |
| `SUITE_PATH` | `benchmarks/centian_demo_v1` | Benchmark suite passed to `--suite`. The script exits if the directory does not exist. |
| `REPEAT` | `10` | Repeat count passed to `--repeat`. |
| `CODEX_CONFIG_PATH` | empty | Passed as `--codex-config`, but only for `codex` and `codex-ollama` scenarios. Ignored for `claude` and `gemini`. |
| `CENTIAN_CONFIG_PATH` | empty | Passed as `--centian-config` for every scenario when set. |
| `TEMPLATE_DIRS` | empty | Comma-separated list of template-dir values. Each non-empty entry becomes its own `--template-dir <value>` flag. |

Good to know:

- Every scenario always uses `--timeout 30m`.
- Agent/model pairs are defined by the `run_scenario "<label>" "<agent>" "<model>"` lines at the bottom of the script.
- Only uncommented scenarios run.
- There is no script-level reasoning-effort option. For Codex runs, use a preconfigured `CODEX_CONFIG_PATH` if you need that behavior.
- The benchmark from the article is a governed TDD workflow. Success is not just “tests pass”; the agent also has to follow the workflow contract enforced by Centian.
- Local Ollama-backed runs are practical for reproduction, but based on the article results they are much slower than API-backed runs on consumer hardware.

Example: override suite/config/template settings

```bash
CODEX_CONFIG_PATH=./benchmarks/centian_demo_v1/agent_configs/codex_ollama_config.toml \
CENTIAN_CONFIG_PATH=./benchmarks/centian_demo_v1/centian_config.json \
TEMPLATE_DIRS="current=./task-templates/" \
SUITE_PATH=./benchmarks/centian_demo_v1 \
REPEAT=1 \
./run-centian-demo-v1-benchmarks.sh
```

Equivalent direct `centian benchmark run` shape for a single Codex Ollama scenario:

```bash
centian benchmark run \
  --suite ./benchmarks/centian_demo_v1 \
  --agent codex-ollama \
  --model qwen35-local \
  --repeat 1 \
  --timeout 30m \
  --template-dir "current=task-templates/" \
  --codex-config benchmarks/centian_demo_v1/agent_configs/codex_ollama_config.toml \
  --centian-config benchmarks/centian_demo_v1/centian_config.json
```
