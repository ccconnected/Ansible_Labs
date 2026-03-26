```
13 Ansible Playbooks

a) Playbook execution to create users

Lab Activities

Your engineering team needs to push service accounts for new users and teams to some of your systems. You have decided to do that in Ansible.
Verify your /root/hosts file, and /root/user_create.yaml files. You may note that the hosts file has variables set that we will be using to push the correct files in our first playbook.
Push the user prod_engineer to the prod servers and the dev_engineer user to the dev servers. In the dev environment the user is allowed to be in the admin group.


Solution
cat /root/hosts

Note: There are variables now assigned to each of the servers (env)
cat /root/user_create.yaml

Note: This will push users to the servers and add the group where appropriate.

Run the Playbook push the users.
ansible-playbook -i /root/hosts /root/user_create.yaml

Run an ad hoc command to verify users were added and place in groups.
ansible servers -i /root/hosts -m shell -a "tail -1 /etc/passwd; grep admin /etc/group"


controlplane $ cat /root/hosts
[servers]
controlplane env=prod
node01 env=dev
controlplane $ cat /root/user_create.yaml
---

- name: User push to environment servers
  hosts: servers
  vars:
  gather_facts: True
  become: True
  tasks:

  - name: Debug env variables just to see them
    debug:
      var: env

  - name: Create the prod_engineer user
    user:
      name: prod_engineer
      comment: Prod engineer 
    when: '"prod" in env'

  - name: Create the dev_engineer user and give admin
    user:
      name: dev_engineer
      comment: Dev engineer
      groups: admin
    when: '"dev" in env'

controlplane $ ansible-playbook -i /root/hosts /root/user_create.yaml

PLAY [User push to environment servers] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [Debug env variables just to see them] ******************************************************************************************************************************************************************************
ok: [controlplane] => {
    "env": "prod"
}
ok: [node01] => {
    "env": "dev"
}

TASK [Create the prod_engineer user] *************************************************************************************************************************************************************************************
skipping: [node01]
changed: [controlplane]

TASK [Create the dev_engineer user and give admin] ***********************************************************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
node01                     : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

controlplane $ ansible servers -i /root/hosts -m shell -a "tail -1 /etc/passwd; grep admin /etc/group"
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
node01 | CHANGED | rc=0 >>
dev_engineer:x:1001:1001:Dev engineer:/home/dev_engineer:/bin/sh
admin:x:117:dev_engineer
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
controlplane | CHANGED | rc=0 >>
prod_engineer:x:1001:1001:Prod engineer:/home/prod_engineer:/bin/sh
admin:x:117:
lpadmin:x:127:

------------------------------------------------------------------------------
b) Playbook execution to add keys to users

Lab Activities
Verify your /root/user_key_add.yaml file. You may note that the hosts file has variables set that we will be using to push the correct files in our first playbook.
Modify and generate keys for the users.


Solution
cat /root/user_key_add.yaml
Note: This will generate keys for the existing users.

Run the Playbook push the users.

ansible-playbook -i /root/hosts /root/user_key_add.yaml
Run an ad hoc command to verify keys were added to the users.

ansible servers -i /root/hosts -m shell -a "ls /home/*engineer/.ssh"


controlplane $ cat /root/user_key_add.yaml
---

- name: User push to environment servers
  hosts: servers
  vars:
  gather_facts: True
  become: True
  tasks:

  - name: Debug env variables just to see them
    debug:
      var: env

  - name: Create the prod_engineer user
    user:
      name: prod_engineer
      comment: Prod engineer 
      generate_ssh_key: yes
      ssh_key_bits: 2048
      ssh_key_file: .ssh/id_rsa
    when: '"prod" in env'

  - name: Create the dev_engineer user and give admin
    user:
      name: dev_engineer
      comment: Dev engineer
      groups: admin
      generate_ssh_key: yes
      ssh_key_bits: 2048
      ssh_key_file: .ssh/id_rsa
    when: '"dev" in env'

controlplane $ ansible-playbook -i /root/hosts /root/user_key_add.yaml

PLAY [User push to environment servers] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [Debug env variables just to see them] ******************************************************************************************************************************************************************************
ok: [controlplane] => {
    "env": "prod"
}
ok: [node01] => {
    "env": "dev"
}

TASK [Create the prod_engineer user] *************************************************************************************************************************************************************************************
skipping: [node01]
changed: [controlplane]

TASK [Create the dev_engineer user and give admin] ***********************************************************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
node01                     : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

controlplane $ ansible servers -i /root/hosts -m shell -a "ls /home/*engineer/.ssh"
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
node01 | CHANGED | rc=0 >>
id_rsa
id_rsa.pub
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
controlplane | CHANGED | rc=0 >>
id_rsa
id_rsa.pub


Look at you, learning Ansible! You Created users and gave them keys in their /home/username/.ssh directories!
```
