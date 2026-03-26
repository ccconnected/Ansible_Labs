```
15 Ansible Playbooks

a) Create and examine roles
Your team has been using Ansible and decided that they need to have better reusability and modularity to their code. You have been tasked with testing and deploying galaxy to see how inherited roles work in Ansible.

Lab Activities
Verify your /root/hosts file exists.
Create a directory called /root/playbooks/roles
Create two galaxies inside of /root/playbooks/roles called update and install .
Update the update and install roles to properly do their functions, which can be referenced from /root/packages_*.yaml files.


Solution
cat /root/hosts

Note: There are variables now assigned to each of the servers (env)
make the directory structure as required.

mkdir -p /root/playbooks/roles
cd /root/playbooks/roles

Note: You're now moved into that directory and can create the required roles using ansible-galaxy command

ls -l
tree
ansible-galaxy init update
and check

ls -l
tree
Now create the second role

ls -l
ansible-galaxy init install
and check again

ls -l
tree

Go into the update directory to update and create the right files.
vi /root/playbooks/roles/update/tasks/main.yml
---
# tasks file for update

- include_tasks: update.yaml
  tags:
    - update

You also have to create that file correctly within the tasks.

vi /root/playbooks/roles/update/tasks/update.yaml
- name: Upgrade all packages to the latest version
  apt:
    name: "*"
    state: latest
  tags:
    - update

Now you have to do that for the second directory

Go into the update directory to update and create the right files.
vi /root/playbooks/roles/install/tasks/main.yml
---
# tasks file for install

- include_tasks: install.yaml
  tags:
    - install

You also have to create that file correctly within the tasks.
vi /root/playbooks/roles/install/tasks/install.yaml

- name: Debug env variables just to see them
  debug:
    var: app
  tags:
    - install

- name: Install apache2 on the web server
  apt:
    pkg: 
    - apache2
    - php
    state: present
  when: '"web" in app'
  tags:
    - install

- name: Install mariadb on the web server
  apt:
    pkg: 
    - mariadb-server
    - mariadb-client
    state: present
  when: '"db" in app'
  tags:
    - install


controlplane $ cat /root/hosts
[servers]
controlplane env=prod app=db
node01 env=dev app=web
controlplane $ mkdir -p /root/playbooks/roles
controlplane $ cd /root/playbooks/roles
controlplane $ ls -l
total 0
controlplane $ tree
.

0 directories, 0 files
controlplane $ ls -l
total 0
controlplane $ tree
.

0 directories, 0 files
controlplane $ ansible-galaxy init update
- Role update was created successfully
controlplane $ ls -l
total 4
drwxr-xr-x 10 root root 4096 Jan 16 21:43 update
controlplane $ tree
.
`-- update
    |-- README.md
    |-- defaults
    |   `-- main.yml
    |-- files
    |-- handlers
    |   `-- main.yml
    |-- meta
    |   `-- main.yml
    |-- tasks
    |   `-- main.yml
    |-- templates
    |-- tests
    |   |-- inventory
    |   `-- test.yml
    `-- vars
        `-- main.yml

9 directories, 8 files
controlplane $ cd ..
controlplane $ ls -l
total 4
drwxr-xr-x 3 root root 4096 Jan 16 21:43 roles
controlplane $ cd roles/
controlplane $ ls -l
total 4
drwxr-xr-x 10 root root 4096 Jan 16 21:43 update
controlplane $ ansible-galaxy init install
- Role install was created successfully
controlplane $ ls -l
total 8
drwxr-xr-x 10 root root 4096 Jan 16 21:46 install
drwxr-xr-x 10 root root 4096 Jan 16 21:43 update
controlplane $ tree
.
|-- install
|   |-- README.md
|   |-- defaults
|   |   `-- main.yml
|   |-- files
|   |-- handlers
|   |   `-- main.yml
|   |-- meta
|   |   `-- main.yml
|   |-- tasks
|   |   `-- main.yml
|   |-- templates
|   |-- tests
|   |   |-- inventory
|   |   `-- test.yml
|   `-- vars
|       `-- main.yml
`-- update
    |-- README.md
    |-- defaults
    |   `-- main.yml
    |-- files
    |-- handlers
    |   `-- main.yml
    |-- meta
    |   `-- main.yml
    |-- tasks
    |   `-- main.yml
    |-- templates
    |-- tests
    |   |-- inventory
    |   `-- test.yml
    `-- vars
        `-- main.yml

