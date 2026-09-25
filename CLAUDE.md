# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Home-lab Ansible playbooks, loosely based on techno-tim's launchpad repo (https://github.com/techno-tim/launchpad). No roles, collections, or lint/test tooling — just an inventory, standalone playbooks, and templates.

## Commands

`ansible.cfg` sets the inventory, so run from the repo root:

```sh
ansible-playbook playbooks/<playbook>.yml
ansible-playbook playbooks/<playbook>.yml --check --diff   # dry run
ansible-playbook playbooks/<playbook>.yml --limit <host>   # single host
ansible-playbook playbooks/<playbook>.yml --syntax-check
ansible-inventory --graph                                  # verify group parsing
```

## Authentication

Every host is managed as a dedicated `ansible` user: SSH key only, passwordless sudo via `/etc/sudoers.d/ansible`, so no `-k`/`-K`. The control node is the `ansible` container on nelson-nuc (Homelab-IaC `stacks/nelson-nuc/ansible`), which replaces the old Pi (`ansible.lan`). Its private key lives on nelson-nuc in `~/containers/ansible/ssh/` (bind-mounted as the container's `/root/.ssh`, with `known_hosts`); the public key is committed as `files/ansible_ed25519.pub`. `authorized_keys` restricts it with `from="192.168.88.101"` (container traffic is NATed to nelson-nuc's LAN IP), so the key doesn't work from anywhere else.

New hosts are set up once with `playbooks/bootstrap.yml`, run as an existing account from a machine that can already log in: `-e ansible_user=<user> -K`. Plays set `become: true` themselves; nothing enables it from the inventory.

## Structure

- `inventory/hosts` — INI inventory. `[all:vars]` defaults every host to user `ansible` and `python3`. Groups: `pis` (Pi-hole boxes), `ubuntu`. `quark-vm.lan` is a CNAME for `quarks.lan`, so list it only once. `updates.yml` (apt dist-upgrade + reboot if required) targets `ubuntu`; `timezone.yml` targets all hosts.
- `playbooks/` — one playbook per task; each targets a group via `hosts:`.
- `playbooks/docker.yml` installs Docker + compose v2 on hosts in `[docker]` (children: `komodo_periphery`) and adds `docker_user` to the docker group. `docker_user` is `CHANGEME` in `[docker:vars]` until the host's account exists; the play refuses the placeholder or a missing account. Fresh hosts get Docker's official apt repo (like nelson-nuc); hosts that already have Docker keep their engine and only get the matching compose plugin (`docker-compose-plugin` for docker-ce, Ubuntu's `docker-compose-v2` for docker.io like quark-vm).
- `playbooks/komodo-periphery.yml` deploys a standalone Komodo Periphery agent to hosts in `[komodo_periphery]`, running as `docker_user` (become, compose project in that user's `~/containers/komodo-periphery`), connecting outbound to Komodo Core on nelson-nuc over Tailscale. Imports `docker.yml` first. First run needs `-e komodo_onboarding_key=<key>` (privileged key created in Core); onboarding state is detected from `core.pub` in the `keys` volume. `komodo_version` must match Core. Never target nelson-nuc (its Periphery is inside the Core project). Background lives in the Homelab-IaC repo's `Komodo-PoC.md` / `Komodo-Migration.md`.
- `playbooks/bootstrap.yml` — one-time creation of the `ansible` user (see Authentication).
- `files/` — static files for playbooks; `ansible_ed25519.pub` is the control container's public key.
- `templates/` — Jinja templates referenced from playbooks with paths relative to the playbook dir (e.g. `src=../templates/timesyncd.conf`).
- `timesyncd.conf` points NTP at the local server `192.168.88.101`, falling back to `time.cloudflare.com`.
