# nextcloud-2-ansible

```bash

cp ~/Downloads/nextcloud-1.pem .

chmod 600 nextcloud-1.pem

# grab Elastic IP from AWS console.  Replace existing IP in hosts, nextcloud_virtual_host.conf, and vars.yml files.

ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-1.pem

ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-1.pem --tags "nextcloud"

ansible-playbook install_nextcloud.yml -i hosts --private-key nextcloud-1.pem --tags "web, database, nextcloud"

```