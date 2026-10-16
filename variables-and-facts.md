# Variables and Facts

**Scope:** how one playbook configures different machines
differently — values you set (variables) and values
Ansible discovers (facts) — plus where to put them
without making a mess.

## Variables you set

You've already met three kinds:

- **Inventory group vars** — in
  [`examples/inventory.ini`](examples/inventory.ini):
  `[webservers:vars] app_port=8080`. Every webserver gets
  it.
- **Inventory host vars** — after a hostname on its own
  line: `web1 app_port=9090` would override the group for
  web1 alone.
- **Playbook `vars:`** — the play-level block in
  `site.yml` (`app_dir`). Good for values intrinsic to
  the playbook itself.

And one built-in, **magic variables**: `inventory_hostname`
(the host's inventory name), `groups` (all groups and
their members), `hostvars` (another host's variables —
how a load balancer learns the web servers' addresses).

## Where files beat inventory lines

Group/host vars outgrow INI files fast. The grown-up
layout keeps them in YAML files Ansible finds by name:

```
group_vars/webservers.yml     # applies to the group
host_vars/web1.yml            # applies to one host
```

```yaml
# group_vars/webservers.yml
app_port: 8080
env_name: production
```

Same effect as the INI block, but with room for lists,
nested data, and comments. The examples use the INI form
so everything fits on one screen; your real project
should use the files.

## Facts: variables Ansible discovers

At the start of every play, Ansible runs `Gathering Facts`
(the task you saw in the playbook output) — the `setup`
module interrogating each machine. Results land in
`ansible_facts`:

```
"ansible_facts": {
    "ansible_distribution": "Ubuntu",
    "ansible_distribution_version": "24.04",
    ...
}
```

Use them anywhere a variable fits:

```yaml
- name: Say where I am
  ansible.builtin.debug:
    msg: "{{ inventory_hostname }} runs {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }}"
```

Facts are how a single playbook installs with `apt` on
Debian-family machines and `dnf` on RedHat-family ones,
sizes a buffer from installed RAM, or skips a task on
the one ancient box. Two practical notes:

- Facts cost time (a second or two per host). Playbooks
  that don't need them can say `gather_facts: false` —
  the demo handler in this guide uses a fact
  (`date_time`), which is why `site.yml` leaves gathering
  on.
- `ansible <host> -m setup` dumps everything a machine
  offers. Grep it before assuming a fact's name.

## Precedence, in plain terms

Ansible has a full variable-precedence ladder (20+
levels). The mortal version:

1. Command line (`-e extra=vars`) beats everything.
2. Host vars beat group vars; a more specific group beats
   a broader one.
3. Playbook `vars:` beat inventory defaults; role
   defaults lose to nearly everything (by design — they're
   *defaults*).

When a value surprises you, it is almost always an
override you forgot existed. `ansible-inventory --host
web1` prints a host's fully-merged variables — check
there before suspecting Ansible of creativity.
