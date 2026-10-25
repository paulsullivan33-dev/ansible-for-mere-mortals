# Ansible for Mere Mortals

**Ansible** is a tool for making many computers match a
description you wrote down. Instead of SSH-ing into each
machine and typing the same commands, you describe the
end state — this package installed, that config file with
these contents, this service running — and Ansible makes
it so, on ten machines as easily as one. It needs nothing
installed on the target machines except SSH and Python,
and it changes nothing when everything already matches.

This guide assumes you can use a terminal and have SSH-ed
into a machine before. Everything Ansible-specific —
inventories, playbooks, modules, idempotency — is
explained from zero.

## What this guide covers

- [**What Ansible is**](what-ansible-is.md) — agentless,
  declarative, idempotent: the three ideas that explain
  everything else
- [**Inventory and config**](inventory-and-config.md) —
  telling Ansible which machines exist
- [**Ad-hoc commands**](ad-hoc-commands.md) — one-off jobs
  across every machine, no playbook required
- [**Your first playbook**](your-first-playbook.md) —
  plays, tasks, and modules, run end to end
- [**Idempotency**](idempotency.md) — the seatbelt: safe
  to run twice, by design
- [**Variables and facts**](variables-and-facts.md) — one
  playbook, many machines, each configured its own way
- [**Templates and handlers**](templates-and-handlers.md) —
  config files generated per machine, services restarted
  only when they should be
- [**Roles**](roles.md) — playbooks grown up: reusable,
  shareable bundles
- [**Secrets with Vault**](vault-secrets.md) — passwords
  in git, encrypted properly
- [**Check mode and debugging**](check-mode-and-debugging.md) —
  see what *would* change before it changes
- [**Running a fleet**](running-a-fleet.md) — groups,
  subsets, rolling changes, and privilege escalation
- [**Troubleshooting**](troubleshooting.md) — symptom →
  fix, with the real error messages
- [**Glossary**](glossary.md) — the terms, collected in
  one place

## What it doesn't cover

Ansible's commercial side (AWX/Tower automation
controller), writing your own modules and plugins,
Windows targets in depth, and using Ansible as a
network-switch configuration tool. This guide is the
Linux-server core that everything else builds on.

## How to use this guide

Every command and playbook was run for real with
ansible-core 2.21.5 — including the shipped files in
[`examples/`](examples/), which run unchanged. The lab
uses two pretend servers reached over a local connection
so it works on one machine; against real servers the same
playbooks travel over SSH, and SSH-specific operational
details that weren't run here are marked **(documented)**.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Terraform for Mere Mortals](https://paulsullivan33-dev.github.io/terraform-for-mere-mortals/)
- [SSH for Mere Mortals](https://paulsullivan33-dev.github.io/ssh-for-mere-mortals/)
- [systemd for Mere Mortals](https://paulsullivan33-dev.github.io/systemd-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/ansible-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/ansible-for-mere-mortals/blob/main/LICENSE-CODE).
