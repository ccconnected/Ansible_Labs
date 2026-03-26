```
12 Ansible Playbooks

Playbook execution to push facts to all servers

Lab Activities
Your team has decided to stamp variables to your servers so that they know information about themselves relative to their job functions.
Verify your /root/hosts file, /root/custom_fact_push.yaml, and /root/group_by.yaml files. You may note that the hosts file has variables set that we will be using to push the correct files in our first playbook. In our second playbook we will be grouping by those facts so that the playbook can dynamically pull facts and group by those facts that servers "know" about themselves in the environment.
Push the custom fact to all servers using custom_fact_push.yaml file
Verify that custom facts have been set by grouping servers together and checking the host groups that are dynamically pulled from servers based on those facts.


Solution
cat /root/hosts

Note: There are variables now assigned to each of the servers (env)
cat /root/custom_fact_push.yaml

Note: this will make the directory to push the patching.fact file into. The files start out named differently, but are the same when placed on the servers
cat /root/group_by.yaml

Note: This playbook shows you the groups that the playbook starts with, pulls information from each server's custom facts, and then shows the playbooks that Ansible can use at the end.

Run the Playbook push the custom facts
ansible-playbook -i /root/hosts /root/custom_fact_push.yaml

Run the playbook to verify groupings by custom facts.
ansible-playbook -i /root/hosts /root/group_by.yaml


controlplane $ cat /root/hosts
[servers]
controlplane env=prod
node01 env=dev
controlplane $ cat /root/custom_fact_push.yaml
---

- name: Patching Facts Push
  hosts: servers
  vars:
  gather_facts: True
  become: True
  tasks:

  - name: Debug env variables just to see them
    debug:
      var: env

  - name: Create the facts directory in case it isn't there
    file:
      path: /etc/ansible/facts.d
      state: directory

  - name: Push over the prod_patching.fact file if prod
    copy:
      src: /root/prod_patching.fact
      dest: "/etc/ansible/facts.d/patching.fact"
    when: '"prod" in env'

  - name: Push over the dev_patching.fact file if dev
    copy:
      src: /root/dev_patching.fact
      dest: "/etc/ansible/facts.d/patching.fact"
    when: '"dev" in env'

controlplane $ cat /root/group_by.yaml
---

- name: System Facts and Group by those facts
  hosts: servers
  vars:
  gather_facts: True
  become: True
  tasks:

    - name: "Show Groups active in this playbook at start"
      debug:
        msg: "{{ group_names }}"

    - name: Setting groups for rebootgroup
      group_by:
        key: "{{ ansible_local.patching.patch_group }}"
      failed_when: false

    - name: Checking Rebooting flag
      group_by:
        key: Rebooting
      when: '"true" in ansible_local.patching.Rebooting'

    - name: "Show Groups active in this playbook at end"
      debug:
        msg: "{{ group_names }}"

controlplane $ cat /root/prod_patching.fact
{
  "patch_group": "group2",
  "Rebooting":   "true",
  "appserver":   "true",
  "database":    "false",
  "webserver":   "false"
}
controlplane $ cat /root/dev_patching.fact
{
  "patch_group": "group1",
  "Rebooting":   "false",
  "appserver":   "true",
  "database":    "false",
  "webserver":   "false"
}


controlplane $ ansible-playbook -i /root/hosts /root/custom_fact_push.yaml

PLAY [Patching Facts Push] ***********************************************************************************************************************************************************************************************

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

TASK [Create the facts directory in case it isn't there] *****************************************************************************************************************************************************************
changed: [node01]
changed: [controlplane]

TASK [Push over the prod_patching.fact file if prod] *********************************************************************************************************************************************************************
skipping: [node01]
changed: [controlplane]

TASK [Push over the dev_patching.fact file if dev] ***********************************************************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=4    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
node01                     : ok=4    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

controlplane $ ansible-playbook -i /root/hosts /root/group_by.yaml

PLAY [System Facts and Group by those facts] *****************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [Show Groups active in this playbook at start] **********************************************************************************************************************************************************************
ok: [controlplane] => {
    "msg": [
        "servers"
    ]
}
ok: [node01] => {
    "msg": [
        "servers"
    ]
}

TASK [Setting groups for rebootgroup] ************************************************************************************************************************************************************************************
ok: [controlplane]
ok: [node01]

TASK [Checking Rebooting flag] *******************************************************************************************************************************************************************************************
ok: [controlplane]
skipping: [node01]

TASK [Show Groups active in this playbook at end] ************************************************************************************************************************************************************************
ok: [controlplane] => {
    "msg": [
        "Rebooting",
        "group2",
        "servers"
    ]
}
ok: [node01] => {
    "msg": [
        "group1",
        "servers"
    ]
}

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node01                     : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   


Look at you, learning Ansible! You pushed custom facts and then pulled them into different host groups. You solved this challenge!
```
