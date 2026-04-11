# Topic 13: Ansible Vault

> 📍 Phase 3 — Advanced | Topic 13 of 28 | File: `13-ansible-vault.md`
> 🔗 Prev: `12-roles.md` | Next: `14-error-handling.md`

---

## 🧠 Concept Overview

Every real Ansible project handles secrets: database passwords, API keys, SSL private keys, cloud credentials. Committing these in plaintext to Git is one of the most common and serious security mistakes in infrastructure automation.

**Ansible Vault** is the built-in encryption system that lets you store secrets alongside your playbooks — encrypted at rest, decrypted transparently at runtime. You commit encrypted files to Git with confidence. Your team checks out the repo and runs playbooks. Vault handles everything in between.

Vault uses AES-256 encryption. The only thing you protect is the vault password itself — everything else (the encrypted files, the playbooks, the inventory) can live safely in version control.

---

## 📖 In-Depth Explanation

### Subtopic 13.1 — Encrypting Files, Strings, and Variables

#### The `ansible-vault` CLI

```bash
# Encrypt an entire file
ansible-vault encrypt group_vars/production/vault.yml

# Decrypt a file (writes plaintext to disk — use carefully)
ansible-vault decrypt group_vars/production/vault.yml

# View encrypted file without decrypting to disk
ansible-vault view group_vars/production/vault.yml

# Edit encrypted file in your $EDITOR
ansible-vault edit group_vars/production/vault.yml

# Re-encrypt with a new password
ansible-vault rekey group_vars/production/vault.yml

# Create a new encrypted file
ansible-vault create group_vars/production/vault.yml

# Encrypt a string inline (for embedding in YAML)
ansible-vault encrypt_string 'supersecret' --name 'db_password'
```

---

#### Encrypting whole files

The most common pattern: keep a `vault.yml` alongside `vars.yml` in each `group_vars/` directory.

```
group_vars/
├── all/
│   ├── vars.yml       ← plaintext variables
│   └── vault.yml      ← encrypted secrets
├── production/
│   ├── vars.yml
│   └── vault.yml
└── staging/
    ├── vars.yml
    └── vault.yml
```

```yaml
# group_vars/production/vars.yml  (plaintext — safe to commit)
db_host: prod-db.internal
db_port: 5432
db_name: myapp_production
db_user: myapp
# Reference vault variable by a conventional name
db_password: "{{ vault_db_password }}"
app_secret_key: "{{ vault_app_secret_key }}"
```

```yaml
# group_vars/production/vault.yml  (ENCRYPTED — run ansible-vault encrypt)
vault_db_password: "supersecretpassword123"
vault_app_secret_key: "a1b2c3d4e5f6789012345678901234567890abcd"
```

> 💡 The `vault_` prefix convention makes it obvious which variables are vault-sourced. The plaintext `vars.yml` becomes self-documenting — readers know `db_password` comes from vault without having to decrypt anything.

After encrypting:
```
# group_vars/production/vault.yml — as it appears in Git
$ANSIBLE_VAULT;1.1;AES256
66386439653236336462626566653337376131333862313332333632386163643434313266666565
3733623234643266623966313436326639306532373636650a653166383438633632353638663832
...
```

---

#### Encrypting individual strings (`encrypt_string`)

Useful when you want most of a file to be readable but need one or two values encrypted:

```bash
ansible-vault encrypt_string 'supersecret' --name 'db_password'
```

Output to paste directly into a YAML file:
```yaml
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  66386439653236336462626566653337376131333862313332333632386163643434313266666565
  3733623234643266623966313436326639306532373636650a653166383438633632353638663832
  ...
```

```yaml
# vars.yml — mix of plaintext and encrypted values
db_host: prod-db.internal
db_port: 5432
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  66386439653236336462626566653337...
app_debug: false
```

> ⚠️ `encrypt_string` values are harder to rotate (must re-encrypt every occurrence). Prefer whole-file encryption for most use cases — it's easier to `ansible-vault edit` one file than to hunt for every `!vault` string.

