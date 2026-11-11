
# Secure AWS EC2 Automation with Ansible

Combined Ansible Labs 27 & 28

## Overview

This project uses Ansible to automatically discover an AWS EC2 instance and configure a MySQL database securely.

Instead of manually connecting to the server and installing everything, Ansible discovers the EC2 instance through AWS Dynamic Inventory and runs a playbook to complete the setup.

## Tools Used

- Ansible
- AWS EC2
- AWS IAM
- AWS Dynamic Inventory
- Ansible Vault
- MySQL
- Python (Boto3)
- SSH

## How It Works

1. An Ubuntu EC2 instance is launched with the tag `server=ivolve`.
2. Ansible uses AWS Dynamic Inventory to discover the running instance.
3. Ansible connects to EC2 through SSH.
4. Ansible installs MySQL and its Python dependencies.
5. The playbook creates a database called `ivolve`.
6. A MySQL user named `ivolveuser` is created with privileges on the database.
7. The user's password is loaded from an encrypted Ansible Vault file.
8. MySQL connectivity and database permissions are verified.

## Project Structure

```text
lab27-28/
├── aws_ec2.yml
├── mysql-playbook.yml
├── vault.yml          # Encrypted - not uploaded
├── .gitignore
└── README.md
```

## AWS Dynamic Inventory

The `aws_ec2.yml` file uses the `amazon.aws.aws_ec2` plugin to discover running EC2 instances in the Stockholm region.

Instances are filtered using the tag `server=ivolve`.

To list discovered hosts:

```bash
ansible-inventory -i aws_ec2.yml --graph
```

## Ansible Vault

Ansible Vault encrypts the MySQL user's password so it isn't stored in plaintext in the playbook.

To create an encrypted Vault file:

```bash
ansible-vault create vault.yml
```

The encrypted file contains a variable named `mysql_user_password`.

The Vault file and SSH private key are excluded from GitHub.

## Running the Playbook

Activate the project's Python virtual environment and run:

```bash
source .venv/bin/activate

ANSIBLE_COLLECTIONS_PATH="$PWD/.venv/collections" \
ansible-playbook -i aws_ec2.yml mysql-playbook.yml \
  --ask-vault-pass
```

The playbook installs MySQL, starts the service, creates the database, and configures the database user.

## Verification

Check the MySQL service:

```bash
sudo systemctl status mysql
```

List the databases:

```bash
sudo mysql -e "SHOW DATABASES;"
```

Verify user permissions:

```bash
sudo mysql -e "SHOW GRANTS FOR 'ivolveuser'@'localhost';"
```

Test the database login:

```bash
mysql -u ivolveuser -p -D ivolve
```

Then run:

```sql
SHOW DATABASES;
```

## Result

Both labs were completed successfully.

- EC2 discovery using AWS Dynamic Inventory
- Automated MySQL installation and configuration
- Database and user creation
- Encrypted password management with Ansible Vault
- Successful database connectivity and verification

## What I Learned

This project helped me understand how Ansible can combine cloud resource discovery, remote configuration, and secret management into one automated workflow.

It also demonstrated how infrastructure automation can be made more secure and reusable without manually configuring each server.
