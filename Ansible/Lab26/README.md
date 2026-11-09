# Lab 26 - Structured Configuration Management with Ansible Roles

## Objective

The goal of this lab is to organize Ansible automation into reusable roles for installing and configuring Docker, kubectl, and Jenkins.

This lab builds on Lab 25, where all installation tasks were written in a single playbook.

## Lab Environment

- Control Node: Ubuntu 22.04 - 192.168.56.50
- Managed Node: Ubuntu 22.04 - 192.168.56.51
- Automation Tool: Ansible
- Virtualization: VirtualBox

## Project Structure

```text
Lab26/
├── inventory.ini
├── playbook.yml
└── roles/
    ├── docker/
    │   └── tasks/
    │       └── main.yml
    ├── kubectl/
    │   └── tasks/
    │       └── main.yml
    └── jenkins/
        └── tasks/
            └── main.yml
```

Each role contains its own tasks, making the automation easier to organize, maintain, and reuse.

## Implementation

### 1. Create Ansible Roles

Generated the role structure using Ansible Galaxy:

```bash
ansible-galaxy init docker
ansible-galaxy init kubectl
ansible-galaxy init jenkins
```

### 2. Docker Role

Created `roles/docker/tasks/main.yml` to:

- Install Docker using the apt module.
- Ensure Docker is running.
- Enable Docker to start automatically after reboot.

### 3. kubectl Role

Created `roles/kubectl/tasks/main.yml` to:

- Download kubectl v1.35.10 using the get_url module.
- Verify the binary using a SHA-256 checksum.
- Install kubectl in `/usr/local/bin` with executable permissions.

### 4. Jenkins Role

Created `roles/jenkins/tasks/main.yml` to:

- Install Java 21 and fontconfig.
- Configure the Jenkins repository signing key.
- Add the Jenkins LTS repository.
- Install Jenkins.
- Start and enable the Jenkins service.

### 5. Main Playbook

Created `playbook.yml` to execute all three roles.

```yaml
---
- name: Configure DevOps tools using Ansible Roles
  hosts: managed
  become: true

  roles:
    - docker
    - kubectl
    - jenkins
```

The main playbook is much shorter than the one used in Lab 25 because each role manages its own tasks.

## Execution

Checked the playbook syntax:

```bash
ansible-playbook -i inventory.ini playbook.yml --syntax-check
```

Executed the playbook:

```bash
ansible-playbook -i inventory.ini playbook.yml -K
```

All three roles executed successfully on the managed node.

## Results

Final Ansible Play Recap:

```text
ansible-managed : ok=10 changed=0 unreachable=0 failed=0
```

All tasks completed successfully without errors.

Since the required tools were already installed during Lab 25, Ansible reported `changed=0`.

This demonstrates idempotency: running the same automation again does not make unnecessary changes when the system is already in the desired state.

## Result

Successfully organized the Docker, kubectl, and Jenkins installation tasks into three reusable Ansible roles.

This lab helped me understand role-based configuration management, modular automation, code reusability, and idempotency.

Compared to a single large playbook, using roles makes the automation easier to maintain and extend.