---

#### Running playbooks with Vault

```bash
# Prompt for vault password interactively
ansible-playbook site.yml --ask-vault-pass

# Provide vault password from a file
ansible-playbook site.yml --vault-password-file ~/.vault_pass

# Provide via environment variable
ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass ansible-playbook site.yml

# Set in ansible.cfg (most convenient for dev)
# [defaults]
# vault_password_file = ~/.vault_pass
```

```bash
# ~/.vault_pass — must be chmod 600
echo "my-vault-password" > ~/.vault_pass
chmod 600 ~/.vault_pass

# Add to .gitignore — NEVER commit this file
echo ".vault_pass" >> .gitignore
```

---

### Subtopic 13.2 — Vault IDs and Multiple Vault Passwords

In large organisations, you need different vault passwords for different environments — production secrets encrypted with a different key than staging secrets.

**Vault IDs** label encrypted content so Ansible knows which password to use:

```bash
# Encrypt with a vault ID label
ansible-vault encrypt --vault-id production@~/.vault_pass_prod group_vars/production/vault.yml
ansible-vault encrypt --vault-id staging@~/.vault_pass_staging group_vars/staging/vault.yml

# Encrypt a string with a vault ID
ansible-vault encrypt_string --vault-id production@~/.vault_pass_prod \
  'supersecret' --name 'db_password'
```

The encrypted content includes the vault ID label:
```
$ANSIBLE_VAULT;1.2;AES256;production
66386439653236...
```

```bash
# Run with multiple vault IDs — Ansible tries each until one works
ansible-playbook site.yml \
  --vault-id production@~/.vault_pass_prod \
  --vault-id staging@~/.vault_pass_staging

# Use a script to retrieve the password dynamically
ansible-playbook site.yml \
  --vault-id production@scripts/get_vault_pass.py
```

#### Vault password scripts

A vault password script is any executable that prints the password to stdout:

```python
#!/usr/bin/env python3
# scripts/get_vault_pass.py
import boto3
import sys

def get_password():
    client = boto3.client('secretsmanager', region_name='eu-west-1')
    response = client.get_secret_value(SecretId='ansible/vault-password')
    print(response['SecretString'], end='')

get_password()
```

```bash
chmod +x scripts/get_vault_pass.py
ansible-playbook site.yml --vault-id production@scripts/get_vault_pass.py
```

This pattern integrates Ansible Vault with AWS Secrets Manager, HashiCorp Vault, or any secret store — the vault password itself never touches disk.

---

### Subtopic 13.3 — CI/CD Integration Patterns for Vault

#### Pattern 1: Environment variable in CI

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Ansible
        run: pip install ansible

      - name: Write vault password file
        run: echo "${{ secrets.ANSIBLE_VAULT_PASSWORD }}" > ~/.vault_pass && chmod 600 ~/.vault_pass

      - name: Run playbook
        run: ansible-playbook -i inventory/production site.yml --vault-password-file ~/.vault_pass

      - name: Clean up vault password file
        if: always()
        run: rm -f ~/.vault_pass
```

> ⚠️ Always clean up the vault password file even if the job fails — use `if: always()` in GitHub Actions or `always` stage in GitLab CI.

---

#### Pattern 2: GitLab CI with masked variable

```yaml
# .gitlab-ci.yml
deploy:
  stage: deploy
  image: python:3.11
  before_script:
    - pip install ansible
    - echo "$ANSIBLE_VAULT_PASSWORD" > ~/.vault_pass
    - chmod 600 ~/.vault_pass
  script:
    - ansible-playbook -i inventory/production site.yml --vault-password-file ~/.vault_pass
  after_script:
    - rm -f ~/.vault_pass
  environment:
    name: production
  only:
    - main
