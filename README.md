# cloud-dev-vps

Ansible playbook to setup cloud VPS for development.
Installs claude code to run in bypass permissions mode.
Also installs uv, nvm, and latest stable node.js using nvm, also installs pnpm.

Sets up nftables (allows only ssh from everywhere, everything else is blocked).

Includes `add-repo` script: takes repo name as arg (assumes username titarenko), creates deploy key, adds custom ssh config, then prints public key and waits for user to add it to github, then clones repo.