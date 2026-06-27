# cloud-dev-vps

Ansible playbook to provision a cloud VPS for development.

## What it installs

- Claude Code (bypass permissions mode)
- uv, nvm, and latest stable Node.js (via nvm)
- pnpm

## Network setup

Configures nftables firewall with a deny-all-inbound baseline. SSH is allowed from anywhere; all other inbound traffic is blocked.

## add-repo script

Interactive script to set up a repository on the VPS:
- Takes a repo name as argument (GitHub username is in ansible vars)
- Generates a deploy key
- Adds a custom SSH config entry
- Prints the public key and waits for you to add it to GitHub
- Clones the repository