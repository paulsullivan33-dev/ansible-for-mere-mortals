# Troubleshooting

**Scope:** the failures you'll actually meet, symptom →
fix. Error messages are verbatim from real runs
(ansible-core 2.21.5) unless marked (documented).

## "UNREACHABLE! => { ... "unreachable": true }"

```
ghost | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ...",
    "unreachable": true
}
```

**Symptom:** a host reports UNREACHABLE while others
succeed. Ansible never got to run anything — this is
transport, not configuration.

**Fix, in order:** the machine is off/asleep; the address
or DNS name in the inventory is wrong; SSH isn't
listening (port, firewall); your key isn't authorized for
that user on that box. Test by hand: `ssh user@host` from
the control machine with the same identity Ansible would
use. Nine times out of ten the manual SSH fails too, with
a clearer message. (Documented in its SSH specifics —
the lab's unreachable case timed out at the connection
stage, exactly as shown.)

## "[ERROR]: YAML parsing failed"

```
[ERROR]: YAML parsing failed: Colons in unquoted values
must be followed by a non-space character.
Origin: /path/to/broken.yml:3:9
```

**Symptom:** the playbook won't even start;
`--syntax-check` reports the same.

**Fix:** it's the YAML, at the flagged line (usually).
The classics: a colon inside an unquoted string
(`msg: value: with colon` — quote the whole string), tabs
instead of spaces (YAML forbids tabs for indentation),
and indentation that's off by one from the mapping above
it. Ansible prints the origin line — start there, not
where your editor's complaint points.

## "'no_such_variable' is undefined"

```
fatal: [web1]: FAILED! => {"msg": "... Error rendering
template: 'no_such_variable' is undefined"}
```

**Symptom:** a task fails the moment it touches a
variable that doesn't exist.

**Fix:** the variable isn't defined in any scope Ansible
can see for that host — a typo, a group_vars file named
for the wrong group, or a role default you assumed. Print
what the host *does* have: `ansible-inventory --host web1`
for the merged inventory variables, or add a `debug: var:`
task just before the failure. (Related non-error: a
variable defined but "ignored" almost always lost a
precedence fight — usually a role `vars/` file outranking
your group_vars. Move role knobs to `defaults/`.)

## "Attempting to decrypt but no vault secrets found"

```
[ERROR]: ... Attempt to use undecryptable variable:
Attempting to decrypt but no vault secrets found.
```

**Symptom:** the playbook dies at a task using a vaulted
value.

**Fix:** you ran without `--vault-password-file` /
`--ask-vault-pass` (or the password file is wrong).
Supply it. If the password is right and it still fails,
the value was encrypted under a *different* vault
password — rekey or re-encrypt with `ansible-vault`.

## "No inventory was parsed, only implicit localhost is available"

```
[WARNING]: No inventory was parsed, only implicit localhost is available
```

**Symptom:** your command ran against localhost only, or
"No hosts matched, nothing to do."

**Fix:** Ansible found no inventory: you're in the wrong
directory (no `ansible.cfg` pointing at one), or `-i`
names a file that isn't there, or the group name in your
command is misspelled versus the inventory's. Confirm
with `ansible <group> --list-hosts` — an empty list here
is the whole diagnosis.

## The interpreter warning

```
[WARNING]: Host 'web1' is using the discovered Python
interpreter at '/usr/bin/python3.12', but future
installation of another Python interpreter could cause
a different one to be discovered.
```

**Symptom:** a warning on (nearly) every run. Harmless
today; it's Ansible saying "I guessed the target's
Python; if the target changes, the guess may change."

**Fix:** silence it properly by pinning the interpreter
per host or group (`ansible_python_interpreter=/usr/bin/python3`)
once you've confirmed it, rather than training yourself
to ignore warnings wholesale.

## Become/sudo fails on a new distribution

**Symptom:** (documented) tasks with `become: true` fail
with sudo errors on a recent distro release, while plain
SSH tasks work.

**Fix:** check whether the target ships **sudo-rs**
(the Rust rewrite several distributions adopted in the
mid-2020s) instead of classic sudo — Ansible's privilege
escalation historically assumed classic sudo's flags and
prompts. Workarounds range from an Ansible version that
handles it, to `become_method` adjustments, to installing
classic sudo on the target. Diagnose the sudo flavor
first (`sudo --version` on the target); don't guess.

## Everything is "changed", every time

**Symptom:** a task reports `changed` on runs where
nothing should have changed.

**Fix:** the task can't verify its own work. Usual
suspects: `command`/`shell` tasks (always changed by
design — add `changed_when:` or `args.creates`), a
template whose rendered output genuinely differs each
run (timestamps and random values embedded in the file
are the classic causes), or a module fighting a
hand-edit that keeps coming back. See
[Idempotency](idempotency.md).
