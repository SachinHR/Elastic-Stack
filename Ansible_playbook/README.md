## Setup Docker and jenkins using ansible playbook

### Prerequisite
```
sudo pip install requests google-auth
```
Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

### Steps

Step 1: Create a dedicated inventory directory
```
sudo mkdir -p /opt/ansible/inventory
```
Step 2: Create a Google IAM service account. It will be used by the Ansible server to authenticate against google cloud for dynamic inventory. 

Step 3: Save the service account file as service-account.json inside /opt/ansible/inventory folder.

Step 4: Create a file named gcp.yaml inside /opt/ansible/inventory directory and add the following content.
```
---
plugin: gcp_compute
projects:
  - <gcp-project-id>
auth_kind: serviceaccount
service_account_file: /opt/ansible/inventory/service-account.json
```
Replace <gcp-project-id> with your google cloud project id. 

Step 5: Change the inventory folder’s permission to 755.
```
sudo chmod -R 755 /opt/ansible
```

Step 6: Now we have all the required configurations for gcp dynamic inventory. Lets test it by listing out all the instances using the ansible-inventory command. Make sure you are running this command from /opt/ansible/inventory directory.
```
ansible-inventory --list -i gcp.yaml
```

Step 7: Open /etc/ansible/ansible.cfg file Add the dynamic inventory config path under default configs.
```
[defaults]

# some basic default values...

inventory      = /opt/ansible/inventory/gcp.yaml
```
With this configuration, by default all the ansible action will consider the gcp config as its inventory file.

You can verify the default inventory configuration by executing the inventory list command.
```
ansible-inventory --list
```

## Grouping GCP Resources

I have added grouping "staging" using labels and Zone. These will return all VMS, which matches the filter.
```
---
plugin: gcp_compute
projects:
  - <gcp-project-id>
auth_kind: serviceaccount
service_account_file: /opt/ansible/inventory/service-account.json
keyed_groups:
  - key: labels
    prefix: label
  - key: zone
    prefix: zone
groups:
  staging: "'jenkins-docker' in name"
```
here jenkins-docker is VM name
Modify your inventory file with the above keyed_groups & groups and execute the following command to list the groups.
```
ansible-inventory --graph
```

### Run playbook
```
cd Ansible_playbook

ansible-playbook playbook.yaml
```
