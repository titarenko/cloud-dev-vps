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
ansible-playbook -i <inventory> site.yml
```

## Conventions

- Target user/host variables belong in inventory or `group_vars/`
- Firewall rules live in the nftables role; the baseline policy is deny-all inbound except SSH
- The `add-repo` script is interactive (it pauses for the user to add the deploy key on GitHub before cloning)
