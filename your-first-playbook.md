# Your First Playbook

**Scope:** a complete, real playbook — every part named,
run end to end. This guide's copy lives in
[`examples/site.yml`](examples/site.yml) and runs
unchanged.

## The playbook

```yaml
---
- name: Configure the demo app
  hosts: webservers
  vars:
    app_dir: /tmp/demoapp-{{ inventory_hostname }}
  tasks:
    - name: Create the app directory
      ansible.builtin.file:
        path: "{{ app_dir }}"
        state: directory
        mode: "0755"

    - name: Copy the notice file
      ansible.builtin.copy:
        src: files/motd.txt
        dest: "{{ app_dir }}/motd.txt"
        mode: "0644"

    - name: Template the app config
      ansible.builtin.template:
        src: templates/app.conf.j2
        dest: "{{ app_dir }}/app.conf"
        mode: "0644"
      notify: Stamp config change

    - name: Show the rendered config path
      ansible.builtin.debug:
        msg: "Config for {{ inventory_hostname }} written to {{ app_dir }}/app.conf"

  handlers:
    - name: Stamp config change
      ansible.builtin.copy:
        content: "config changed at {{ ansible_facts['date_time']['iso8601'] }}\n"
        dest: "{{ app_dir }}/.config-stamp"
        mode: "0644"
```

## Anatomy, top to bottom

- A **play** — one `hosts:` block: *which machines*, and
  the tasks to apply to them. A playbook can hold several
  plays (web servers first, then databases).
- **`vars:`** — play-level variables. `app_dir` is built
  from `inventory_hostname`, a built-in variable holding
  each machine's inventory name — so web1 and web2 get
  *different* directories from *one* line. Variables get
  a full chapter next.
- A **task** — a name (write it like a sentence a human
  would say; it appears in the output) plus exactly one
  module call:
  - `file` — directories and files exist (or don't), with
    modes.
  - `copy` — a file from the control machine lands on the
    target, as-is.
  - `template` — like copy, but the file is a Jinja2
    template filled in per machine (next chapters).
  - `debug` — prints a message. Trivial, and the first
    tool you reach for when learning.
- **`{{ ... }}`** — Jinja2 substitution: anywhere it
  appears, the variable's value is spliced in before the
  task runs.
- **`notify:` / `handlers:`** — "if the config actually
  changed, also do this, once, at the end." Handlers get
  their own chapter with templates.

Module names are written `ansible.builtin.file` — the
fully-qualified form. Bare `file` also works; the long
form says unambiguously *whose* module you mean when
collections enter the picture.

## Run it

```
ansible-playbook site.yml
```

```
PLAY [Configure the demo app] **************************************************

TASK [Gathering Facts] *********************************************************
ok: [web1]
ok: [web2]

TASK [Create the app directory] ************************************************
changed: [web1]
changed: [web2]
...
PLAY RECAP *********************************************************************
web1                       : ok=6    changed=4    unreachable=0    failed=0
web2                       : ok=6    changed=4    unreachable=0    failed=0
```

Reading the output: every task reports per host —
`ok` (already right), `changed` (Ansible made it right),
or `failed`. The **PLAY RECAP** is the scoreboard: six
tasks OK'd (including the automatic fact-gathering),
four changed something, nothing unreachable, nothing
failed. The file each machine ended up with:

```
# Managed by Ansible -- edits will be overwritten
[app]
port = 8080
host = web1
environment = production
```

`port` came from the inventory's group vars,
`host` from each machine's own name, `environment` from a
default in the template. One playbook, two machines,
correctly different results — and if you run it again,
*four* becomes *zero*. That's the next chapter.
