# 📚 Ansible — From Zero to Architect

> A self-paced, notes-based course covering Ansible from fundamentals to architect level.
> Built with the AI Markdown Course Builder format. GitHub-ready, interview-focused.

---

## 🗺️ How to Use This Repo

1. Start with `PROGRESS-TRACKER.md` to see the full syllabus and your current position
2. Work through phases in order — each `.md` file is a complete, self-contained topic
3. Each file includes: concept notes, Mermaid diagrams, code examples, interview Q&A, and a revision card
4. To resume with AI: copy the **AI Resume Prompt** from `PROGRESS-TRACKER.md` and paste it into any Claude session

---

## 📋 Phases

| Phase | Level | Topics | Goal |
|-------|-------|--------|------|
| 🟢 Phase 1 | Fundamentals | 5 topics | Understand Ansible's model, write and run your first playbooks |
| 🔵 Phase 2 | Intermediate | 7 topics | Build structured, reusable playbooks with vars, loops, templates, and roles |
| 🟠 Phase 3 | Advanced | 7 topics | Secrets, error handling, testing, custom modules, Galaxy |
| 🔴 Phase 4 | Senior / Production | 6 topics | Scale, AWX/AAP, network automation, Windows, CI/CD |
| 🟣 Phase 5 | Architect / Expert | 3 topics | Collections, security hardening, org-wide design |

---

## 📁 Repo Structure

```
ansible-zero-to-architect/
├── README.md
├── PROGRESS-TRACKER.md               ← Live tracker + AI resume prompt ← KEY FILE
├── phase-1-fundamentals/
│   ├── 01-what-is-ansible.md
│   ├── 02-installation-and-setup.md
│   ├── 03-inventory.md
│   ├── 04-ad-hoc-commands.md
│   └── 05-playbook-basics.md
├── phase-2-intermediate/
│   ├── 06-variables.md
│   ├── 07-facts-and-magic-variables.md
│   ├── 08-conditionals-and-loops.md
│   ├── 09-handlers.md
│   ├── 10-templates-jinja2.md
│   ├── 11-files-and-modules.md
│   └── 12-roles.md
├── phase-3-advanced/
│   ├── 13-ansible-vault.md
│   ├── 14-error-handling.md
│   ├── 15-tags-and-limits.md
│   ├── 16-includes-and-imports.md
│   ├── 17-custom-modules-and-plugins.md
│   ├── 18-testing-with-molecule.md
│   └── 19-ansible-galaxy.md
├── phase-4-senior-production/
│   ├── 20-performance-and-scaling.md
│   ├── 21-awx-and-aap.md
│   ├── 22-dynamic-inventory-advanced.md
│   ├── 23-network-automation.md
│   ├── 24-windows-automation.md
│   └── 25-cicd-integration.md
├── phase-5-architect/
│   ├── 26-collections-development.md
│   ├── 27-security-hardening.md
│   └── 28-architecture-and-org-design.md
└── resources/
    ├── cheatsheets/
    │   └── ansible-cheatsheet.md
    └── project-ideas.md
```

---

## 🤝 Who This Is For

- **DevOps / SRE engineers** who want to go beyond basic playbooks
- **Sysadmins** transitioning to infrastructure-as-code
- **Platform engineers** building internal automation platforms
- **Anyone preparing** for Ansible interviews, the Red Hat EX294 exam, or the RHCE certification

---

## 🛠️ Prerequisites

- Basic Linux command-line comfort (ssh, file permissions, package managers)
- A text editor and a terminal
- At least one Linux VM or EC2 instance to practice against (or use Docker via Molecule)

---

## 📌 Quick Links

- 🔖 [Official Ansible Docs](https://docs.ansible.com/)
- 🌐 [Ansible Galaxy](https://galaxy.ansible.com/)
- 🧪 [Molecule Docs](https://molecule.readthedocs.io/)
- 🏢 [Red Hat Automation Hub](https://console.redhat.com/ansible/automation-hub)
