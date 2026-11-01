---
name: runcrate
description: Rent GPUs and run GPU work on Runcrate through its MCP tools or its CLI, and keep the spend in check. Use when the user asks to check GPU or model prices or stock, rent or launch a GPU, run a training, fine-tuning, inference or benchmark command on a GPU, follow a job's logs, schedule a GPU run, start a GPU cluster, move files to or from a GPU machine, manage volumes, the firewall or HTTPS links, stop a machine, check Runcrate spend and budgets, or top up a Runcrate balance or handle an insufficient balance.
license: MIT
metadata:
  homepage: https://www.runcrate.ai
  docs: https://www.runcrate.ai/docs/getting-started/agents
---

# Runcrate

Runcrate is a GPU cloud you drive in plain words, paid from prepaid credits: no commitment, no minimum, no quota requests, and credits never expire. The listed rate is the rate, for any GPU in stock. A person can also top up without a credit card in the dashboard.

The `runcrate` MCP server and the `runcrate` CLI both act on the user's own account. The user signs in through the browser and picks one environment; you act only in that environment, and its monthly budget caps what you can spend. Never ask for a key or a password.

## Connect

Use the MCP tools when your client has them; otherwise use the CLI.

MCP server: `https://mcp.runcrate.ai/mcp` (streamable HTTP, OAuth). If the `runcrate` tools are missing, add it with the command for your client; the user signs in on first use:

```bash
claude mcp add --transport http --scope user runcrate https://mcp.runcrate.ai/mcp      # Claude Code, then /mcp
codex mcp add runcrate --url https://mcp.runcrate.ai/mcp                               # Codex
gemini mcp add --transport http --scope user runcrate https://mcp.runcrate.ai/mcp      # Gemini CLI
```

Cursor: add `{"mcpServers":{"runcrate":{"url":"https://mcp.runcrate.ai/mcp"}}}` to `~/.cursor/mcp.json`. Any other client: https://runcrate.ai/mcp.md has the steps.

CLI (macOS and Linux; Homebrew: `brew install runcrateai/tap/runcrate`; Windows: `irm https://runcrate.ai/install.ps1 | iex`):

```bash
curl -sSL https://runcrate.ai/install.sh | sh
export PATH="$HOME/.local/bin:$PATH"   # the installer may have put it there
runcrate login                         # opens the browser to sign in
```

## Check prices and stock first

Stock and prices change through the day. Read them now, never from memory, and tell the user the hourly rate before you launch anything.

