# Secrets with Vault

**Scope:** keeping passwords and keys *in* your Ansible
project — and therefore in git — without keeping them in
plaintext. Ansible Vault encrypts values and whole files
with a password.

## The shape of it

`ansible-vault` encrypts; at playbook run time, you supply
the vault password and Ansible decrypts in memory. The
encrypted form of a single value, produced for real:

```yaml
api_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          30393563343364396365363266626531366231643438323334...
```

That block can sit in a variable file committed to git.
The ciphertext is safe to share; the **vault password**
is now the secret, and it lives outside the repo.

## Encrypting

One value, for a variable file (its output is what you
see above):

```
ansible-vault encrypt_string --stdin-name api_key \
    --vault-password-file .vaultpass
```

(type the secret, Ctrl-D). Or a whole file — useful when
a `host_vars` file is *all* secrets:

```
ansible-vault encrypt group_vars/all/secrets.yml
```

Companion verbs: `view` (read without decrypting to
disk), `edit` (decrypt → editor → re-encrypt), `rekey`
(change the vault password). The password itself can come
from a prompt (`--ask-vault-pass`), a file
(`--vault-password-file`, what scripts use — protect it
like the secret it is: `chmod 600`, never committed), or
a script that fetches it from a password manager.

## Using a vaulted value

Reference it like any variable. The lab's playbook loads
the variable file and (to prove decryption worked) prints
the secret's first characters:

```
ansible-playbook vaultplay.yml --vault-password-file .vaultpass
```

```
"msg": "The API key starts with: s3cret..."
```

(In real playbooks the secret goes into a module
argument, not a debug print — secrets in logs have a way
of staying in logs.)

Run the same playbook *without* the password and Ansible
fails the moment the value is needed — verbatim:

```
[ERROR]: ... Attempt to use undecryptable variable:
Attempting to decrypt but no vault secrets found.
```

That failure is the system working: no password, no
plaintext, no exceptions.

## Rules for mortals

1. **The vault password is the real secret.** Store it
   in your password manager and/or on paper with your
   important documents — the same discipline as the
   backup-password chapter of the sister guide. Lose it
   and the encrypted values are gone in the same way a
   backup repository's would be: permanently.
2. **Encrypt values, not whole projects.** Vault whole
   files only when they're all-secret; mixed files become
   unreadable in code review and ungreppable forever.
3. **`encrypt_string` output goes in variable files**,
   so `git blame` still shows *when a key rotated* without
   showing the key.
4. A vault password file used by automation is a standing
   credential on the control machine. That's the same
   trade-off as every automation secret everywhere; make
   it consciously.
