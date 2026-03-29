# 📊 Progress Tracker — Ansible: Zero to Architect

> **Course:** Ansible — From Zero to Architect
> **Started:** 2026-03-29
> **Current Position:** Phase 1 · Topic 1 of 28 · `01-what-is-ansible.md`
> **Last Completed:** Not started yet
> **Next Up:** Topic 1 — What is Ansible? (`01-what-is-ansible.md`)

---

## 🤖 AI Resume Prompt

> Copy this block and paste it to any AI to continue exactly where you left off.

```
I am learning Ansible using the AI Markdown Course Builder format.
Here is my current progress:

- Course: Ansible — From Zero to Architect
- Total Topics: 28 across 5 Phases
- Last completed topic: None — Not started yet
- Current phase: Phase 1 — Fundamentals
- Next topic to generate: Topic 1 — What is Ansible? (`01-what-is-ansible.md`)

Please continue the course from Topic 1. Generate the full .md notes file for it
following the standard AI Markdown Course Builder format:
Concept Overview, In-Depth Explanation, Architecture/Diagrams, Flow/Lifecycle,
Code Examples, Patterns & Best Practices, Interview Questions, Scenario-Based Problems,
Quick Notes, and References.

After generating, update my tracker so I can paste it again next time.
```

---

## 📋 Full Syllabus & Progress

### 🟢 Phase 1 — Fundamentals

| #   | Topic                | File                           | Subtopics                                                            | Status         |
| --- | -------------------- | ------------------------------ | -------------------------------------------------------------------- | -------------- |
| 1   | What is Ansible?     | `01-what-is-ansible.md`        | Agentless model, Control/managed nodes, Idempotency                  | ✅ Done        |
| 2   | Installation & Setup | `02-installation-and-setup.md` | pip/OS install, ansible.cfg, ping test                               | ✅ Done        |
| 3   | Inventory            | `03-inventory.md`              | INI/YAML formats, Groups, host_vars/group_vars, Dynamic inventory    | ✅ Done        |
| 4   | Ad-Hoc Commands      | `04-ad-hoc-commands.md`        | Module syntax, Common modules, Ad-hoc vs playbooks                   | ⬜ Not Started |
| 5   | Playbook Basics      | `05-playbook-basics.md`        | YAML syntax, Plays/tasks/modules, Running playbooks, Execution order | ⬜ Not Started |

### 🔵 Phase 2 — Intermediate

| #   | Topic                   | File                              | Subtopics                                                            | Status         |
| --- | ----------------------- | --------------------------------- | -------------------------------------------------------------------- | -------------- |
| 6   | Variables               | `06-variables.md`                 | Variable types, 22-level precedence, Special vars                    | ⬜ Not Started |
| 7   | Facts & Magic Variables | `07-facts-and-magic-variables.md` | setup module, Custom facts, Disabling facts                          | ⬜ Not Started |
| 8   | Conditionals & Loops    | `08-conditionals-and-loops.md`    | when clause, loop/with_items, loop_control                           | ⬜ Not Started |
| 9   | Handlers                | `09-handlers.md`                  | Defining handlers, Execution order, flush_handlers                   | ⬜ Not Started |
| 10  | Templates & Jinja2      | `10-templates-jinja2.md`          | Filters/tests/lookups, template vs copy, Complex configs             | ⬜ Not Started |
| 11  | Files & Modules         | `11-files-and-modules.md`         | File modules, Package management, Service management                 | ⬜ Not Started |
| 12  | Roles                   | `12-roles.md`                     | Role structure, ansible-galaxy init, Dependencies, include vs import | ⬜ Not Started |

### 🟠 Phase 3 — Advanced

| #   | Topic                    | File                               | Subtopics                                                                   | Status         |
| --- | ------------------------ | ---------------------------------- | --------------------------------------------------------------------------- | -------------- |
| 13  | Ansible Vault            | `13-ansible-vault.md`              | Encrypting files/strings, Vault IDs, CI/CD integration                      | ⬜ Not Started |
| 14  | Error Handling           | `14-error-handling.md`             | ignore_errors/failed_when, block/rescue/always, any_errors_fatal            | ⬜ Not Started |
| 15  | Tags & Limits            | `15-tags-and-limits.md`            | Assigning tags, --tags/--limit flags, Tag strategy                          | ⬜ Not Started |
| 16  | Includes & Imports       | `16-includes-and-imports.md`       | import_tasks vs include_tasks, import_playbook, Passing vars                | ⬜ Not Started |
| 17  | Custom Modules & Plugins | `17-custom-modules-and-plugins.md` | Writing Python modules, Action/callback/filter plugins, AnsibleModule class | ⬜ Not Started |
| 18  | Testing with Molecule    | `18-testing-with-molecule.md`      | Molecule architecture, Testinfra verifiers, CI/CD integration               | ⬜ Not Started |
| 19  | Ansible Galaxy           | `19-ansible-galaxy.md`             | Installing roles/collections, requirements.yml, Publishing                  | ⬜ Not Started |

### 🔴 Phase 4 — Senior / Production

| #   | Topic                      | File                               | Subtopics                                                          | Status         |
| --- | -------------------------- | ---------------------------------- | ------------------------------------------------------------------ | -------------- |
| 20  | Performance & Scaling      | `20-performance-and-scaling.md`    | forks/serial/async, SSH pipelining, Strategies, Profiling          | ⬜ Not Started |
| 21  | AWX & AAP                  | `21-awx-and-aap.md`                | AWX architecture, Job templates/credentials, Workflows, AAP vs AWX | ⬜ Not Started |
| 22  | Dynamic Inventory Advanced | `22-dynamic-inventory-advanced.md` | Cloud inventory plugins, Constructed plugin, Caching               | ⬜ Not Started |
| 23  | Network Automation         | `23-network-automation.md`         | Connection types, Network modules, NAPALM/cli_command              | ⬜ Not Started |
| 24  | Windows Automation         | `24-windows-automation.md`         | WinRM/Kerberos, Windows modules, PowerShell integration            | ⬜ Not Started |
| 25  | CI/CD Integration          | `25-cicd-integration.md`           | GitHub Actions/GitLab CI, Secrets in CI, GitOps patterns           | ⬜ Not Started |

### 🟣 Phase 5 — Architect / Expert

| #   | Topic                     | File                                | Subtopics                                                                                   | Status         |
| --- | ------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------- | -------------- |
| 26  | Collections Development   | `26-collections-development.md`     | Collection structure, galaxy.yml, Private Hub, Versioning                                   | ⬜ Not Started |
| 27  | Security Hardening        | `27-security-hardening.md`          | Least privilege/become, No-log/audit trail, CIS benchmarks, SIEM integration                | ⬜ Not Started |
| 28  | Architecture & Org Design | `28-architecture-and-org-design.md` | Repo design, RBAC/team structure, Multi-cloud patterns, When NOT to use Ansible, Governance | ⬜ Not Started |

---

## 📈 Progress Summary

- ✅ Completed: 0 / 28
- 🔄 In Progress: 0
- ⬜ Not Started: 28
- 📊 Overall: 0%

---

## 🗂️ Completed Topics Log

| #   | Topic | Completed On | File |
| --- | ----- | ------------ | ---- |
| —   | —     | —            | —    |
