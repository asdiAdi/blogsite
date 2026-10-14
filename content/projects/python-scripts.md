+++
title = "Python Scripts (AI CLI)"
date = "2026-09-05"
draft = false
cover = "/images/cover/python-scripts.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++

[Github](https://github.com/asdiAdi/python_scripts)

A collection of small Python scripts behind a single `ai` dispatcher. Each folder in `scripts/` becomes a subcommand.
<!--more-->

{{< details summary="TLDR" >}}

- I made this because I like making little scripts on Linux and Windows to make life easier, and AI makes that much faster now.
- Technologies used: [Python 3.14](https://www.python.org/), [uv](https://docs.astral.sh/uv/), [OpenRouter](https://openrouter.ai/), [python-dateutil](https://dateutil.readthedocs.io/), [pytest](https://pytest.org/), [hatchling](https://hatch.pypa.io/)
- Install as a CLI: `uv tool install git+https://github.com/asdiAdi/python_scripts`, then run `ai --list`
  {{< /details >}}

I was a fan of making scripts in Linux and Windows to make my life easier.
With the help of AI this is easier to do now and so I tried to use it to make me some scripts that I think I can use.

The idea is simple: instead of a pile of unrelated scripts, there is one dispatcher (`main.py`) that auto-discovers every `scripts/*/cli.py`.
Folder `hello_world` becomes command `hello-world`, and so on.
Each script folder owns its `cli.py`, `test_cli.py`, and `README.md`.

## Install

```bash
uv tool install git+https://github.com/asdiAdi/python_scripts
ai --list
ai ask "what is the capital of france"
# → Paris
```

Uninstall:

```bash
uv tool uninstall python_scripts
```

Local dev:

```bash
uv sync --group dev
uv run python main.py --list
uv run python main.py hello-world --name Alice
# → Hello Alice
```

## Configuration

Configuration is TOML-only (`~/.config/ai/config.toml`):

```bash
ai config init   # writes ~/.config/ai/config.toml
ai config show   # resolved values, secrets redacted
ai config path   # print location
```

Scripts that call a model read the `[openrouter]` section via `common/prompt.py`. If `api_key` or `base_url` is missing they print `error: ...` to stderr and exit `1`.

## Commands

| Command       | What it does                                                                   |
| ------------- | ------------------------------------------------------------------------------ |
| `ask`         | Ask AI any question, answered in N sentences                                   |
| `commit`      | Stage all changes, generate Conventional Commit candidates, pick one to commit |
| `config`      | Manage `~/.config/ai/config.toml`                                              |
| `generate`    | Copy a bundled template into the current directory                             |
| `hello-world` | Sample script, template for new scripts                                        |
| `pr`          | Generate a GitHub-ready PR title + body from branch diff vs base               |
| `recap`       | Summarize opencode.db activity in bullets                                      |

### ask

```bash
uv run python main.py ask "what is the capital of France?"
uv run python main.py ask -n 3 what happened today?
```

```text
$ uv run python main.py ask what is the capital of France?
Paris is the capital of France.
```

### pr

Diffs the current branch against `--base` (default `main`) via `git diff main...HEAD`, plus `git log`, `git status`, and `git diff --stat`, then asks the model for a title + `## Summary` / `## Changes` / `## Testing` / `## Notes` body.

```bash
uv run python main.py pr
uv run python main.py pr --base develop
```

### recap

Summarizes opencode.db activity in bullets. Day boundaries are 6am Manila. DB path and model come from `~/.config/ai/config.toml` (`recap.db_path`, `openrouter.model`).

```bash
uv run python main.py recap
uv run python main.py recap yesterday
uv run python main.py recap last-week -b 3
uv run python main.py recap "january 2023" --bullets all
```

```text
$ uv run python main.py recap yesterday -b 2
- Fixed login retry handling
- Updated recap README
```

## Structure

| Path       | What                                                                                         |
| ---------- | -------------------------------------------------------------------------------------------- |
| `main.py`  | Dispatcher, auto-discovers `scripts/*/cli.py`                                                |
| `common/`  | Shared helpers (config, prompt, io, logging)                                                 |
| `scripts/` | One folder per command (`ask`, `commit`, `config`, `generate`, `hello_world`, `pr`, `recap`) |
| `tests/`   | Dispatcher tests                                                                             |

Every `scripts/<name>/cli.py` must define `register(subparsers, command)` + `run(args)`, and `register` must call `parser.set_defaults(_func=run)`. `run()` returns `str` (printed), `int` (exit code), or `None`.

## Add a script

```bash
cp -r scripts/hello_world scripts/my_tool
# edit scripts/my_tool/cli.py, test_cli.py, README.md
uv run pytest scripts/my_tool/test_cli.py -v
```

## Tests

```bash
uv run pytest -v            # everything
uv run pytest scripts/hello_world/test_cli.py -v
uv run pytest tests/test_main.py -v
```

Full per-command usage lives in each `scripts/<name>/README.md` in the repo.
