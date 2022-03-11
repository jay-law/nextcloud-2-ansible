# Overview

This tutorial is designed to walk users through installing Nextcloud in AWS (Amazon Web Services) using Terraform and Ansible.  It's more of a working example opposed to an in-depth tutorial.

In the first part of the tutorial, Terraform will be used to:
- Create an EC2 instance
- Create security groups and assign them to the EC2 instance
- Create an elastic IP address and assign it to the EC2 instance
- Attach pem file to EC2 instance

In the second part of the tutorial, Ansible will be used to:
- Install and configure MySQL (database)
    - `roles/common/tasks/configure_database.yml`
- Install and configure Apache (web server)
    - `roles/common/tasks/configure_web.yml`
- Install and configure Nextcloud (application)
    - `roles/common/tasks/configure_nextcloud.yml` 
- Install a Nextcloud app
    - `roles/apps/tasks/main.yml`

Tested with the following:
- Local machine
    - Ubuntu 20.04.4 LTS (64-bit)
    - Terraform 1.1.6
    - Ansible (core) 2.12.2
    - Python 3.8.10
- Remote machine (AWS EC2 instance)
    - Ubuntu 20.04 LTS (64-bit)

**NOTE** Part of this documentation is copied from the previous tutorial (`nextcloud-1-ansible`).  Some sections can be skipped if they were already performed.  For example, Ansible doesn't need to be installed again.

# Prerequisites

## Technologies Used

- **Nextcloud** - A self-hosted productivity platform.  
    - Skills required - None to little
- **AWS** - Cloud platform used to host the infrastructure.
    - Skills required- Very little
- **Terraform** - Tool used to create the infrastructure.
    - Skills required - None to little
- **Ansible** - Tool used to modify the software within the infrastructure.
    - Skills required - Little to moderate

## Ensure AWS Creds Exist Locally

An existing AWS account is required.  Follow the instructions in this link to create an account - https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/.

Additionally, your AWS creds will need to configured locally.  Follow this link to get those setup - https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html.

```bash
# Ensure AWS creds exist in `~/.aws/credentials`
$ cat ~/.aws/credentials
[default]
aws_access_key_id=djjgoijd
aws_secret_access_key=8dh3hgf8hfoih
```

# Configure Local Environment

## Clone This Repo

```bash
# Clone repo with SSH if GitHub account is configured with SSH keys
$ git clone git@github.com:jay-law/nextcloud-2-ansible.git

# or
# Clone repo with HTTPS
$ git clone https://github.com/jay-law/nextcloud-2-ansible.git

# cd into the repo directory
$ cd nextcloud-2-ansible/ 
```

## Copy Permission (.pem) File

Copy the `nextcloud-2.pem` file (created in the `nextcloud-2-terraform` tutorial) over to the `nextcloud-2-ansible` directory (current working directory).

```bash
# This code might work if the pem was downloaded to the default 'Downloads' directory
$ cp ~/Downloads/nextcloud-2.pem .

# Change the permission or Ansible will complain
$ sudo chmod 600 nextcloud-2.pem
```

## Get IP Address

The EC2 instance created from `nextcloud-2-terraform` will have an elastic IP address.  There are multiple ways to get the address.  Once of the easiest is to manually copy the `Public IPv4 address` field from the EC2 Dashboard.

Replace the existing IP in the following locations:
- `hosts` file 
- `elastic_ip` variable in the `group_vars/vars.yml` file
- `Servername` variable in the `remote_config_files/nextcloud_virtual_host.conf` file

## Install Ansible (Locally)

Ansible will be used to install Nextcloud on the EC2 instance.

Guide - https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu

```bash
# Update repo
$ sudo apt update

# Install required packages
$ sudo apt install software-properties-common

# Add the Ansible repo
$ sudo add-apt-repository --yes --update ppa:ansible/ansible

# Install
$ sudo apt install ansible

# Verify install
$ ansible --version
```

# Install Nextcloud

```bash
# Perform a test run to ensure no issues will arise
$ ansible all -m ping -i hosts -u ubuntu --private-key nextcloud-2.pem

44.194.224.88 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

# Run playbbok to install and configure Nextcloud
# Tags are being used to run specific roles.  See  'roles/common/tasks/main.yml' for the tag association
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-2.pem --tags "web, database, nextcloud"

PLAY [all] *******************

TASK [Gathering Facts] *******************
ok: [44.194.224.88]

TASK [common : Run 'apt update'] *******************
changed: [44.194.224.88]

TASK [common : Installing packages for MySQL] *******************
changed: [44.194.224.88]

...

TASK [common : Restarting Apache] *******************
changed: [44.194.224.88]

PLAY RECAP *******************
44.194.224.88              : ok=27   changed=23   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Nextcloud should be up and running at the IP in the hosts file.  Paste the IP into your browser.  Use the admin username and password specified in the `install_nextcloud.yml` file to login.  No SSL was configured so you might get a 'Secure Site Not Available' warning.  Just select 'Continue to HTTP Site' in Firefox. 

Tinker around with these scripts to build on your AWS, Terraform, and Ansible experience.

Be sure to destroy the instance once you are finished tinkering.  These scripts have very little security.

```bash
$ terraform destroy
```

# Other

## Links

- [Official server install documentation](https://docs.nextcloud.com/server/stable/admin_manual/installation/source_installation.html)
    - The documentation is very well written.  This guide (`nextcloud-2-ansible`) was created by reading the official documentation, executing the Linux commands provided by the documentation on an EC2 instance, then converting the commands into Ansible tasks.

## Additional Execution Commands 

Specific Ansible roles can be executed without having to run the entire playbook.  This is useful for debugging or testing newly written code.

Reexecuting some roles will throw errors as they are not configured to check for existing conditions.  For example, rerunning the `database` role will throw an error because it will try to create a user that already exists.

```bash

# Execute a single task
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-2.pem --tags "nextcloud"

# Execute all three tasks in roles/common/tasks (as used above)
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-2.pem --tags "web, database, nextcloud"

# Install the AppsOrder app within Nextcloud
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-2.pem --tags "apps_apporder"
```