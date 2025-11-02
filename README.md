# infra-labs
Learning and lab work in Linux administration, automation, and observability. Building skills step by step in Bash, Python, Ansible/AWX, Grafana/Prometheus, and beyond.

## Labs Index

- **Lesson 2 – Managed Environment**
  - Folder: [`ansible/lesson2-managed-env/`](ansible/lesson2-managed-env/)
  - Lesson README: [`ansible/lesson2-managed-env/README.md`](ansible/lesson2-managed-env/README.md)
  - Deep Notes: [`ansible/lesson2-managed-env/docs/lesson2-notes.md`](ansible/lesson2-managed-env/docs/lesson2-notes.md)

### Quick validate
```bash
ansible-inventory --graph
ansible all -m ping -o
ansible all -b -m command -a 'id' -o


- **Lesson 3 – Ad-hoc**
  - Folder: [ansible/lesson3-ad-hoc/](ansible/lesson3-ad-hoc/)
  - Lesson README: [ansible/lesson3-ad-hoc/README.md](ansible/lesson3-ad-hoc/README.md)
  - Deep Notes: [ansible/lesson3-ad-hoc/docs/lesson3-notes.md](ansible/lesson3-ad-hoc/docs/lesson3-notes.md)

- **Lesson 4 – Playbooks**
  - Folder: [ansible/lesson4-playbooks/](ansible/lesson4-playbooks/)
  - Lesson README: [ansible/lesson4-playbooks/docs/README.md](ansible/lesson4-playbooks/docs/README.md)
  - Deep Notes: [ansible/lesson4-playbooks/docs/lesson4_notes.md](ansible/lesson4-playbooks/docs/lesson4_notes.md)

- **Lesson 5 – Variables & Vault**
  - Folder: [ansible/lesson5-variables/](ansible/lesson5-variables/)
  - Deep Notes: [ansible/lesson5-variables/docs/lesson5_notes.md](ansible/lesson5-variables/docs/lesson5_notes.md)
