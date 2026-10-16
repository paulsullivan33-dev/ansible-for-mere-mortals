# Running a Fleet

**Scope:** the jump from "two web servers" to "all my
machines" — addressing subsets, taming blast radius, and
running tasks as root. Fleet-scale SSH details are marked
(documented) where this guide's local-connection lab
couldn't exercise them.

## Addressing subsets

Every Ansible command takes a **pattern**:

```
ansible webservers -m ping            # a group
ansible web1 -m ping                  # one host
ansible 'webservers:!web2' -m ping    # group minus a host
ansible 'webservers[0]' -m ping       # first host only
ansible-playbook site.yml --limit web1
```

`--limit` is the safety valve you reach for most: test
the change on one machine, read the recap, then run the
rest. The pattern syntax is the same for ad-hoc commands
and playbooks.

## Blast radius, managed

- **`serial:`** on a play — apply to N machines at a
  time (a number, or a percentage), finishing each batch
  before starting the next. The rolling-update pattern:
  if batch one melts, the rest never started. (Documented
  in mechanics; the lab's two hosts don't make batches
  interesting.)
- **`max_fail_percentage:`** — stop the play if more than
  this share of hosts fail. Circuit breaker for the day
  something's wrong with the playbook, not the machines.
- **Check mode first** — covered last chapter; on a
  fleet it's not optional politeness, it's procedure.

## Privilege: become

Most real configuration needs root. `become: true` (on a
play, or per task) runs the module via **sudo** on the
target (documented — the lab ran as its own user locally):

```yaml
- name: Configure the fleet
  hosts: all
  become: true
```

with `--ask-become-pass` if your sudo needs a password,
or passwordless sudo configured for your account on the
targets. Ansible's prompts and failures around sudo are
plain in the [Troubleshooting](troubleshooting.md)
chapter, including the current gotcha: distributions that
ship **sudo-rs** in place of classic sudo can break
Ansible's privilege escalation until versions catch up
— if `become` fails oddly on a brand-new distro, check
which sudo it ships.

## Keeping the fleet definition sane

- **Groups by role, then by place**: `webservers`,
  `databases`, and (if you must) `dc_east` — a host joins
  every group it belongs to, and group_vars stack.
- **One project folder per fleet**, with `ansible.cfg`,
  inventory, and playbooks together, in git (vault the
  secrets, chapter 9). Your future self, on a new laptop,
  reclones the folder and is operational in minutes.
- **Names you'll still understand in a year.** `web1`
  beats `srv-04-final2`. Future you is a stranger; be
  kind.

## What changes at fleet scale — and what doesn't

The playbook you wrote for two hosts is the fleet
playbook. What scales is *care*: subsets before all,
serial batches before big-bang, check mode before real
mode. Ansible's model doesn't change from 2 machines to
200; only the consequences of skipping the rehearsals do.
