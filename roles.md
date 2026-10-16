# Roles

**Scope:** packaging a chunk of playbook — tasks,
templates, files, variables — into a named, reusable unit
with a standard layout. Roles are how Ansible work gets
shared between projects and teams.

## The problem roles solve

Your "set up a web server" logic lives in one project's
playbook. A second project needs it too. Copy-paste gives
you two diverging versions and a slow-motion mess. A
**role** is that logic extracted into a bundle with a
name: any playbook can declare `roles: [webserver]` and
get the whole behavior, with its variables as the knobs.

## The standard layout

`ansible-galaxy role init` generates the skeleton (run
for real in the lab):

```
roles/demoapp/
├── tasks/main.yml        # the tasks (the only part that's required in practice)
├── handlers/main.yml
├── templates/            # .j2 files, referenced by bare name
├── files/                # files for copy, referenced by bare name
├── vars/main.yml         # role variables (strong — hard to override)
├── defaults/main.yml     # default variables (weak — meant to be overridden)
└── meta/main.yml         # author, description, dependencies on other roles
```

The demo role's whole job, in
[`examples/roles/demoapp/tasks/main.yml`](examples/roles/demoapp/tasks/main.yml):

```yaml
---
- name: Role task - drop a marker file
  ansible.builtin.copy:
    content: "installed by the demoapp role\n"
    dest: "/tmp/demoapp-{{ inventory_hostname }}/role-marker.txt"
    mode: "0644"
```

## Using a role

```yaml
---
- name: Use a role
  hosts: webservers
  roles:
    - demoapp
```

```
ansible-playbook roleplay.yml
```

```
PLAY RECAP *********************************************************************
web1                       : ok=2    changed=1    unreachable=0    failed=0
web2                       : ok=2    changed=1    unreachable=0    failed=0
```

`changed=1` per host: the marker file landed on both.
That's the whole invocation — the role brought its tasks
with it, found its own files/templates by name, and
blended into the play as if written there.

## defaults vs vars: the one subtlety that matters

Both files hold role variables; they differ in
*precedence* (see [Variables and facts](variables-and-facts.md)):

- **`defaults/main.yml`** — weak. Anything anywhere can
  override them. This is where a role publishes its knobs:
  "I install nginx with worker_processes=2 *unless you
  say otherwise*."
- **`vars/main.yml`** — strong. Overrides most other
  sources. For the role's internal constants that callers
  shouldn't casually change.

Putting a knob in `vars/` when callers need to tune it is
the classic role-authoring mistake; it surfaces as
"your variable is being ignored," which is a
[Troubleshooting](troubleshooting.md) entry in every
other tool too.

## Finding other people's roles

**Ansible Galaxy** is the public role (and collection)
library, and `ansible-galaxy install <name>` fetches into
your project. Treat Galaxy roles like any code you
download: read the tasks before running them — a role
runs with your SSH credentials and your sudo. Well-kept
roles from known authors are a genuine shortcut;
abandoned ones are someone else's 2019 opinions frozen in
YAML. For a mortal fleet, one more honest option exists:
skip roles until a playbook is genuinely reused. Roles
are packaging, and packaging before reuse is ceremony.
