# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

Ansible playbook that provisions a cloud VPS for development. Key things it sets up:

- Claude Code and Antigravity CLI (both configured to run in bypass-permissions mode)
- uv, nvm, latest stable Node.js via nvm, pnpm
- SSH hardening: keys-only authentication, no password login, root login via keys only (`prohibit-password`)
- nftables firewall: SSH allowed from everywhere with automatic brute-force protection (IPs rate-limited to 3 new connections/min, banned for 15 minutes with auto-expiry), all other inbound traffic blocked
- `add-repo` script: accepts a repo name (assumes GitHub username `titarenko`), generates a deploy key, adds a custom SSH config entry, prints the public key and waits for the user to add it to GitHub, then clones the repo

## Running the playbook

```bash
ansible-playbook playbook.yml
```

Edit `inventory.ini` with the VPS host and `group_vars/all.yml` for `dev_user` and `github_username`.

## Structure

- `roles/base/` — full package upgrade (`apt full-upgrade`), installs htop, gdu, curl, git, deploys `/etc/ssh/sshd_config.d/99-hardening.conf` (keys-only auth, no password login)
- `roles/nftables/` — installs nftables, deploys config from template with brute-force protection (dynamic IPv4/IPv6 sets, 15m auto-expiry), enables service
- `roles/dev_tools/` — installs uv, nvm, Node.js LTS, pnpm, Claude Code; deploys `add-repo` script
- `roles/dev_tools/templates/add-repo.j2` — templated with `github_username` from vars

## SSH and Firewall

SSH accepts **keys only** — no password authentication. Root login is allowed via keys (`PermitRootLogin prohibit-password`).

**Brute-force protection** uses nftables dynamic sets with automatic timeouts:
- `ssh_brute4` and `ssh_brute6` sets track source IPs that exceed 3 new SSH connections per minute (burst 5 packets)
- Offending IPs are automatically added to the set and dropped for 15 minutes
- The kernel reaps expired entries automatically — no daemon or cron job needed
- Inspect active bans: `sudo nft list set inet filter ssh_brute4`

## Conventions

- `become: true` at play level for system tasks; `become_user: "{{ dev_user }}"` for user-level installs
- nvm-based installs source `~/.nvm/nvm.sh` in each shell task since nvm is not on the system PATH
- Claude Code bypass permissions is set via `~/.claude/settings.json`, and Antigravity CLI bypass permissions is set via `~/.gemini/antigravity-cli/settings.json`; the `add-repo` script is interactive and pauses for the GitHub deploy key step
