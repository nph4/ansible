# CLAUDE.md

`README.md` is the source of truth for this repo — read it first. This file holds Claude's working notes: conventions, gotchas, and things to remember while making changes. Don't duplicate README content here.

## Keeping docs current

- When a change alters behavior, usage, inventory groups, or playbooks, update `README.md` in the same change.
- Add notes here only for things that help with editing the repo but don't belong in user-facing docs.

## Working notes

- Validate changes with `--syntax-check` and `ansible-inventory --graph`; there's no lint or test tooling, and playbooks can't be run against real hosts from here (the control key only works from the nelson-nuc container).
- Plays set `become: true` themselves — don't add `ansible_become` to the inventory.
- Don't list `quarks.lan` alongside `quark-vm.lan`; they're the same host.
- Never add nelson-nuc to `[komodo_periphery]`.
- Template `src:` paths are relative to the playbook dir (`../templates/...`).