```

---

#### Pattern 3: HashiCorp Vault integration (advanced)

For teams already using HashiCorp Vault, retrieve the Ansible vault password from HCP Vault at CI runtime:

```bash
#!/bin/bash
# scripts/get_vault_pass_from_hcp.sh
vault kv get -field=password secret/ansible/vault-password
```

```yaml
# GitHub Actions
- name: Get vault password from HashiCorp Vault
  uses: hashicorp/vault-action@v2
  with:
    url: https://vault.example.com
    token: ${{ secrets.VAULT_TOKEN }}
    secrets: |
      secret/ansible/vault-password password | ANSIBLE_VAULT_PASSWORD

- name: Run playbook
  env:
    ANSIBLE_VAULT_PASSWORD_FILE: /dev/stdin
  run: |
    echo "$ANSIBLE_VAULT_PASSWORD" | ansible-playbook site.yml \
      --vault-password-file /dev/stdin
```

---

#### Pattern 4: `no_log: true` for tasks handling secrets

Even when vault is in use, Ansible may log decrypted values in task output. Always suppress logging for tasks that touch secrets:

```yaml
tasks:
  - name: Create database user
    community.postgresql.postgresql_user:
      name: "{{ db_user }}"
      password: "{{ db_password }}"    # decrypted from vault at runtime
      state: present
    no_log: true    # prevent password appearing in logs or AWX job output

  - name: Set application secret key
    ansible.builtin.lineinfile:
      path: /etc/myapp/.env
      regexp: '^SECRET_KEY='
      line: "SECRET_KEY={{ app_secret_key }}"
    no_log: true
```

---

## 🏗️ Architecture & System Design

How Vault integrates with the rest of the Ansible stack:

```mermaid
graph TD
    A[Git Repo] --> B[Encrypted vault.yml files\nSafe to commit]
    A --> C[Plaintext vars.yml\nReferences vault_ vars]
    A --> D[Playbooks / Roles\nno secrets embedded]

    E[Vault Password Source] --> F{How stored?}
    F -->|Dev| G[~/.vault_pass file\nchmod 600]
    F -->|CI/CD| H[CI Secret Variable\nmasked + protected]
    F -->|Enterprise| I[HashiCorp Vault\nor AWS Secrets Manager]

    G --> J[ansible-playbook runtime]
    H --> J
    I --> J

    J --> K[Decrypt vault.yml\nin memory only]
    K --> L[Variables available\nas plaintext in memory]
    L --> M[Tasks execute\nwith decrypted values]
    M --> N[no_log: true\nprevents logging secrets]
```

---

## 🔄 Flow / Lifecycle

```mermaid
sequenceDiagram
    participant DEV as Developer
    participant CLI as ansible-playbook
    participant VF as Vault Password Source
    participant ENC as Encrypted Files
    participant HOST as Managed Node

    DEV->>CLI: ansible-playbook site.yml --vault-password-file ~/.vault_pass
    CLI->>VF: Read vault password
    VF-->>CLI: Password returned
    CLI->>ENC: Load group_vars/production/vault.yml
    CLI->>CLI: Decrypt using AES-256 + password
    CLI->>CLI: Merge vault vars into variable scope
    CLI->>CLI: Resolve {{ db_password }} → decrypted value
    CLI->>HOST: Execute task with decrypted value (in memory)
    HOST-->>CLI: Result
    CLI->>CLI: no_log: true → suppress secret from output
    CLI->>DEV: Play recap (no secrets in output)
```

---

## 💻 Code Examples

### ✅ Example 1: Full vault setup for a multi-environment project

```bash
# 1. Create vault password files (one per environment)
echo "production-vault-password-here" > ~/.vault_pass_prod
echo "staging-vault-password-here" > ~/.vault_pass_staging
chmod 600 ~/.vault_pass_prod ~/.vault_pass_staging

# 2. Add to .gitignore
echo ".vault_pass*" >> .gitignore

# 3. Create and encrypt vault files
ansible-vault create \
  --vault-id production@~/.vault_pass_prod \
  group_vars/production/vault.yml

ansible-vault create \
  --vault-id staging@~/.vault_pass_staging \
  group_vars/staging/vault.yml

