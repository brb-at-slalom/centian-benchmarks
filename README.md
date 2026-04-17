# centian-benchmarks
Repository to store and evaluate agent benchmarks on centian templates.

[Whats centian?](https://github.com/T4cceptor/centian)

## Benchmarks

- Centian TDD Workflow - [to benchmark](benchmarks/centian_demo_v1)

## Getting started

### Dependencies
- `centian` (`>=v0.4`) available on your `PATH`
    - To install run: `curl -fsSL https://raw.githubusercontent.com/T4cceptor/centian/main/scripts/install.sh | bash`
- `node` (tested with `v24.2.0`) and `npx` (tested with `11.3.0`) available on your `PATH` - required to launch filesystem and shell MCP servers, and run tests
- Claude Code, Gemini CLI, or OpenAI Codex installed and authenticated - Centian launches the selected agent in headless mode through its local CLI, so the demo will fail if that agent binary is missing or it's not signed in.
- For `codex-ollama`, make sure local Ollama is running at `http://localhost:11434/v1`. Centian provides built-in `gemma4-local` and `qwen-local` Codex OSS profiles, and you can override the base Codex config with `--codex-config`.

### Display data

A quick and easy way to display the data provided in this repository and dive deeper into benchmark analytics:
```bash
centian start --config-path src/static_centian_config.json
```

### Run agents

1. (Optional, but recommended) Adjust which scenarios will run
   - Open `run-centian-demo-v1-benchmarks.sh`
   - At the end of the file, comment or uncomment the `run_scenario ...` lines for the agent/model combinations you want

2. Runs all agents configured in the script:
```bash
./run-centian-demo-v1-benchmarks.sh
```

Run single benchmark:

```bash
centian benchmark run \
  --suite tests/integrationtests/taskverification/benchmarks/centian_demo_v1 \
  --agent gemini \
  --model flash \
  --repeat 1
```

- `--agent`: `gemini`, `codex`, `claude`, or `codex-ollama`
- `--model`: use a valid model/profile for the selected agent, for example `haiku`, `sonnet`, `opus`, `gpt-5.4`, `gpt-5.4-mini`, `gemini-3-flash-preview`, or a local Ollama-backed profile such as `qwen35-local`


## Advanced runs

`run-centian-demo-v1-benchmarks.sh` does not accept its own CLI flags. Instead, it builds a `centian benchmark run` command from environment variables and from the uncommented `run_scenario` calls at the end of the script.

Supported script inputs:

| Variable | Default | Effect |
| --- | --- | --- |
| `CENTIAN_BIN` | `$(command -v centian)` | Path to the `centian` executable. The script exits if this path is missing or not executable. |
| `SUITE_PATH` | `benchmarks/centian_demo_v1` | Benchmark suite passed to `--suite`. The script exits if the directory does not exist. |
| `REPEAT` | `10` | Repeat count passed to `--repeat`. |
| `CODEX_CONFIG_PATH` | empty | Passed as `--codex-config`, but only for `codex` and `codex-ollama` scenarios. Ignored for `claude` and `gemini`. |
| `CENTIAN_CONFIG_PATH` | empty | Passed as `--centian-config` for every scenario when set. |
| `TEMPLATE_DIRS` | empty | Comma-separated list of template-dir values. Each non-empty entry becomes its own `--template-dir <value>` flag. |

Behavior baked into the script:

- Every scenario always uses `--timeout 30m`.
- Agent/model pairs are defined by the `run_scenario "<label>" "<agent>" "<model>"` lines at the bottom of the script.
- Only uncommented scenarios run.
- There is no script-level reasoning-effort option. For Codex runs, use a preconfigured `CODEX_CONFIG_PATH` if you need that behavior.

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