18 directories, 16 files
controlplane $ vi /root/playbooks/roles/update/tasks/main.yml
controlplane $ vi /root/playbooks/roles/update/tasks/update.yaml
controlplane $ vi /root/playbooks/roles/install/tasks/main.yml
controlplane $ vi /root/playbooks/roles/install/tasks/install.yaml

------------------------------------------------------------------------------
b) Playbook execution that inherits roles

Lab Activities

So now that we have the roles correct, let's make a playbook that inherits those roles based on correct tags. By default if no tags are given, it will run all roles.
Create a playbook to inhert roles. Name it /root/playbooks/environment.yaml
Test the playbook with and without the different tags install and update .


Solution
Create the correct playbook
vi /root/playbooks/environment.yaml

Copy in this value:

- hosts: all
  gather_facts: true
  vars:
  tasks:

  roles:
  - update
  - install

Run the Playbook to see it work with no tags
This playbook may take some time as it is updating the servers. Possibly even a minute.
ansible-playbook -i /root/hosts /root/playbooks/environment.yaml

Run it with just the update tag
ansible-playbook -i /root/hosts /root/playbooks/environment.yaml --tags=update

Run it with just the install tag
ansible-playbook -i /root/hosts /root/playbooks/environment.yaml --tags=install

How did these compare?

Did it only do the work requried in each tag?

Why do you think the default is to just use all inherited roles?

controlplane $ vi /root/playbooks/environment.yaml
controlplane $ ansible-playbook -i /root/hosts /root/playbooks/environment.yaml

PLAY [all] ***************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [update : include_tasks] ********************************************************************************************************************************************************************************************
included: /root/playbooks/roles/update/tasks/update.yaml for controlplane, node01

TASK [update : Upgrade all packages to the latest version] ***************************************************************************************************************************************************************
changed: [node01]
changed: [controlplane]

TASK [install : include_tasks] *******************************************************************************************************************************************************************************************
included: /root/playbooks/roles/install/tasks/install.yaml for controlplane, node01

TASK [install : Debug env variables just to see them] ********************************************************************************************************************************************************************
ok: [controlplane] => {
    "app": "db"
}
ok: [node01] => {
    "app": "web"
}

TASK [install : Install apache2 on the web server] ***********************************************************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

TASK [install : Install mariadb on the web server] ***********************************************************************************************************************************************************************
skipping: [node01]
changed: [controlplane]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=6    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
node01                     : ok=6    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

controlplane $ ansible-playbook -i /root/hosts /root/playbooks/environment.yaml --tags=update

PLAY [all] ***************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [update : include_tasks] ********************************************************************************************************************************************************************************************
included: /root/playbooks/roles/update/tasks/update.yaml for controlplane, node01

TASK [update : Upgrade all packages to the latest version] ***************************************************************************************************************************************************************
ok: [node01]
ok: [controlplane]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node01                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

controlplane $ ansible-playbook -i /root/hosts /root/playbooks/environment.yaml --tags=install

PLAY [all] ***************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [install : include_tasks] *******************************************************************************************************************************************************************************************
included: /root/playbooks/roles/install/tasks/install.yaml for controlplane, node01

TASK [install : Debug env variables just to see them] ********************************************************************************************************************************************************************
ok: [node01] => {
    "app": "web"
}
ok: [controlplane] => {
    "app": "db"
}

TASK [install : Install apache2 on the web server] ***********************************************************************************************************************************************************************
skipping: [controlplane]
ok: [node01]

TASK [install : Install mariadb on the web server] ***********************************************************************************************************************************************************************
skipping: [node01]
ok: [controlplane]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
node01                     : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   


Look at you, learning Ansible! You created roles via ansible-galaxy and then inherited them inside of an ansible playbook!
```
