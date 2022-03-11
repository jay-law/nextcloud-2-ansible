# nextcloud-2-ansible

```bash

$ cp ~/Downloads/nextcloud-1.pem .

$ chmod 600 nextcloud-1.pem

# grab Elastic IP from AWS console.  Replace existing IP in hosts, nextcloud_virtual_host.conf, and vars.yml files.

# Run playbook
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-1.pem
```

```bash

# Execute a single task
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-1.pem --tags "nextcloud"

# Execute all three tasks in roles/common/tasks
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-1.pem --tags "web, database, nextcloud"

# Install the AppsOrder app within Nextcloud
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-1.pem --tags "apps_apporder"



ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-1.pem --tags "get_ip"
```