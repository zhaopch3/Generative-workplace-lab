# Workplace Simulation Lab: Simulating Organizational Behavior with Generative Agents

A multi-agent simulation platform for organizational behavior research. Four agents driven by large language models form a functional team (one team lead and three team members) and spend a working week in a pixel-art office: they plan their days, move around, talk, form memories, experience emotions, and fill in questionnaires at scheduled times. The researcher sets only the initial conditions (for example, the team lead’s leadership style); after that the world unfolds freely, with no scripted events.

The platform is meant for organizational behavior research in general: the researcher defines the experimental conditions, and agents answer the original scales of published studies at the measurement times those studies used. The built-in templates come from the first application, a study of abusive supervision: the same team is run under a high-abuse and a low-abuse condition, matched by random seed.

The project builds on [GenerativeAgentsCN](https://github.com/x-glacier/GenerativeAgentsCN) (a refactor of [wounderland](https://github.com/Archermmt/wounderland), derived from Stanford’s [Generative Agents](https://github.com/joonspk-research/generative_agents)). The upstream README is kept in [docs/upstream_README_en.md](./docs/upstream_README_en.md).

## About this review edition

This copy of Workplace Simulation Lab V27.4 is prepared for reviewers. The program is the V27.4 release. Material that is not needed to read, run or check the platform has been left out:

- the development notes and the per-version release notes (an English summary of the versions is in [CHANGELOG.md](./CHANGELOG.md));
- the second-language copies of this README and of the upstream README;
- a few scratch scripts used during development to test local embedding connections, and a map-alignment debug image;
- two research files that belong to the reproduction package (the list of reverse-keyed items and the coding manual for the replication predictors).

Apart from that, only three things differ from the release: two code comments that pointed to the removed notes, the file name of the debug image the map tool writes, and the default app name the platform sends to OpenRouter. Some code comments and the dictionary of the second interface language remain in Chinese.

No API keys are included. Add your own in the API settings panel; keys are stored only in a local `.env` file in the project folder.

Where to look first:

| What | Where |
| --- | --- |
| Installing, running a batch experiment, outputs | This README |
| The eight world templates (experimental groups) | `generative_agents/data/world_presets/`. The suffix gives the language of the world: `_CN` for the pilot reported in the proposal, `_EN` for the English translations. Each pair differs only in the leadership style of the team lead. |
| Leadership-style text of each condition (original and English translation) | `generative_agents/data/experiment_conditions.json` |
| Agents | `generative_agents/frontend/static/assets/village/agents/`: the English-named folders (Klaus, Ayesha, Maria, Wolfgang and 22 others) are used in `_EN` worlds; the same characters under their names in the other language are used in `_CN` worlds |
| Prompts | `generative_agents/data/prompts_en/` (`_EN` worlds) and `generative_agents/data/prompts/` (`_CN` worlds) |
| Replication questionnaires | `generative_agents/data/scale_battery/battery_v1.json`. It contains items of published scales and is provided for review only. |
| Automated tests (191) | `generative_agents/tests/`; see “Development and tests” below |

> The interface opens in English, and the English templates run the simulation in English. Chinese is available through the language switch at the top right and through the `_CN` templates. The pilot reported in the proposal ran the `_CN` templates.

## Features

- **Persona manipulation**: the team lead’s “leadership style” has a high-abuse and a low-abuse version, written from Tepper’s (2000) definition. All other persona text, world rules and initial conditions are identical across the two conditions.
- **Paired batch runs**: run several groups with several replicates each. The i-th world in every group uses the same random seed and therefore draws the same personalities, giving matched pairs. Worlds run in parallel and are fully isolated.
- **Personality sampling**: each agent receives a Big Five profile drawn from real IPIP-50 respondents, written as short descriptions following the BFI-10 items (can be turned off).
- **Replication questionnaires**: a built-in battery taken from published abusive-supervision studies, administered over the five workdays with the original respondent, time frame and measurement waves. Items are used verbatim. Questionnaires can be edited or added on the experiment setup page.
- **World rules**: workplace rules shared by all agents (hours, meetings, division of labor, leave, raising concerns, and so on), giving the behaviors measured by the scales real channels in the world.
- **Live view and replay**: watch agents move and talk on the map in real time; replay any finished world step by step with the event log.
- **Data export**: questionnaire answers, psychological states, behavior and dialogue logs, and model-call logs, downloadable per batch. Worlds with identical configuration are grouped automatically into comparable groups.

## Requirements

- Windows 10/11 (one-click launcher provided); macOS and Linux via the command line.
- Python 3.11 (3.10–3.12 work).
- 16 GB RAM or more: each running world uses about 0.5–1 GB.
- An API key for a chat model (e.g., OpenRouter or DeepSeek).
- An embedding model for memory retrieval: a cloud service or a local Ollama model (a dedicated GPU helps but is not required).

## Installation

1. Download this repository (Code → Download ZIP on GitHub, or `git clone`) and unzip it into a folder whose path has no special characters.
2. Install the environment (once, about 3–10 minutes, needs internet):
   - Windows: double-click `Install.bat`. If Python is missing it installs Python 3.11 with the built-in winget (or opens the download page); it then creates a separate `.venv` inside the project folder, installs the dependencies, checks that they import, and, if Ollama is installed, offers to download the local embedding model. When the official package index is slow (common in mainland China) it suggests the Tsinghua mirror. Double-clicking `Start.bat` also works: without a `.venv` it runs the installer first. Installer and launcher messages are in English, or in Chinese on a Chinese system (set `SIM_LANG=en` or `SIM_LANG=zh` to choose).
   - macOS / Linux: run `bash install.sh` in the project folder.

   Everything is installed into the project’s `.venv` (about 600 MB) and does not touch other Python installations; deleting the project folder uninstalls it completely. The installer never asks for or stores API keys. If the environment breaks, run `Install.bat --recreate` (macOS / Linux: `bash install.sh --recreate`) to rebuild it.

   Manual installation also works: `python -m venv .venv`, then `.venv\Scripts\pip install -r requirements.txt` (macOS / Linux: `.venv/bin/pip install -r requirements.txt`). Dependencies for the local HuggingFace embedding and the Baidu Qianfan, Zhipu and iFlytek Spark channels are in `requirements-optional.txt`; install them only when needed.

3. Start the app:
   - Windows: double-click `Start.bat`.
   - macOS / Linux: `.venv/bin/python launch.py`.

   Your browser opens `http://127.0.0.1:5071/research`. The app runs in the background: closing the terminal window does not stop it, and simulations keep running. To stop it or before upgrading, end it with `Get-Process python | Stop-Process -Force` in PowerShell (Windows) or `pkill -f server.py` (macOS / Linux); running simulations stop too, and saved worlds can be resumed later. The launcher warns you if an older version is still running in the background.

## First run: API settings

Open “Experiments & archive” in the top menu and expand “⚙ API settings”. There are two parts; fill both and click “Save both settings” once.

**① Chat model** (used by agents to think, plan and talk)

| Provider | Notes |
|---|---|
| OpenRouter (default) | One key for hundreds of models; can also serve embeddings. |
| DeepSeek (official) | Recommended model: `deepseek-flash`. The app turns DeepSeek’s thinking mode off by default (with thinking on, each call generates hundreds to thousands of hidden tokens, which is several times slower and billed as output). |
| OpenAI (official) | Embeddings switch automatically to OpenAI `text-embedding-3-small`. |
| Google Gemini (official) | Uses Gemini’s OpenAI-compatible endpoint; get a key from Google AI Studio. No embedding channel, so choose one in part ② (local Ollama recommended). |
| Anthropic Claude (official) | Uses Claude’s OpenAI-compatible endpoint; get a key from the Claude Console. Recent Claude models do not accept a custom temperature, so the app calls them at the default temperature. No embedding channel, so choose one in part ②. |
| Custom compatible endpoint | Any OpenAI-compatible server, e.g., a local vLLM. |

**② Embedding model** (turns memories into vectors for retrieval)

| Channel | Notes |
|---|---|
| Auto | Follows the chat provider; uses OpenRouter when the chat provider is DeepSeek, and keeps the saved embedding setting for Gemini or Claude. |
| OpenRouter / OpenAI cloud | Default model `text-embedding-3-small`. |
| Alibaba Cloud Model Studio (DashScope) | Default model `text-embedding-v4`; reachable from mainland China without a proxy. |
| Local Ollama | Default model `bge-m3:latest`; free and offline. Install [Ollama](https://ollama.com/) and run `ollama pull bge-m3` first. |
| Local HuggingFace | Needs about 2 GB of extra dependencies. |
| Custom compatible endpoint | Any OpenAI-compatible `/embeddings` service. |

When both parts use the same provider, enter the key once. After saving, click “Test connection” and start only when both the chat model and the embedding report OK.

API keys are stored only in the `.env` file in the project root on your machine and are never written into simulation archives. Do not upload `.env` anywhere.

## Running experiments

### Single run (debugging and observation)

1. Open “New experiment”, enter a name, and choose or load a world template.
2. Choose “Single run” and click “Start experiment”.
3. In “Experiments & archive”, click “View” to watch the world live on the map.

### Batch run (formal experiments)

1. Open “New experiment” and enter a name.
2. Choose “Batch run”.
3. Under “Experimental groups”, select both “Persona-manipulation-high-abuse_EN” and “Persona-manipulation-low-abuse_EN” (in English mode this pair is selected by default; hold Ctrl to change the selection).
4. Set “Replicates per group” and “Parallel worlds” (with 16 GB RAM, at most 6).
5. Check the total number of worlds shown below, then click “Start experiment”.
6. Follow progress in the batch panel on “Experiments & archive”. If a world fails, click “Rerun failed/stopped worlds”; it resumes from where it stopped.

In batch mode, each world’s duration, step count and personas come from the template files; the form at the top of the page applies only to single runs. The chat and embedding models come from the global API settings.

### Built-in templates

| Template | Content |
|---|---|
| Persona-manipulation-high-abuse_EN / low-abuse_EN | A team in a company’s marketing operations department (team lead Klaus; members Ayesha, Maria and Wolfgang). Starts Monday 08:00 and runs five workdays (120 steps of 60 minutes) with no scheduled events. The two groups differ only in the team lead’s leadership style: nine kinds of concrete behaviour written as habits Klaus has formed over years of leading the team, describing only what others can see, without evaluative words (final in V26; see the V26 notes in `changelog/`). |
| Event-manipulation-high-abuse_EN / low-abuse_EN | The same team, manipulated through 8 scheduled events; starts 13 February 08:00 and runs 7 days (168 steps). |
| Persona-manipulation-high-abuse_CN / low-abuse_CN, Event-manipulation-high-abuse_CN / low-abuse_CN | The same four templates for Chinese-language worlds, used for the pilot study: the same four agents think, talk and answer surveys in Chinese. The suffix marks the language of the world: _EN for English-language worlds, _CN for Chinese-language worlds. The _EN templates are translations of the _CN ones (V27), with every other setting identical. Until V27.3 the _EN templates had no suffix and the _CN templates had Chinese names; the old names still work. |

### Interface language and simulation language (V27)

- **Interface language**: the interface opens in English. The language switch at the top right (Chinese / EN) changes all interface text and is remembered by the browser. It affects display only, never data. In English mode the setup page shows the English agents and templates; a link under the agent list shows the Chinese ones too.
- **Simulation language**: set by the world template. The English templates above build English worlds: prompts (`data/prompts_en/`), personas, world rules, leadership styles, place names and surveys are in English, and answers are parsed in English formats. Chinese templates behave exactly as before. English and Chinese worlds behave differently, so **do not pool their data**; they never share a comparison group, and `experiment_manifest.csv` has a `language` column.
- Every English survey item comes from a source that can be checked (the original reproduction items, the coding workbook or public scale pages); the source of each survey is recorded in `data/i18n/surveys_en.json`. English personas and leadership styles are translations of the Chinese ones (`data/i18n/content_en.json`). After changing a Chinese template, run `python tools/build_en_assets.py` to regenerate the English version.

## Results and data

- When a batch finishes, click “Download whole batch” under “Batch history” on “Experiments & archive”. Each comparable group is in its own subfolder of the zip.
- Raw files live in `generative_agents/results/`:

| Folder | Content |
|---|---|
| `checkpoints/<world>/` | Full state at every step (resume or replay from any step) and line-by-line dialogue logs |
| `surveys/<world>/` | Questionnaire answers (CSV and JSONL) |
| `psych/<world>/` | Changes in agents’ psychological states (mood, stress, energy, sociability) |
| `monitor/<world>/` | Behavior timeline (`timeline.jsonl`) and a record of every model call (`model_calls.jsonl`, with latency and token usage) |

## Time and cost (rough guide)

These are rough estimates; actual numbers depend on the model, network and number of parallel worlds.

- One world (five workdays) makes about 3,000–4,000 chat-model calls with about 7 million input tokens.
- With `deepseek-flash` (thinking off) and local embeddings, one world takes about 1–1.5 hours; 10 worlds with 6 in parallel take about 2.5–3 hours.
- At DeepSeek’s off-peak price, one world costs about US$1. DeepSeek’s peak hours are 9:00–12:00 and 14:00–18:00 Beijing time on weekdays; all other hours are half price.

### Cheap mode (V25.1, for pilots)

On the setup page, expand “Cost-saving options (for limited budgets)” and tick “Budget mode” to cut model calls: questionnaire prompts put the shared part first so the provider can cache it, questionnaires due at the same time are answered several per call, events seen in one step are rated for importance in one call, and with 60-minute steps hourly plans are not split into sub-tasks. In an offline comparison this cut model calls by about 70% and input by about 60%; run time should roughly halve. Cheap mode slightly changes answers and behavior, its worlds are put in their own comparison group, and it is **for pilots, prompt tuning and model comparisons only, not for study data**. Unticked, the app behaves exactly as the main mode. See the V25.1 notes in `changelog/` (in Chinese).

## Research design notes

- **Use one chat model and one embedding model for an entire study.** Different models behave and retrieve memories differently, so their data cannot be pooled. Providers also update the model behind a given name, so record the model name and run dates in the paper, and run all groups of a batch in the same period.
- Do not use the intervention tools on the live monitoring page in formal experiments; intervened worlds are flagged in the logs and cannot be read as spontaneous behavior.
- Every model-call record stores the model name and thinking setting, so runs can be audited afterwards.

## Copyright and data

- The replication questionnaires come from published studies, and some scales are copyrighted (for example, the Maslach Burnout Inventory). Before publishing these items in a public repository, make sure you have permission. Otherwise, publish only the build script (`tools/build_v24_battery.py`) and let users generate the questionnaire file `data/scale_battery/battery_v1.json` and the questionnaire part of the templates locally from their own coding sheet.
- `generative_agents/results/` (simulation data) and `.env` (API keys) are excluded in `.gitignore` and will not be committed.

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| The launcher says “The system cannot find the path specified” or a dependency is missing | The environment is not installed. Double-click `Install.bat` (macOS / Linux: `bash install.sh`); if problems remain, add `--recreate`. |
| Embedding test fails with `SSLError` or a dropped connection | The network to the cloud embedding service is unstable (common on some networks). Switch to DashScope or local Ollama. The app retries for about one minute automatically. |
| Local Ollama test returns 404 | Wrong embedding model name, or the model is not downloaded. Use `bge-m3:latest` and run `ollama pull bge-m3`. |
| Runs are slow | Check latencies in `monitor/<world>/model_calls.jsonl`. Common causes: the model’s thinking mode is on, embeddings go through a proxy, or too many worlds run at once. |
| The browser cannot open the page | Run the launcher again and use the address it prints (it moves to 5072 and up when 5071 is taken). Ports 5060 and 5061 are blocked by browsers, so the app starts at 5071. |
| After upgrading, the page still shows the old version | The old server is still running in the background. Stop it as above and start again; the version is shown at the top left of the page. |

## Development and tests

Run all tests from the `generative_agents` folder:

```
python -m unittest discover -s tests -p "test_*.py"
```

Project layout:

```
launch.py                     Launcher (finds a free port and opens the browser; log in logs/server.log)
generative_agents/
  server.py                   Web interface and experiment management
  start.py                    Simulation process for a single world
  modules/                    Agents, memory, model calls, questionnaires, export
  data/prompts/               All prompt templates
  data/world_presets/         World templates (experimental groups)
  data/scale_battery/         Replication questionnaires
  frontend/                   Pages, map and character assets
  tools/                      Scripts for loading questionnaires, building templates and maps
  tests/                      Automated tests
```

A summary of the versions is in [CHANGELOG.md](./CHANGELOG.md).

## Acknowledgements and license

- Stanford Generative Agents: Park, J. S., O’Brien, J. C., Cai, C. J., Morris, M. R., Liang, P., & Bernstein, M. S. (2023). *Generative Agents: Interactive Simulacra of Human Behavior*. UIST ’23.
- [wounderland](https://github.com/Archermmt/wounderland) and [GenerativeAgentsCN](https://github.com/x-glacier/GenerativeAgentsCN).
- Personality pool: the public IPIP Big Five dataset (BIG5) from Open Psychometrics; processing is described in `generative_agents/data/personality/SOURCE.md`.
- Definition and measurement of abusive supervision: Tepper, B. J. (2000). Consequences of abusive supervision. *Academy of Management Journal, 43*(2), 178–190.
- This project keeps the upstream Apache License 2.0; see [LICENSE](./LICENSE).