# 4. Run with both vault IDs
ansible-playbook site.yml \
  -i inventory/production \
  --vault-id production@~/.vault_pass_prod \
  --vault-id staging@~/.vault_pass_staging
```

### ✅ Example 2: Conventional vars.yml + vault.yml split

```yaml
# group_vars/all/vars.yml — PLAINTEXT, committed to Git
db_host: "{{ vault_db_host }}"
db_port: 5432
db_name: myapp
db_user: "{{ vault_db_user }}"
db_password: "{{ vault_db_password }}"
smtp_host: smtp.example.com
smtp_user: "{{ vault_smtp_user }}"
smtp_password: "{{ vault_smtp_password }}"
aws_access_key: "{{ vault_aws_access_key }}"
aws_secret_key: "{{ vault_aws_secret_key }}"
```

```yaml
# group_vars/all/vault.yml — ENCRYPTED, committed to Git
# (shown here pre-encryption for clarity)
vault_db_host: prod-db.internal
vault_db_user: myapp_prod
vault_db_password: "s3cur3P@ssw0rd!"
vault_smtp_user: alerts@example.com
vault_smtp_password: "smtp-pass-here"
vault_aws_access_key: "AKIAIOSFODNN7EXAMPLE"
vault_aws_secret_key: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

### ✅ Example 3: Vault in a role's defaults/vault pattern

```yaml
# roles/myapp/defaults/main.yml
# Reference vault variables — actual secrets in group_vars/vault.yml
myapp_db_password: "{{ vault_myapp_db_password }}"
myapp_api_key: "{{ vault_myapp_api_key }}"
```

```yaml
# roles/myapp/tasks/configure.yml
- name: Deploy application environment file
  ansible.builtin.template:
    src: env.j2
    dest: /etc/myapp/.env
    owner: myapp
    mode: '0600'    # restrict read to app user only
  no_log: true     # don't log the rendered template content
  notify: Restart myapp
```

### ✅ Example 4: Rotating vault passwords

```bash
# Rekey a single file with a new password
ansible-vault rekey \
  --vault-id old@~/.vault_pass_old \
  --new-vault-id new@~/.vault_pass_new \
  group_vars/production/vault.yml

# Rekey all vault files in the project
find . -name "vault.yml" -exec \
  ansible-vault rekey --ask-vault-pass {} \;
```

### ❌ Anti-pattern — Plaintext secrets in group_vars

```yaml
# ❌ NEVER DO THIS — secret committed to Git in plaintext
# group_vars/production/vars.yml
db_password: "supersecretpassword123"
aws_secret_key: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
api_token: "ghp_abc123def456"

# ✅ Encrypt secrets with Vault
# group_vars/production/vars.yml
db_password: "{{ vault_db_password }}"    # plaintext reference

# group_vars/production/vault.yml (ENCRYPTED)
vault_db_password: "supersecretpassword123"
```

---

## ⚙️ Configuration & Options

### `ansible.cfg` vault settings

```ini
[defaults]
vault_password_file = ~/.vault_pass         # default vault password file
# vault_identity_list = prod@~/.vault_prod,staging@~/.vault_staging
```

### `ansible-vault` command reference

| Command | What it does |
|---------|-------------|
| `encrypt <file>` | Encrypt a plaintext file in-place |
| `decrypt <file>` | Decrypt to plaintext in-place (careful!) |
| `view <file>` | View decrypted content without writing to disk |
| `edit <file>` | Edit in `$EDITOR`, re-encrypt on save |
| `rekey <file>` | Change the vault password for a file |
| `create <file>` | Create a new encrypted file |
| `encrypt_string <value>` | Encrypt a string for embedding in YAML |

### Vault ID syntax

```
--vault-id label@source
```

| Source format | Example | What it does |
|--------------|---------|-------------|
| `filename` | `prod@~/.vault_pass` | Read password from file |
| `prompt` | `prod@prompt` | Prompt interactively |
| `script` | `prod@./get_pass.py` | Run script, use stdout as password |

---

## 🧩 Patterns & Best Practices

