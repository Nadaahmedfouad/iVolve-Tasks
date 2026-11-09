# Lab 25 - Automated Configuration Using Ansible Playbooks

## Objective

The goal of this lab is to use an Ansible playbook to automate the installation and configuration of Docker, kubectl, and Jenkins on a remote Ubuntu server.

Instead of executing individual commands manually, all installation steps are defined in a single YAML playbook.

## Lab Environment

I used the same two Ubuntu 22.04 virtual machines configured in Lab 24.

| Machine         | IP Address    | Role                 |
| --------------- | ------------- | -------------------- |
| ansible-control | 192.168.56.50 | Ansible Control Node |
| ansible-managed | 192.168.56.51 | Managed Node         |

SSH key authentication was already configured between the two machines.

## Tools Used

- Ansible
- Ubuntu 22.04
- Docker
- kubectl
- OpenJDK 21
- Jenkins
- VirtualBox

## Implementation

### 1. Prepare the Inventory

I reused the inventory file from Lab 24.

**inventory.ini**

```ini
[managed]
ansible-managed ansible_host=192.168.56.51 ansible_user=ansible
```

### 2. Create the Ansible Playbook

Created `playbook.yml` to automate the installation of all required tools.

The playbook uses:

```yaml
---
- name: Configure DevOps tools
  hosts: managed
  become: true

  tasks:
```

- `hosts: managed` targets the managed node.
- `become: true` allows tasks to run with sudo privileges.
- `tasks` contains the automation steps.

### 3. Install Docker

Used the Ansible `apt` module to install Docker and the `service` module to ensure it is running.

```yaml
- name: Install Docker
  ansible.builtin.apt:
    name: docker.io
    state: present

- name: Ensure Docker service is running
  ansible.builtin.service:
    name: docker
    state: started
    enabled: true
```

Since Docker was already installed in Lab 24, Ansible recognized its existing state without reinstalling it.

### 4. Install kubectl

Used the `get_url` module to download kubectl v1.35.10 from the official Kubernetes distribution.

```yaml
- name: Install kubectl
  ansible.builtin.get_url:
    url: https://dl.k8s.io/release/v1.35.10/bin/linux/amd64/kubectl
    dest: /usr/local/bin/kubectl
    mode: "0755"
    checksum: "sha256:c49a28f1f4efc4d475ed05f47f2ce8e563bf7bef69ed77fe1b924ea6d83b6327"
```

The checksum verifies the downloaded file, while the permissions make kubectl executable.

### 5. Install Java

Jenkins requires Java, so OpenJDK 21 and fontconfig were installed first.

```yaml
- name: Install Java for Jenkins
  ansible.builtin.apt:
    name:
      - fontconfig
      - openjdk-21-jre
    state: present
    update_cache: yes
```

### 6. Install Jenkins

The Jenkins installation was automated through several tasks:

1. Create the APT keyrings directory.
2. Download the Jenkins repository signing key.
3. Add the Jenkins LTS repository.
4. Install the Jenkins package.
5. Start and enable the Jenkins service.

The final service configuration:

```yaml
- name: Ensure Jenkins service is running
  ansible.builtin.service:
    name: jenkins
    state: started
    enabled: true
```

This ensures Jenkins is running and starts automatically after reboot.

## Execution

First, checked the playbook syntax:

```bash
ansible-playbook -i inventory.ini playbook.yml --syntax-check
```

Then executed the playbook:

```bash
ansible-playbook -i inventory.ini playbook.yml -K
```

Ansible connected to the managed node using SSH and executed the configuration tasks.

## Verification

Verified the installed tools remotely using Ansible:

**Docker**

```bash
ansible managed -i inventory.ini -m command -a "docker --version"
```

**kubectl**

```bash
ansible managed -i inventory.ini -m command -a "kubectl version --client"
```

**Java**

```bash
ansible managed -i inventory.ini -m command -a "java -version"
```

**Jenkins Service**

```bash
ansible managed -i inventory.ini -m command -a "systemctl is-active jenkins"
```

Jenkins returned:

```text
active
```

I also accessed the Jenkins web interface from my Windows browser:

```text
http://192.168.56.51:8080
```

Completed the initial Jenkins setup, installed the suggested plugins, and created an administrator account.

## Idempotency

One important concept demonstrated in this lab is idempotency.

When Docker was already installed, Ansible returned `ok` instead of reinstalling it.

This means Ansible checks the current configuration and only makes changes when needed.

The playbook can be executed repeatedly to maintain the desired configuration.

## Result

Successfully automated the installation and configuration of Docker, kubectl, Java, and Jenkins on a remote Ubuntu server using a single Ansible playbook.

This lab helped me understand YAML playbooks, Ansible modules, task execution, privilege escalation, service management, and idempotency.
