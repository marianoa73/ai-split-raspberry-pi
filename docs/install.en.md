# Installation guide: everything on one Raspberry Pi 4B

[Versión en español](install.md)

A guide for someone else to build the same system **on a single Raspberry Pi 4B with 8 GB** (plus a separate machine if
they want the self-hosted model). Steps marked with 🧑 **are done by a person** because they need an account, a payment, a
browser login or a decision; the rest can be run or automated.

> Each tool's installation commands change over time: always use each tool's official documentation
> and take this guide as the order and the configuration. Section 1 explains what was tested and what was not.

## 1. Prerequisites

### Check them before you start
Before installing anything, run the prerequisite check. **It changes nothing on the system**: it only reads and tells you what
is missing (`FALTA`, "missing") and what is worth a look (`aviso`, "warning"). If there is any `FALTA`, fix it and run it again; only
continue to step 2 when it says "Listo para instalar" ("ready to install").

```bash
git clone <URL> ~/ai-split-raspberry-pi        # or download the repository as a .zip
bash ~/ai-split-raspberry-pi/src/preflight.sh
```

To run it you need `bash`, `git` (or the .zip), `curl` and `python3`, which come with the systems below. Everything else
it checks is installed in step 2.

### Prerequisite list

| # | Requirement | Minimum | What checks it | How to fix it |
|---|---|---|---|---|
| 1 | Board | Raspberry Pi 4B with **8 GB** | memory ≥ 7000 MB | there is no lower-memory alternative |
| 2 | Architecture | **64-bit** (`aarch64`/`arm64`) | `uname -m` | flash the 64-bit system |
| 3 | Operating system | **Raspberry Pi OS Lite 64-bit** (Debian 12 or 13) or **Ubuntu Server 24.04 LTS arm64** | `/etc/os-release` | official Imager 🧑 |
| 4 | Free disk | **20 GB** on `/` (recommended: 32 GB or more; see below) | `df` | USB 3 SSD or a 64 GB card |
| 5 | User | regular, with `sudo` and SSH (not root) | `id`, `sudo` | step 2.1 |
| 6 | Base tools | `tmux`, `git`, `curl`, `python3` 3.9 or newer | `command -v` | step 2.3 |
| 7 | Node.js | 20 or newer (LTS) | `node -v` | step 2.4 |
| 8 | Network | Internet access and name resolution | `getent hosts` | network cable |
| 9 | Time | synchronized (HTTPS needs it) | `timedatectl` | `sudo timedatectl set-ntp true` |
| 10 | Ports 80 and 443 | free on the board | `ss -ltn` | stop the other service using them |
| 11 | Accounts 🧑 | Claude Code, Codex and OpenRouter (with a **spending cap** and privacy settings reviewed) | cannot be automated | your own subscriptions |
| 12 | Domain 🧑 | a domain or subdomain (a free dynamic DNS works) and, for access from outside, **ports 80 and 443 forwarded** on the router | cannot be automated | DNS provider and router |
| 13 | Self-hosted model (optional) | separate machine with 32 GB of RAM or more, on the same network | cannot be automated | see [`local-model.md`](local-model.md) (Spanish) |

Items 11 and 12 cannot be verified by the script: they are your own decisions and paperwork.

### Versions this guide was written with
They come from the author's real setup (Raspberry Pi 4B, Raspberry Pi OS based on **Debian 13 "trixie"**, arm64, kernel 6.x):
**Node.js 24.19**, **tmux 3.5a**, **Python 3.13**, **k3s v1.35**. Caddy comes from its official repository; pin the
version you install and write it down. With very different versions the steps may change: the official documentation rules.

### Disk space
- **Measured** on one of the author's boards (58 GB card, system and k3s with Open WebUI): 25 GB used in total; k3s data
  took about 225 MB, not counting container images.
- **Estimated** for this guide: system 3–5 GB, k3s and the Open WebUI image 3–5 GB, AI tools and Node 1–2 GB,
  repository and logs 1–2 GB. That is why the minimum is 20 GB and 32 GB or more is recommended. If you also keep backups
  or documents on the board, add them.

> This guide describes an **all-on-one-board** installation, but the author currently runs it split across two Raspberry Pis. The check,
> the scripts and the tests were run; what was **not** done is a complete end-to-end installation on a single
> board with everything running together. If you do it, measure the memory and tell us what to adjust (see [`limitations.md`](limitations.md), Spanish).

## 2. Prepare the board

1. 🧑 Flash the system to the SSD (or the card) with the official Imager; enable SSH and create the user.
2. Update: `sudo apt update && sudo apt full-upgrade -y`.
3. Install the basics: `sudo apt install -y tmux git python3 curl ufw`.
4. Install Node.js (LTS version) from Node's official repository, because the AI tools need it.
5. Firewall: `sudo ufw default deny incoming && sudo ufw allow ssh && sudo ufw allow 80,443/tcp && sudo ufw enable`.
6. Clone this repository: `git clone <URL> ~/ai-split-raspberry-pi`.
7. Copy the configuration: `mkdir -p ~/.config/ai-split && cp ~/ai-split-raspberry-pi/examples/ai-split.env.example ~/.config/ai-split/ai-split.env` and edit it (user, domain, sessions).

