<h1 align="center">Lesson 10: Troubleshooting Ansible</h1>

## Troubleshooting Without Logs
- Relevant Ansible processing information is written to STDOUT when running ansible-playbook.
- With ansible-navigator, use: ansible-navigator run myplay.yml -m stdout
- Always start troubleshooting by reading the PLAY RECAP.
- If something is wrong, scroll upward in the output to find the failure.
- Increase verbosity using:
  ansible-playbook -v myplay.yml
  ansible-playbook -vv myplay.yml
  ansible-playbook -vvv myplay.yml
  ansible-playbook -vvvv myplay.yml

### Example Output
TASK [storage : Update facts]
ok: [ansible2]

PLAY RECAP
ansible1: ok=1 changed=0 unreachable=0 failed=1 skipped=8 rescued=1 ignored=0
ansible2: ok=1 changed=0 unreachable=0 failed=0 skipped=12 rescued=0 ignored=0

## Understanding Logging
- Ansible does not log to files by default — everything goes to STDOUT.
- To log output to a file, set log_path in ansible.cfg:
[defaults]
log_path = ./logs/ansible.log
- Avoid logging to /var/log unless using sudo.
- ansible-navigator creates artifact directories containing detailed playbook execution information.
- Best practice: view these artifacts using ansible-navigator in interactive mode.

## Managing ansible-navigator Artifacts
- Artifact files are created for every run.
- They contain sensitive information and will fill disk over time.
- Disable artifact creation with:
ansible-navigator:
  playbook-artifact:
    enable: false

## Using the debug Module
- The debug module prints variables and helps analyze play behavior.
Example:
- name: Print variable
  debug:
    var: myvar

- Use verbosity control:
- name: Print debug only with -vv
  debug:
    msg: "Extra debug info"
    verbosity: 2

- Run with:
ansible-playbook play.yml -vv
