# Ad-Hoc Commands

**Scope:** one-off jobs run straight from the command
line — no playbook file. This is Ansible as the world's
best fleet-wide remote shell, and for many people it's
reason enough to install it.

## The shape

```
ansible <group> -m <module> -a "<module arguments>"
```

`-m` names a module; `-a` passes its arguments. Skip `-m`
and it defaults to running a shell command.

## Real examples, run for real

**Is everything alive?** (from the last chapter)

```
ansible webservers -m ping
```

**One shell command, everywhere:**

```
ansible webservers -m shell -a "hostname; uptime"
```

```
web1 | CHANGED | rc=0 >>
htch-runtime
 18:20:15 up  1:35,  0 user,  load average: 1.49, 0.76, 0.53
web2 | CHANGED | rc=0 >>
...
```

Note `CHANGED`: shell commands always report changed,
because Ansible can't know what a raw command did. That
matters for idempotency — see its
[chapter](idempotency.md).

**Ask each machine about itself** — the `setup` module
gathers **facts**:

```
ansible web1 -m setup -a "filter=ansible_distribution*"
```

```
web1 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_major_version": "24",
        "ansible_distribution_release": "noble",
        "ansible_distribution_version": "24.04",
        ...
    },
    "changed": false
}
```

Unfiltered, `setup` returns hundreds of facts per machine
(OS, hardware, network, disks). Facts are how playbooks
adapt: "on Debian-family machines use `apt`, on
RedHat-family use `dnf`" is just a fact check away.

## The jobs ad-hoc is perfect for

- "Restart this service on the three sick machines, now":
  `ansible webservers -m service -a "name=nginx state=restarted"`
- "How much disk is left everywhere": `-m shell -a "df -h /"`.
- "Copy this one file out, once": `-m copy -a "src=... dest=..."`.
- "Who's logged in / what's running / which kernel":
  gather first, fix second.

## Where ad-hoc ends

The moment a command matters enough to run *again* next
month — or needs variables, conditions, templates, or a
restart-if-changed — it has outgrown the command line.
That's the exact point where it becomes a playbook, next
chapter. Rule of thumb: ad-hoc is for *questions* and
*emergencies*; playbooks are for *state*.