**What experienced engineers do:**
- Always use the `vault_` prefix convention for encrypted variables — makes `vars.yml` self-documenting
- Store vault passwords in a secret manager (AWS Secrets Manager, HashiCorp Vault) and retrieve them via script at runtime — no vault password file on CI runners
- Use separate vault IDs per environment — production and staging secrets encrypted with different keys
- Add `no_log: true` to every task that uses vault-sourced variables — prevents secrets appearing in AWX/Tower job output
- Use `ansible-vault view` (not `decrypt`) when you need to read an encrypted file — view decrypts in memory, decrypt writes to disk

**What beginners typically get wrong:**
- Committing `.vault_pass` files to Git — the entire point of vault is negated
- Encrypting the wrong file (vars.yml instead of vault.yml) — the vars file needs to be readable for documentation purposes
- Not using `no_log: true` — secrets leak into CI logs or AWX output
- Using the same vault password for all environments — a compromised staging password shouldn't expose production secrets
- Forgetting `chmod 600` on vault password files — other users on the system can read them

**Senior-level nuance:**
- For large teams, consider skipping Ansible Vault entirely for secrets that change frequently and using HashiCorp Vault or AWS Secrets Manager with the `community.hashi_vault` or `amazon.aws` lookup plugins instead. Vault is excellent for static secrets; a dedicated secret manager is better for dynamic, frequently-rotated credentials.
- Git history permanently contains all commits — if a secret was ever committed in plaintext, it must be considered compromised even after removal. Use `git filter-repo` to purge the history and rotate the secret immediately.

---

## 🔗 How It Connects

- **Builds on:** `12-roles.md` — role `defaults/` and `group_vars/` are where vault variables live
- **Leads to:** `14-error-handling.md` — once secrets are handled safely, we focus on robust failure management
- **Related concepts:** Topic 6 (variable precedence — vault vars follow the same rules), Topic 25 (CI/CD integration — vault in pipelines), Topic 27 (security hardening — vault is one layer of a defence-in-depth strategy)

---

## 🎯 Interview Questions (Conceptual)

**Q1: What encryption algorithm does Ansible Vault use?**
> **A:** AES-256 in CTR mode with a PBKDF2-derived key. The vault password you provide is not used directly — it's passed through PBKDF2 with a random salt to derive the actual AES key. This means brute-forcing the vault password requires per-attempt key derivation, making it computationally expensive.

**Q2: What is the difference between `ansible-vault encrypt` and `ansible-vault encrypt_string`?**
> **A:** `encrypt` encrypts an entire file — the whole file becomes ciphertext. `encrypt_string` encrypts a single value and outputs a `!vault` YAML block you can embed inline in any YAML file. Use whole-file encryption for dedicated vault files; use `encrypt_string` sparingly for single values in otherwise-plaintext files.

**Q3: What is a Vault ID and why would you use multiple?**
> **A:** A Vault ID is a label attached to encrypted content that identifies which password was used. Multiple Vault IDs let you encrypt different files with different passwords — for example, production secrets with one password, staging with another. At runtime you provide all passwords and Ansible tries each until one decrypts a given file. This means a compromised staging password doesn't expose production secrets.

**Q4: How do you prevent vault-decrypted values from appearing in Ansible output?**
> **A:** Add `no_log: true` to any task that uses vault-sourced variables. This suppresses the task's input and output from all logging — including AWX/Tower job output, callback plugins, and verbose mode. Apply it to tasks creating users with passwords, writing secrets to files, or calling APIs with tokens.

**Q5: What happens if you commit a `.vault_pass` file to Git by mistake?**
> **A:** Every secret protected by that vault password must be considered compromised. Immediately rotate all secrets, rekey all vault files with a new password, and purge the file from Git history using `git filter-repo`. Notify your security team. The leaked password may now be in dozens of developer clones, CI caches, and GitHub's own servers.

---

## 🧠 Scenario-Based Interview Problems

