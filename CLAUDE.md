# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

Ansible playbook that provisions a cloud VPS for development. Key things it sets up:

- Claude Code (installed to run in bypass-permissions mode)
- uv, nvm, latest stable Node.js via nvm, pnpm
- nftables firewall: SSH allowed from everywhere, all other inbound traffic blocked
- `add-repo` script: accepts a repo name (assumes GitHub username `titarenko`), generates a deploy key, adds a custom SSH config entry, prints the public key and waits for the user to add it to GitHub, then clones the repo

## Running the playbook

```bash
ansible-playbook playbook.yml
```

Edit `inventory.ini` with the VPS host and `group_vars/all.yml` for `dev_user` and `github_username`.

## Structure

- `roles/nftables/` — installs nftables, deploys config from template, enables service
- `roles/dev_tools/` — installs uv, nvm, Node.js LTS, pnpm, Claude Code; deploys `add-repo` script
- `roles/dev_tools/templates/add-repo.j2` — templated with `github_username` from vars

## Conventions

- `become: true` at play level for system tasks; `become_user: "{{ dev_user }}"` for user-level installs
- nvm-based installs source `~/.nvm/nvm.sh` in each shell task since nvm is not on the system PATH
- Claude Code bypass permissions is set via `~/.claude/settings.json`; the `add-repo` script is interactive and pauses for the GitHub deploy key step
