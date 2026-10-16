# Inventory and Config

**Scope:** telling Ansible what machines exist and how to
reach them. Nothing runs until this file is right, and
every "Ansible can't see my server" problem lives here.

## The inventory

An **inventory** lists your machines, usually grouped by
what they're *for*. The simple INI format (from this
guide's [`examples/`](examples/inventory.ini)):

```ini
[webservers]
web1 ansible_host=127.0.0.1 ansible_connection=local
web2 ansible_host=127.0.0.1 ansible_connection=local

[webservers:vars]
app_port=8080
```

Read it as: a group called `webservers` containing two
hosts. Variables after a hostname apply to that host;
the `[group:vars]` block applies to the whole group —
`app_port=8080` here becomes usable on both machines
(see [Variables and facts](variables-and-facts.md)).

Two lab-isms to translate for real life:

- `ansible_host=127.0.0.1` would normally be the server's
  real address or DNS name (or omitted — the hostname
  itself is used).
- `ansible_connection=local` runs tasks on the control
  machine itself, which is how this guide's two "servers"
  live on one box so you can reproduce everything. Real
  servers use the default connection: SSH. Same
  playbooks, same modules — only the transport differs.

A YAML inventory says the same thing with more structure;
INI is fine until it isn't (nested groups are where YAML
starts winning).

## Check what Ansible sees

Before running anything real, ask:

```
ansible webservers --list-hosts
```

```
  hosts (2):
    web1
    web2
```

If a machine is missing here, no playbook will find it
either. If groups are nested or overlapping, this command
(and `ansible-inventory --graph`) is how you confirm the
shape matches your mental model.

## ansible.cfg

A small config file next to your inventory saves flags on
every command:

```ini
[defaults]
inventory = inventory.ini
host_key_checking = False
retry_files_enabled = False
```

- `inventory` — so you stop typing `-i inventory.ini`.
- `host_key_checking = False` — don't stop to ask "are
  you sure you want to connect" for new hosts. Fine for a
  lab and a home fleet; on anything serious, pre-seed
  known_hosts instead and leave checking on. (Documented —
  the lab never makes a real SSH connection.)
- `retry_files_enabled = False` — don't scatter `.retry`
  files after failed runs.

Ansible reads `ansible.cfg` from the current directory
first, so one project folder = one self-contained setup:
config, inventory, playbooks. That's the layout the
examples in this guide use, and the one to copy.

## The first contact

```
ansible webservers -m ping
```

```
web1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

`ping` is an Ansible module, not the network command: it
proves Ansible can connect, find Python on the target,
and run code there. `SUCCESS` + `pong` from every host is
the green light for everything after this chapter. If it
fails instead, the [Troubleshooting](troubleshooting.md)
chapter has the usual suspects word for word.
