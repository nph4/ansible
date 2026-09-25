# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Home-lab Ansible playbooks, loosely based on techno-tim's launchpad repo (https://github.com/techno-tim/launchpad). No roles, collections, `ansible.cfg`, or lint/test tooling — just an inventory, standalone playbooks, and templates.

## Commands

There is no `ansible.cfg`, so always pass the inventory explicitly from the repo root:

```sh
ansible-playbook -i inventory/hosts playbooks/<playbook>.yml
ansible-playbook -i inventory/hosts playbooks/<playbook>.yml --check --diff   # dry run
ansible-playbook -i inventory/hosts playbooks/<playbook>.yml --limit <host>   # single host
ansible-playbook -i inventory/hosts playbooks/<playbook>.yml --syntax-check
ansible-inventory -i inventory/hosts --graph                                  # verify group parsing
```

Add `-K` if sudo on the target requires a password. `ansible_user='riker'` in `[all:vars]` is refused on nelson-nuc and quark-vm (their users are `nelson` and `quark`), so set `ansible_user` per host where needed.

## Structure

- `inventory/hosts` — INI inventory. `[all:vars]` defaults every host to user `riker` and `python3` (see the note above about `riker`). Groups: `pis` (Pi-hole boxes), `ubuntu` (become via sudo, set in `[ubuntu:vars]`), `proxmox`. `updates.yml` (apt dist-upgrade + reboot if required) targets `ubuntu`; `timezone.yml` targets all hosts.
- `playbooks/` — one playbook per task; each targets a group via `hosts:`.
- `playbooks/docker.yml` installs Docker + compose v2 on hosts in `[docker]` (children: `komodo_periphery`). Fresh hosts get Docker's official apt repo (like nelson-nuc); hosts that already have Docker keep their engine and only get the matching compose plugin (`docker-compose-plugin` for docker-ce, Ubuntu's `docker-compose-v2` for docker.io like quark-vm). Needs sudo — neither existing host has passwordless sudo, so pass `-K`.
- `playbooks/komodo-periphery.yml` deploys a standalone Komodo Periphery agent (compose project in `~/containers/komodo-periphery`, no sudo) to hosts in `[komodo_periphery]`, connecting outbound to Komodo Core on nelson-nuc over Tailscale. Imports `docker.yml` first, so run it with `-K`. First run needs `-e komodo_onboarding_key=<key>` (privileged key created in Core); onboarding state is detected from `core.pub` in the `keys` volume. `komodo_version` must match Core. Never target nelson-nuc (its Periphery is inside the Core project). Background lives in the Homelab-IaC repo's `Komodo-PoC.md` / `Komodo-Migration.md`.
- `templates/` — Jinja templates referenced from playbooks with paths relative to the playbook dir (e.g. `src=../templates/timesyncd.conf`).
- `timesyncd.conf` points NTP at the local server `192.168.88.101`, falling back to `time.cloudflare.com`.