## 3. Install the three assistants

1. Install each tool following its official documentation.
2. 🧑 Log in to each one (it opens the browser or asks for a code). **Each tool stores its own keys; do not put them in `ai-split.env`.**
3. Recommended: create one Linux user per assistant (`sudo adduser --disabled-password svc-builder`) and log in to
   each tool with the matching user, so they do not share files or credentials.
4. Start the consoles in `tmux`:
   ```bash
   set -a; . ~/.config/ai-split/ai-split.env; set +a
   ~/ai-split-raspberry-pi/src/consoles.sh start
   tmux attach -t claude     # to leave without closing it: Ctrl+B, then D
   ```
5. Start them automatically: copy [`examples/systemd/ai-consoles.service`](../examples/systemd/ai-consoles.service) to
   `~/.config/systemd/user/`, run `systemctl --user enable --now ai-consoles` and `sudo loginctl enable-linger $USER`.
6. Test delegation with [`src/delegate.sh`](../src/delegate.sh) (see [`getting-started.en.md`](getting-started.en.md)); `IDLE_PATTERN` must be adjusted to the tool.

## 4. Publish over HTTPS (Caddy)

1. Install Caddy from its official repository (Debian/Ubuntu package).
2. Generate the hash of the monitoring password: `caddy hash-password` and paste it into `ai-split.env` (`MONITOR_PASS_HASH`).
3. Copy [`examples/caddy/Caddyfile`](../examples/caddy/Caddyfile) to `/etc/caddy/Caddyfile` and make the service read the
   variables (`sudo systemctl edit caddy` → `[Service]` / `EnvironmentFile=/home/USER/.config/ai-split/ai-split.env`).
4. `sudo caddy validate --config /etc/caddy/Caddyfile && sudo systemctl reload caddy`.
5. Caddy obtains and renews the Let's Encrypt **HTTPS certificate** by itself. This requires the domain to point to your IP and
   ports 80/443 to reach the board. **Local network only:** use an internal name and Caddy's internal CA
   (`tls internal`), and trust that certificate on your devices.

## 5. Web monitoring

1. Create the output folder: `sudo mkdir -p /var/lib/ai-split/monitor && sudo chown $USER /var/lib/ai-split/monitor`.
2. Copy [`ai-monitor.service`](../examples/systemd/ai-monitor.service) and [`ai-monitor.timer`](../examples/systemd/ai-monitor.timer) to
   `~/.config/systemd/user/` and run `systemctl --user enable --now ai-monitor.timer`.
3. 🧑 Open `https://YOUR_DOMAIN/monitor/`, log in with the user and password and check that the consoles show up.
4. Emergency pause: `touch ~/.config/ai-split/monitor.pause`. Details in [`monitoring.md`](monitoring.md) (Spanish).

## 6. Open WebUI on k3s

1. Install k3s with the official installer (single machine) and check `sudo k3s kubectl get nodes`.
2. Edit [`examples/k8s/open-webui.yaml`](../examples/k8s/open-webui.yaml): the Ollama address (`OLLAMA_BASE_URL`) and **pin the image version**.
3. `sudo k3s kubectl apply -f examples/k8s/open-webui.yaml`. It listens on port 30080 of the board itself and Caddy publishes it.
4. 🧑 The first account that signs up becomes the administrator: create it and keep sign-up closed (the manifest already disables it).

## 7. Self-hosted model (optional, on the other machine) 🧑

1. Install Ollama and download the models (see [`local-model.md`](local-model.md), Spanish). Make it listen on the local network.
2. **Firewall on the model machine:** allow its port (11434) only from the Raspberry's IP. Never publish it to the Internet.
3. Put that address in `OLLAMA_BASE_URL` and check from Open WebUI that it lists the models.

## 8. Final check

```bash
python3 -m unittest discover -s tests -v   # router, monitor and bilingual docs tests
bash tests/test_delegate.sh                # delegation with a simulated tmux
bash tests/test_consoles.sh                # console launcher with a simulated tmux
bash tests/test_preflight.sh               # prerequisite check with simulated data
```

And by hand: the three consoles visible in `tmux ls`, the `/monitor/` page refreshing behind a password, valid HTTPS and
Open WebUI responding. Measure memory (`free -h`) with everything running and compare it with [`hardware.md`](hardware.md) (Spanish).

## 9. What is still manual (and should be)

- Logging in to each assistant and keeping the subscriptions.
- Deciding what is delegated to each one: the rule is by data risk (see [`architecture.md`](architecture.md), Spanish).
- Approving everything published outside and changing models.
- Renewing or reviewing keys, and the backups.
