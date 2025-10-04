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

