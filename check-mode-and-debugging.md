# Check Mode and Debugging

**Scope:** answering "what would this do?" before it does
it, and finding out why when it does something else.

## Check mode: the dry run

`--check` runs the playbook in simulation: modules report
what *would* change and change nothing.

The lab's proof: with the inventory's `app_port` edited
from 8080 to 9090 on a scratch copy, `--check --diff`
predicted the template change line by line —

```
ansible-playbook site.yml -i inventory-newport.ini --check --diff
```

```
-port = 8080
+port = 9090
changed: [web1]
changed: [web2]
```

— and the actual file on disk still said `port = 8080`
afterwards. Predicted, diffed, not applied.

Honest limits of the simulation:

- Modules that run raw commands (`command`, `shell`)
  generally *skip* in check mode — Ansible can't simulate
  arbitrary shell — so a check run can be quieter than
  the real one.
- Later tasks may depend on earlier changes (a task that
  reads a file an earlier task creates). In check mode
  the file never gets created, so downstream tasks can
  fail in simulation yet pass live. A check-mode failure
  is information, not verdict.
- Pair `--check` with `--diff` for file tasks; without
  the diff you get "something would change" but not what.

Discipline: any playbook that touches a machine you care
about gets a `--check --diff` run first, read by a human.
This is the moral equivalent of the backup guide's
restore drill: rehearse the dangerous thing safely.

## Seeing more

- `-v` … `-vvvv` — increasing verbosity: module arguments,
  then connection details, then SSH-level tracing. `-v`
  is the everyday one; it expands task results to include
  the module's full return values.
- `ansible-playbook --syntax-check site.yml` — parse
  only. Catches YAML and structure errors in a second,
  without touching a host (see
  [Troubleshooting](troubleshooting.md) for what its
  failures look like).
- `--list-tasks` / `--list-hosts` — what *would* run, and
  where, without running it.
- `--start-at-task "Name"` — resume a long playbook at a
  named task while developing it.

## The debug module, used properly

`debug` prints variables mid-run — the print statement
of Ansible:

```yaml
- name: What port did this host actually get?
  ansible.builtin.debug:
    var: app_port
```

Two mortal mistakes to skip: debugging with `shell: echo`
(which reports CHANGED and pollutes recaps), and leaving
noisy debug tasks in the finished playbook. The other
debugging instrument is failing deliberately: the `fail`
module with a `when:` condition turns "weird state
discovered late" into "stopped early, with a message."

## When a task fails

Ansible stops that *host* at the failed task (other hosts
continue — see `--limit` rescue patterns in
[Running a fleet](running-a-fleet.md)). Read the failure
from the bottom up: the `fatal:` line carries the module's
own message, usually precise. Before rerunning blindly,
remember the standing advice: playbooks are idempotent,
so fix the cause and rerun the whole thing. Partial
re-runs (`--start-at-task`) are for development, not for
operations.
