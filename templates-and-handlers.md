# Templates and Handlers

**Scope:** generating config files per machine with
Jinja2 templates, and restarting things only when a
change actually requires it — the pattern at the heart
of most real playbooks.

## Templates

A **template** is a config file with placeholders,
rendered per host at run time. This guide's
[`examples/templates/app.conf.j2`](examples/templates/app.conf.j2):

```jinja
# Managed by Ansible -- edits will be overwritten
[app]
port = {{ app_port }}
host = {{ inventory_hostname }}
environment = {{ env_name | default('production') }}
```

Three Jinja2 moves, which cover most real templates:

- `{{ variable }}` — substitution, from any variable or
  fact.
- `{{ var | default('...') }}` — a filter; here, a
  fallback when `env_name` isn't set anywhere.
- (Not pictured, but next in line: `{% if %}` blocks and
  `{% for %}` loops — how one template serves machines
  with genuinely different shapes.)

The `template` module renders and copies in one step,
comparing the result against what's on the target — so a
template task is idempotent exactly when its rendered
output stops changing.

Why templates rather than one file per machine: the file
is *mostly* the same, and the sameness is the point. When
the service's config format changes, you edit one
template, and every machine's next run converges —
instead of hunting down fourteen hand-maintained variants.

## Handlers: the restart pattern

The eternal problem: a service must restart when its
config changes — and *only* then. Restarting on every run
breaks idempotency (and, in production, causes pointless
blips); never restarting leaves new configs unloaded.

Ansible's answer is the **handler**: a task that runs
only when notified by a task that *changed*:

```yaml
    - name: Template the app config
      ansible.builtin.template:
        src: templates/app.conf.j2
        dest: "{{ app_dir }}/app.conf"
      notify: Stamp config change

  handlers:
    - name: Stamp config change
      ansible.builtin.copy:
        content: "config changed at {{ ansible_facts['date_time']['iso8601'] }}\n"
        dest: "{{ app_dir }}/.config-stamp"
```

(The demo handler stamps a file; the real-world version
is `service: name=myapp state=restarted`.)

Behavior, all verified in the lab:

- Run 1: template changes → `RUNNING HANDLER [Stamp config change]`
  appears after all tasks, once per changed host.
- Run 2: nothing changed → the handler section doesn't
  appear at all. Silence means "no restart needed."
- Ten tasks notify the same handler → it still runs
  **once**, at the end of the play. Handlers deduplicate
  notifications the way you'd want.

Ordering guarantees worth knowing: handlers run at the
end of the play, in the order they're *defined* (not
notified), and a failed host skips its handlers unless
you force them (`--force-handlers`) — a machine that
failed mid-play probably shouldn't get its restart.

## The pair, working together

Templates + handlers are the load-bearing pattern of the
whole tool: **describe files, generate them per machine,
bounce the service precisely when the description
changed.** Master these two chapters' worth of YAML and
you can read — and write — most playbooks in the wild.