**Scenario 1: "You're setting up a CI/CD pipeline that needs to run Ansible playbooks against production. The playbooks use Vault-encrypted secrets. How do you provide the vault password to CI without storing it in the repo?"**
> **Problem:** Vault password must reach CI without being committed or exposed.
> **Approach:** Store the vault password as a protected, masked CI secret variable (GitHub Actions secret / GitLab CI variable). In the pipeline job, write it to a temp file (`echo "$VAULT_PASS" > ~/.vault_pass && chmod 600 ~/.vault_pass`), run the playbook with `--vault-password-file ~/.vault_pass`, and delete it in the `always`/`after_script` step. For higher security, store the vault password in AWS Secrets Manager or HashiCorp Vault and use a vault-password script that retrieves it at runtime — the CI runner needs only IAM/token access, not the password itself.
> **Trade-offs:** Direct CI secret variables are simple but the password is accessible to any job in that CI project. Secrets Manager adds a network dependency but enables audit logging and automatic rotation.

**Scenario 2: "An engineer accidentally committed a database password in plaintext to the main branch of your Ansible repo two weeks ago. How do you respond?"**
> **Problem:** Secret exposure via Git history — the commit is in history even after it was "fixed".
> **Approach:** (1) Assume compromised — immediately rotate the database password. (2) Purge the file from all Git history using `git filter-repo --path path/to/file --invert-paths` and force-push. (3) Notify all engineers to re-clone (their local histories still contain the secret). (4) Check if the repo was public — if so, assume any secret in history is permanently compromised regardless of purging. (5) Audit access logs for the database from the day of the commit. (6) Encrypt the replacement password with Ansible Vault and add a pre-commit hook (`detect-secrets` or `git-secrets`) to prevent recurrence.
> **Trade-offs:** History rewriting disrupts all contributors and breaks any open PRs. Schedule it during low-activity hours and communicate clearly. Prevention (pre-commit hooks, mandatory vault for all secrets) is far cheaper than response.

---

## ⚡ Quick Notes — Revision Card

- 📌 Vault uses **AES-256** encryption — secure at rest, decrypted in memory at runtime only
- 📌 Whole file: `ansible-vault encrypt file.yml` | Single value: `ansible-vault encrypt_string`
- 📌 Vault ID: `--vault-id label@source` — label encrypted content, use multiple passwords per env
- 📌 `vault_` prefix convention — `vault_db_password` in `vault.yml`, `db_password: "{{ vault_db_password }}"` in `vars.yml`
- 📌 `--vault-password-file ~/.vault_pass` | `--ask-vault-pass` (interactive) | `ANSIBLE_VAULT_PASSWORD_FILE` (env)
- 📌 `ansible-vault view` = safe (in memory) | `ansible-vault decrypt` = writes plaintext to disk
- ⚠️ NEVER commit `.vault_pass` files — add to `.gitignore`
- ⚠️ NEVER encrypt `vars.yml` — encrypt only `vault.yml` (vars.yml must be readable for docs)
- ⚠️ Always `no_log: true` on tasks that use vault-sourced variables
- ⚠️ Git history is permanent — a leaked secret is compromised forever, even after removal
- 💡 Separate vault IDs per environment — compromised staging ≠ compromised production
- 🔑 For frequently-rotated secrets, use HashiCorp Vault or AWS Secrets Manager + lookup plugins instead

---

## 🔖 References & Further Reading

- 📄 [Ansible Vault — Official Docs](https://docs.ansible.com/ansible/latest/vault_guide/index.html)
- 📄 [ansible-vault CLI reference](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html)
- 📄 [Vault IDs and Multiple Passwords](https://docs.ansible.com/ansible/latest/vault_guide/vault_managing_passwords.html)
- 📝 [Best Practices for Vault](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html#keep-vaulted-variables-safely-visible)
- 🎥 [Jeff Geerling — Ansible Vault](https://www.youtube.com/watch?v=TIirzef1YRE)
- 📚 *Ansible for DevOps* — Jeff Geerling (Chapter 10)
- ➡️ Related in this course: [`12-roles.md`] · [`14-error-handling.md`]

---
