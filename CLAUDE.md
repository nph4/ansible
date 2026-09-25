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

Add `-K` if sudo on the target requires a password.

## Structure

- `inventory/hosts` — INI inventory. All hosts connect as `riker` with `python3`. Groups: `pis` (Pi-hole boxes), `ubuntu` (become via sudo, set in `[ubuntu:vars]`), `proxmox`. `updates.yml` (apt dist-upgrade + reboot if required) targets `ubuntu`; `timezone.yml` targets all hosts.
- `playbooks/` — one playbook per task; each targets a group via `hosts:`.
- `templates/` — Jinja templates referenced from playbooks with paths relative to the playbook dir (e.g. `src=../templates/timesyncd.conf`).
- `timesyncd.conf` points NTP at the local server `192.168.88.101`, falling back to `time.cloudflare.com`.
