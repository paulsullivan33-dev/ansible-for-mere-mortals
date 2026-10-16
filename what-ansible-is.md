# What Ansible Is

**Scope:** the three ideas under everything else in this
guide. Understand these and Ansible stops being syntax
and becomes obvious.

## The problem it solves

You have five servers. A config file needs a line changed
on all of them. The manual way: SSH in, edit, restart,
repeat five times — five chances to typo, forget one, or
leave machine three slightly different from the others.
Multiply by every change, forever. Machines drift apart;
nobody can say what any of them *should* look like.

Ansible's answer: write the desired state down once, in a
file, and let a program bring every machine to it.

## Idea 1: agentless

There's no Ansible software to install and babysit on the
machines being managed. Ansible runs on **one** control
machine (your laptop, a jump box) and reaches the targets
over **SSH**, using the **Python** that's already on
nearly every Linux system. The target needs no account in
some Ansible cloud, no daemon, no open port beyond SSH.

The trade-off is honest: SSH + Python means Ansible isn't
the tool for managing machines you can't SSH to, and very
large fleets feel the cost of thousands of SSH
connections. For mortal fleets — a handful to a few
hundred machines — agentless is almost pure win.

## Idea 2: declarative

A shell script says *do these steps*. An Ansible
**playbook** says *this is how it should be*:

```yaml
- name: The app directory exists
  ansible.builtin.file:
    path: /srv/app
    state: directory
```

That's not "run mkdir" — it's a claim about the world.
Whether Ansible creates the directory, fixes its
permissions, or does nothing depends on what it finds.
You describe destinations, not journeys.

## Idea 3: idempotent

**Idempotent** means: running it twice has the same
effect as running it once. A correct playbook run against
an already-correct machine reports `changed=0` and
touches nothing. This sounds like a nicety until you're
the one running a change against production at midnight:
idempotency is what makes "just run it again, it might
fix itself" a safe sentence. It's important enough to
get [its own chapter](idempotency.md).

## The vocabulary, in one breath

- **Module** — a small program that manages one kind of
  thing (`file`, `copy`, `template`, `apt`, `service`,
  `user`...). Hundreds ship with Ansible.
- **Task** — one module invocation with a name: "the app
  directory exists."
- **Playbook** — a YAML file of tasks, grouped into
  **plays**, each play aimed at a set of machines.
- **Inventory** — the list of your machines and how
  they're grouped.
- **Facts** — information Ansible gathers from each
  machine (OS, IP addresses, disk sizes) for your tasks
  to use.

Everything in this guide is those five words in different
arrangements.