- No sign-in needed: `curl -fsSL https://www.runcrate.ai/pricing.md` (every GPU in stock per GPU-hour, with the time stock was read, and every model's price) or `curl -fsSL https://www.runcrate.ai/gpus.json` (JSON: `gpus[]` with `name`, `vram`, `min_price_hr`, `available_options`, `gpu_counts`, `regions`; `available_options: 0` means sold out now).
- MCP: `list_gpu_types` (optional `gpu`, `region`, `gpu_count`) lists what you can launch now, with live prices per GPU-hour and type ids.
- CLI: `runcrate instances types` or `runcrate instances types --gpu H100`.

## Pick the right shape of work

- **One command that ends** (a training run, a fine-tune, a benchmark, a script): run it as a job. It runs the command on its own fresh GPU machine under bash in `/workspace`, and the machine stops, with its billing, as soon as the command exits.
  - MCP: `run_job` with `gpu` (such as `"H100"`, or a list to try in order such as `["H100", "A100-80GB:2"]`), `command` and `max_runtime_minutes` (1 to 10080), all required. Pass tokens in `variables`, never inside `command`. Follow it with `get_job_logs` (pass the returned cursor as `after`, `follow: true` to wait for new output) until `done` is true, then `get_job` for the exit code and the final cost.
  - CLI: `runcrate run --gpu H100 --max-runtime 2h -- python train.py` prints the job id and follows the logs. Add `--detach` to return at once, then `runcrate jobs logs <id> -f` or `runcrate jobs wait <id>`. Pass tokens with `--env HF_TOKEN=$HF_TOKEN`.
- **A machine the user keeps working on** (a notebook, a server, interactive debugging): launch an instance.
  - MCP: `create_instance` with `name`, `gpu` (or `type_id` from `list_gpu_types`), `max_runtime_minutes` (5 to 10080) and, when it may sit unattended, `idle_shutdown_minutes` (5 to 1440). Then `ssh_execute`, `file_upload` and `file_download`. `set_instance_auto_stop` extends or changes the limits later.
  - CLI: `runcrate instances create --name train-1 --gpu H100 --max-runtime 4h --idle-stop 30m`. Then `runcrate ssh train-1 -- nvidia-smi` and `runcrate cp ./train.py train-1:/workspace/train.py`. Change limits with `runcrate instances auto-stop train-1 --max-runtime 8h`. The user can open a terminal on it with `runcrate ssh <name>`.
- **Several machines as one** (a Slurm, Kubernetes or Ray cluster, or the same job on many machines): `cluster_offers`, then `create_cluster` with an `offer_id`. A cluster is bought by the machine (1, 2, 4 or 8 GPUs each); show the user `hourly_rate` for the whole cluster and `hourly_rate_per_node` per machine first. It starts whole or not at all, is billed from when every machine is up, and has no auto stop: `delete_cluster` when done. Poll `get_cluster` until `status` is `ready`. Its machines run on Ethernet, so it suits independent jobs, job arrays and Ray tasks, not training one model across machines.
- **Later or on a repeat**: `create_schedule` with `run_at` (once) or `cron` and a timezone (CLI: `runcrate schedule create`).

## Spend rules

1. Check the price first (above). Tell the user the hourly rate before you launch anything.
2. Always set a max runtime (`max_runtime_minutes`, CLI `--max-runtime`) to a realistic upper bound. On an instance left unattended, also set an idle stop (`idle_shutdown_minutes`, CLI `--idle-stop`). When either fires the machine is terminated, not paused, and its local disk is deleted, so copy results off first, or attach a volume (`storage_id`, CLI `--volume`), whose files are kept.
3. When work on an instance is done, stop it: MCP `delete_instance`, CLI `runcrate instances delete <name-or-id>`. Billing stops at once. A job still running: MCP `cancel_job`, CLI `runcrate jobs cancel <id>`. Check nothing is left running with `list_instances` and `list_jobs` (CLI `runcrate ps` and `runcrate jobs list --status running,queued`).
4. `get_environment_budget` shows the cap, what is spent this month and what is left. At 100% launches fail and running machines stop. Only a workspace owner or admin changes a budget, in the dashboard.
5. Ask before destructive calls: `delete_instance`, `delete_volume`, `remove_firewall_rule`, `delete_instance_link`, `cancel_job`, `delete_cluster`.

## Files and storage

- `file_upload` and `file_download` (CLI `runcrate cp`) move small files (up to 10 MB), such as a script or a results file.
- A volume (`create_volume`, CLI `runcrate volumes create`) keeps files when a machine stops. Pass its id as `storage_id` to `create_instance` or `run_job` to boot with its files.

## Firewall

New environments allow only SSH (port 22). `list_firewall_rules`, `add_firewall_rule`, `update_firewall_rule` and `remove_firewall_rule` manage which inbound ports every machine in the environment accepts, and from where. Only owners and admins can change them.

## HTTPS links

To reach a web app or API on an instance (listening over plain HTTP on 0.0.0.0), use `create_instance_link` with `instance_id` and `port`. It serves that port at `https://<port>-<instance>.runcrate.app` and needs no firewall rule. Pick `access`: `workspace` (the default, signed-in members of the workspace), `public` (anyone with the URL) or `ips` (only the addresses in `sources`). `list_instance_links` shows an instance's links; `delete_instance_link` removes one. Workspace owners and admins, and whoever launched the instance, can add and remove links.

## Secrets

`list_secrets` (CLI `runcrate secrets list`) shows the names of the environment's secrets (never values). Every instance and job launched there gets them as environment variables, so a command can read `$HF_TOKEN` directly. A value passed for one launch (`run_job` `variables`, CLI `--env`) wins over a secret of the same name. When a command needs a token that is not listed, ask the user to add it in the dashboard (https://runcrate.ai/dashboard/settings/secrets); never ask them to paste a value into the chat.

## Balance, top-ups and insufficient balance

- Read the balance and runway: MCP `billing_balance` (`floating_balance_usd`, `burn_usd_per_hour`, `runway_hours`); CLI `runcrate billing balance`.
- A launch the balance cannot cover fails with HTTP 402 `insufficient_credits`; an inference call fails with 402 `insufficient_balance`. Do not retry the same launch. Read the balance, then top up or tell the user.
- Top up from the workspace's saved default card: MCP `top_up` with `amount_usd` (20 to 10000); CLI `runcrate billing topup 50 --yes`. Tell the user the amount before you spend it. A retry with the same `idempotency_key` (CLI `--idempotency-key K`, printed on the first call) never charges twice.
  - `requires_action`: nothing is charged yet. Give the user `confirm_url`; a workspace owner or manager confirms it, then call again with the same key.
  - `BILLING_PERMISSION_REQUIRED`: a workspace owner or manager turns on Top-ups for this sign-in in the dashboard (API keys). Tell the user.
  - `TOPUP_CAP_REACHED` or `declined`: tell the user; do not retry.
  - No card, or the card won't go through: the user can top up without a credit card in the dashboard, under Billing, then Add credits.
- Keep long work funded: MCP `set_auto_recharge` with `enabled: true`, `threshold_usd` and `target_usd`; CLI `runcrate billing auto-recharge set --threshold 25 --target 100`.

## When something fails

- `BUDGET_REACHED`: the environment spent its monthly budget. Tell the user; do not retry.
- `insufficient_credits` or `insufficient_balance`: see the section above.
- Not signed in, or the wrong environment: ask the user to reconnect Runcrate in their client (or run `runcrate login`) and pick the environment again.

## More

- Every CLI command: [references/cli.md](references/cli.md)
- MCP tools: https://runcrate.ai/docs/mcp/tools
- Inference API (OpenAI-compatible at `https://api.runcrate.ai/v1`): https://runcrate.ai/docs/models/overview
- Everything for agents: https://www.runcrate.ai/llms.txt
