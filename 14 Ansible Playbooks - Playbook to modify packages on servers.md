```
14 Ansible Playbooks

a) Playbook execution update packages

Lab Activities
Your team has been tasked to update all servers in the environment to maintain compliance with your GRC (Governance, Risk, and Compliance) Security team. You also need to configure some new servers for software in the environment.
Verify your /root/hosts file, and /root/packages_update.yaml files. You may note that the hosts file has variables set that we will be using to push the correct files in our first playbook.
Update all the packages to the newest version


Solution
cat /root/hosts

Note: There are variables now assigned to each of the servers (env)
cat /root/packages_update.yaml

Note: This will update all current packages on the server

Run the Playbook update all the packages.
ansible-playbook -i /root/hosts /root/packages_update.yaml

Better run it twice just to compare the output.


controlplane $ cat /root/hosts
[servers]
controlplane env=prod app=db
node01 env=dev app=web
controlplane $ cat /root/packages_update.yaml
---

- name: Upgrade Packages
  hosts: servers
  vars:
  gather_facts: True
  become: True
  tasks:

  - name: Upgrade all packages to the latest version
    apt:
      name: "*"
      state: latest

controlplane $ ansible-playbook -i /root/hosts /root/packages_update.yaml

PLAY [Upgrade Packages] **************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [Upgrade all packages to the latest version] ************************************************************************************************************************************************************************
changed: [node01]
changed: [controlplane]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node01                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

controlplane $ ansible-playbook -i /root/hosts /root/packages_update.yaml

PLAY [Upgrade Packages] **************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [Upgrade all packages to the latest version] ************************************************************************************************************************************************************************
ok: [node01]
ok: [controlplane]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node01                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

controlplane $ 

------------------------------------------------------------------------------
b) Playbook execution to add packages based on env variables

Lab Activities
Verify your /root/packages_install.yaml file. You may note that the hosts file has variables set that we will be using to install packages on the right servers.


Solution
cat /root/packages_install.yaml

Note: This will conditionally install certain packages on hosts.

Run the Playbook push the users.
ansible-playbook -i /root/hosts /root/packages_install.yaml

Run it again to see that nothing has to change. This demonstrates idempotency, where the result of the previous run is the same in subsequent runs.


controlplane $ cat /root/packages_install.yaml
---

- name: Package install for environment
  hosts: servers
  vars:
  gather_facts: True
  become: True
  tasks:

  - name: Debug env variables just to see them
    debug:
      var: app

  - name: Install apache2 on the web server
    apt:
      pkg: 
      - apache2
      - php
      state: present
    when: '"web" in app'

  - name: Install mariadb on the web server
    apt:
      pkg: 
      - mariadb-server
      - mariadb-client
      state: present
    when: '"db" in app'

controlplane $ ansible-playbook -i /root/hosts /root/packages_install.yaml

PLAY [Package install for environment] ***********************************************************************************************************************************************************************************

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
    "app": "db"
}
ok: [node01] => {
    "app": "web"
}

TASK [Install apache2 on the web server] *********************************************************************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

TASK [Install mariadb on the web server] *********************************************************************************************************************************************************************************
skipping: [node01]
changed: [controlplane]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
node01                     : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

controlplane $ ansible-playbook -i /root/hosts /root/packages_install.yaml

PLAY [Package install for environment] ***********************************************************************************************************************************************************************************

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
    "app": "db"
}
ok: [node01] => {
    "app": "web"
}

TASK [Install apache2 on the web server] *********************************************************************************************************************************************************************************
skipping: [controlplane]
ok: [node01]

TASK [Install mariadb on the web server] *********************************************************************************************************************************************************************************
skipping: [node01]
ok: [controlplane]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
node01                     : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   


Look at you, learning Ansible! You updated and pushed packages. You installed new packages to newly deployed systems. You solved this challenge!
```
