---
name: runcrate
description: Launch and run GPU work on Runcrate through its MCP tools. Use when the user asks to rent or launch a GPU, run a training, fine-tuning, inference or benchmark command on a GPU, check GPU prices, follow a job's logs, schedule a GPU run, move files to or from a GPU machine, manage volumes or the firewall, or check Runcrate spend and budgets.
---

# Runcrate

Runcrate is a GPU cloud you drive in plain words. The `runcrate` MCP server gives you 46 tools on the user's own account. The user signed in through the browser and picked one environment; you act only in that environment, and its monthly budget caps what you can spend.

## Pick the right shape of work

- **One command that ends** (a training run, a fine-tune, a benchmark, a script): use `run_job`. It runs the command on its own fresh GPU machine under bash in `/workspace`, and the machine stops, with its billing, as soon as the command exits. Follow it with `get_job_logs` (pass the returned cursor as `after`, `follow: true` to wait for new output) until `done` is true, then `get_job` for the exit code and the final cost.
- **A machine the user keeps working on** (a notebook, a server, interactive debugging): use `create_instance`, then `ssh_execute`, `file_upload` and `file_download`. The user can open a terminal on it with `runcrate ssh <name>`.
- **Later or on a repeat**: `create_schedule` with `run_at` (once) or `cron` and a timezone.

## Spend rules

1. Check the price first: `list_gpu_types` returns live prices per GPU-hour. Tell the user the hourly rate before you launch anything.
2. Always set `max_runtime_minutes` to a realistic upper bound. On an instance left unattended, also set `idle_shutdown_minutes`. When either fires the machine is terminated, not paused, so copy results off first.
3. When work on an instance is done, stop it with `delete_instance`. Billing stops at once.
4. `get_environment_budget` shows the cap, what is spent this month and what is left. At 100% launches fail and running machines stop. Only a workspace owner or admin changes a budget, in the dashboard.
5. Ask before destructive calls: `delete_instance`, `delete_volume`, `delete_environment`, `remove_firewall_rule`, `cancel_job`.

## Files and storage

- `file_upload` and `file_download` move small files (up to 10 MB), such as a script or a results file.
- A volume (`create_volume`) keeps files when a machine stops. Pass its id as `storage_id` to `create_instance` to boot with its files.

## Firewall

New environments allow only SSH (port 22). `add_firewall_rule` opens a port on every machine in the environment; set `https: true` on a single TCP port to get an HTTPS link to a web app or API. Only owners and admins can change it.

## When something fails

- `BUDGET_REACHED`: the environment spent its monthly budget. Tell the user; do not retry.
- Not signed in, or the wrong environment: ask the user to reconnect Runcrate in their client and pick the environment again.
- Docs: https://runcrate.ai/docs/mcp/tools
