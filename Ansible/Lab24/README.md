# Lab 24 - Initial Ansible Configuration and Ad-Hoc Execution

## Objective

The goal of this lab is to set up Ansible on a control node, establish SSH communication with a managed node, create an inventory file, and use an Ansible ad-hoc command to install Docker remotely.

## Lab Environment

I used two Ubuntu 22.04 virtual machines running on VirtualBox.

| Machine         | IP Address    | Purpose                   |
| --------------- | ------------- | ------------------------- |
| ansible-control | 192.168.56.50 | Runs Ansible              |
| ansible-managed | 192.168.56.51 | Receives Ansible commands |

Both VMs use NAT for internet access and a Host-only network for communication.

## Steps

### 1. Install Ansible

On the control node, I updated the package list and installed Ansible.

```bash
sudo apt update
sudo apt install -y ansible
```

Verified the installation:

```bash
ansible --version
```

Installed version: **Ansible 2.10.8**

### 2. Configure SSH Authentication

Generated an SSH key pair on the control node:

```bash
ssh-keygen -t ed25519 -C "ansible-control"
```

Installed and verified the SSH server on the managed node:

```bash
sudo apt install -y openssh-server
sudo systemctl status ssh --no-pager
```

Copied the public key from the control node to the managed node:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub ansible@192.168.56.51
```

Tested the SSH connection:

```bash
ssh ansible@192.168.56.51
```

The connection was successful without requiring the managed node's login password.

### 3. Create Ansible Inventory

Created an inventory file named `inventory.ini`:

```ini
[managed]
ansible-managed ansible_host=192.168.56.51 ansible_user=ansible
```

The inventory defines the managed node's IP address and SSH username.

### 4. Test Ansible Connectivity

Ran the Ansible ping module:

```bash
ansible managed -i inventory.ini -m ping
```

Result:

```text
ansible-managed | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

This confirmed that Ansible could communicate with the managed node successfully over SSH.

### 5. Install Docker Using an Ad-Hoc Command

Installed Docker remotely using the Ansible `apt` module:

```bash
ansible managed -i inventory.ini -b -K -m apt -a "name=docker.io state=present update_cache=yes"
```

Command explanation:

- `managed`: Target group from the inventory.
- `-i`: Specifies the inventory file.
- `-b`: Executes the task with sudo privileges.
- `-K`: Prompts for the sudo password.
- `-m apt`: Uses the Ansible apt module.
- `state=present`: Ensures Docker is installed.

The installation completed successfully with a `CHANGED` status.

### 6. Verify Docker Installation

Checked the Docker version remotely:

```bash
ansible managed -i inventory.ini -b -K -a "docker --version"
```

Result:

```text
Docker version 29.1.3
```

Checked the Docker service:

```bash
ansible managed -i inventory.ini -b -K -a "systemctl is-active docker"
```

Result:

```text
active
```

Docker was successfully installed and running on the managed node.

## Result

Successfully configured Ansible to manage a remote Ubuntu server using SSH key authentication.

I was able to execute commands remotely and install Docker without manually performing the installation on the managed node.

This lab helped me understand Ansible inventory management, SSH authentication, ad-hoc commands, privilege escalation, and remote configuration management.
