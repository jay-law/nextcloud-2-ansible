# nextcloud-2-ansible

```bash

$ cp ~/Downloads/nextcloud-2.pem .

$ chmod 600 nextcloud-2.pem

# grab Elastic IP from AWS console.  Replace existing IP in hosts, nextcloud_virtual_host.conf, and vars.yml files.

# Run playbook
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-2.pem
```

```bash

# Execute a single task
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-2.pem --tags "nextcloud"

# Execute all three tasks in roles/common/tasks
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-2.pem --tags "web, database, nextcloud"

# Install the AppsOrder app within Nextcloud
$ ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-2.pem --tags "apps_apporder"



ansible -m ping --private-key nextcloud-2.pem -i hosts all


```