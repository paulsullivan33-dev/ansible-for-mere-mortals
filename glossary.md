# Glossary

**Ad-hoc command** — a one-off Ansible run from the
command line (`ansible web -m ping`), no playbook file.
For questions and emergencies.

**Become** — privilege escalation: running a task as
another user, usually root via sudo (`become: true`).

**Check mode** — `--check`: simulate a playbook run and
report what *would* change, changing nothing.

**Collection** — a packaged bundle of modules, roles, and
plugins distributed via Galaxy; the modern unit of
Ansible content beyond core.

**Control machine** — the one machine Ansible itself runs
on. Everything else is a target it reaches over SSH.

**Fact** — information gathered from a target by the
`setup` module (OS, addresses, hardware), available to
tasks as `ansible_facts[...]`.

**Galaxy** — the public library of roles and collections
(`ansible-galaxy install ...`), and the command-line tool
that fetches them and scaffolds roles.

**Group** — a named set of hosts in the inventory
(`webservers`), the usual unit of addressing and of
group_vars.

**Handler** — a task that runs only when notified by a
task that changed; the standard "restart the service if
the config changed" mechanism.

**Idempotent** — safe to repeat: running a task or
playbook again, against an already-correct target,
changes nothing and reports `changed=0`.

**Inventory** — the list of target hosts, their groups,
and how to reach them. INI or YAML.

**Jinja2** — the templating language inside templates
and `{{ ... }}` substitutions everywhere in Ansible.

**Module** — a small program managing one kind of thing
(`file`, `copy`, `apt`, `service`). A task is one module
call.

**Play / playbook** — a playbook is a YAML file of plays;
a play aims a set of tasks at a set of hosts.

**Role** — a reusable bundle of tasks, templates, files,
handlers, and variables under a standard directory
layout, invoked by name.

**Task** — one named module invocation: the unit of
`ok` / `changed` / `failed` reporting.

**Vault** — Ansible's built-in encryption for variables
and files (`ansible-vault`), unlocked by a vault
password at run time.
