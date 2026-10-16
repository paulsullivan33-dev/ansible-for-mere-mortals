# Idempotency

**Scope:** Ansible's seatbelt — why a proper playbook is
safe to run twice, how to recognize one that isn't, and
what to do about it.

## The property

The same playbook, run again immediately:

```
PLAY RECAP *********************************************************************
web1                       : ok=5    changed=0    unreachable=0    failed=0
web2                       : ok=5    changed=0    unreachable=0    failed=0
```

`changed=0` on every host. The playbook didn't "do
nothing" — it *verified* everything: directory present,
files present with the right content and modes, config
current. Nothing needed fixing, so nothing was touched.
Run it a hundred times; the hundredth looks like the
second.

Compare a shell script doing the same job:
`mkdir /srv/app` fails the second time ("File exists").
`echo 'port=8080' >> app.conf` *appends a duplicate line*
the second time. Shell scripts describe steps; steps
have side effects; side effects accumulate. Playbooks
describe state; state either matches or gets corrected.

## Why it matters more than it sounds

- **Recovery is "run it again."** A playbook interrupted
  halfway — network blip, laptop closed — leaves machines
  half-configured. With idempotent tasks, you just rerun;
  completed tasks verify-and-skip, the rest proceed.
- **Drift gets healed, not documented.** Someone edited
  a config by hand? Next run notices the file's contents
  differ and puts it back (that's the `changed` you'll
  see, and it's a feature).
- **Midnight confidence.** "Will this break something if
  I run it twice?" is the wrong question to be asking
  during an incident. Idempotent playbooks make the
  answer boring: no.

## How modules achieve it

Each module knows its resource's current state and the
desired state, and acts only on the difference. `file`
stats the path first. `copy` compares checksums before
transferring. `apt` asks the package manager what's
installed. The work is in the modules, which is why
"prefer a module over a shell command" is Ansible's
first commandment.

## The honest exceptions

- **`command` / `shell` / `raw` can't know.** A raw
  command's effect is opaque, so it reports `CHANGED`
  every run (you saw this in
  [Ad-hoc commands](ad-hoc-commands.md)) and re-executes
  every time. Sometimes that's fine (read-only commands).
  When it isn't, give Ansible the missing knowledge:
  `args: creates: /path/marker` ("skip if this file
  exists") or `changed_when: false` for pure queries.
- **A task can be *wrong* idempotently.** Declaring the
  wrong desired state gets you the wrong state, reliably,
  on every machine, forever. Idempotency makes execution
  safe; it doesn't make decisions right. That's what
  [check mode](check-mode-and-debugging.md) is for.
- **Restarts always "change."** A `service state=restarted`
  task restarts, every run, by definition. The pattern
  for "restart only when the config changed" is handlers
  — [next chapters](templates-and-handlers.md).

## The habit

Before any playbook earns trust: run it twice in a row.
First run converges, second run must say `changed=0`. A
task that changes on the second run is telling you it
doesn't understand its own resource — find out why before
that task meets a machine you care about.
