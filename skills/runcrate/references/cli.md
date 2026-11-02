# Runcrate CLI commands

Copied from the CLI reference, https://www.runcrate.ai/docs/cli/reference. Run `runcrate <command> --help` for every flag; most list commands take `--json`.

| Command | Description |
|---------|-------------|
| `runcrate login` | Authenticate via browser |
| `runcrate logout` | Clear credentials |
| **Instances** | |
| `runcrate ps` | List instances (alias for `instances list`) |
| `runcrate instances list [-q]` | List instances (`-q` for IDs only) |
| `runcrate instances types [--gpu X]` | Browse GPUs and pricing |
| `runcrate instances create --name X --gpu A100` | Deploy instance |
| `runcrate instances create ... --max-runtime 4h` | Terminate 4h after it starts running (`5m` to `168h`) |
| `runcrate instances create ... --idle-stop 30m` | Terminate after 30m idle: GPU under 5%, nobody connected (`5m` to `24h`) |
| `runcrate instances auto-stop <name-or-id> --max-runtime 8h --idle-stop off` | Change auto stop on a running instance (`off` turns one off) |
| `runcrate instances info <name-or-id>` | Instance details (IP, GPU, cost, max runtime, idle stop, auto stop at, stop reason) |
| `runcrate instances status <name-or-id>` | Live status check |
| `runcrate instances delete <name-or-id>` | Terminate instance |
| **Jobs** ([guide](https://www.runcrate.ai/docs/cli/jobs)) | |
| `runcrate run --gpu A4000 --max-runtime 2h -- <cmd>` | Run a command on its own GPU machine and follow its logs |
| `runcrate run ... --detach -- <cmd>` | Submit a job and return its id |
| `runcrate run ... --code . --retries 2 --tag k=v -- <cmd>` | Upload code, retry failed machines, tag the job |
| `runcrate run ... --count 10 --max-parallel 4 -- <cmd>` | Run a group of jobs and print their ids |
| `runcrate jobs list [--status s1,s2] [--tag k=v] [--group id] [--since 24h] [--limit n] [-q]` | List jobs, newest first |
| `runcrate jobs status <id>` | Job status, attempts, exit code, cost, tags and outputs |
| `runcrate jobs logs <id> [-f]` | Print job logs (`-f` to follow until it ends) |
| `runcrate jobs wait <id> [--timeout 2h]` | Wait for a job to end and exit with its code |
| `runcrate jobs artifacts <id>` | List the files the job wrote to `$RUNCRATE_OUTPUT_DIR` |
| `runcrate jobs download <id> [path...] [-o dir]` | Download its outputs, checked against their SHA-256 |
| `runcrate jobs rerun <id> [--gpu ...] [--env K=V]` | Run a job again as a new job |
| `runcrate jobs cancel <id>` | Cancel a job and stop its machine |
| `runcrate jobs delete <id> [--yes]` | Delete a job that ended, with its logs and outputs |
| `runcrate jobs group status\|cancel <group-id> [--yes]` | A group's progress, or cancel its live jobs (asks first) |
| **Schedules** ([guide](https://www.runcrate.ai/docs/cli/schedules)) | |
| `runcrate schedule create --cron '0 2 * * *' --tz Europe/London --max-runtime 2h ... -- <cmd>` | Run a job on a repeating schedule |
| `runcrate schedule create --at '2026-10-21 09:00' --name X --gpu A100 --max-runtime 8h` | Start an instance once, later |
| `runcrate schedule list [-q] [--json]` | List schedules with their next run and last result |
| `runcrate schedule get <id>` | Schedule details |
| `runcrate schedule pause <id>` / `resume <id>` | Stop or restart future runs |
| `runcrate schedule run-now <id>` | Launch once now, paused or not |
| `runcrate schedule delete <id> [--yes]` | Delete a schedule (what it started keeps running) |
| **Access** | |
| `runcrate ssh <instance>` | Interactive shell (keyless) |
| `runcrate ssh <instance> -- <cmd>` | Run command and exit |
| `runcrate cp ./local instance:/remote` | Upload file |
| `runcrate cp instance:/remote ./local` | Download file |
| **Volumes** | |
| `runcrate volumes list` | List volumes |
| `runcrate volumes create --name X --size 100` | Create volume (GB) |
| `runcrate volumes resize <id> --size 200` | Resize volume |
| `runcrate volumes delete <id>` | Delete volume |
| `runcrate volumes regions` | Available regions |
| **SSH Keys** | |
| `runcrate ssh-keys list` | List SSH keys |
| `runcrate ssh-keys add --name X --file <path>` | Add SSH key |
| `runcrate ssh-keys delete <id>` | Delete SSH key |
| **Billing** | |
| `runcrate billing balance` | Current credit balance and runway |
| `runcrate billing usage` | Spending breakdown |
| `runcrate billing topup <amount>` | Add credits from the default card (`--idempotency-key K` to re-send one) |
| `runcrate billing auto-recharge` | Show auto-recharge |
| `runcrate billing auto-recharge set --threshold X --target Y` | Turn auto-recharge on with a threshold and target |
| `runcrate billing auto-recharge off` | Turn auto-recharge off |
| **Workspaces & Environments** | |
| `runcrate workspaces` | List workspaces |
| `runcrate workspaces switch` | Switch active workspace |
| `runcrate envs` | List environments |
| `runcrate envs switch <name>` | Switch environment |
| `runcrate envs create <name>` | Create environment |
| `runcrate envs delete <name>` | Delete environment |
| **Other** | |
| `runcrate templates` | List OS templates |
| `runcrate config show` | Show CLI configuration |
| `runcrate version` | Show version |
| `runcrate update` | Self-update |
