```
17 Ansible Playbooks

Playbook execution remove security violations

Lab Activities
You cleaned up the golden image over in lab Updating Golden Image https://killercoda.com/het-tanis/course/Linux-Labs/203-updating-golden-image . Now your team needs you to deploy that change to all the active systems.
Verify your /root/hosts file, /root/remove_packages.j2 and /root/remove_packages.yaml files.
Remove the bzip2 and containerd packages from all servers to ensure that old software is no longer on your systems and to stop developer abuse of containerd on all systems.


Solution
cat /root/hosts
cat /root/remove_packages.yaml

You may want to remove the commented debug variables before you run this. What are the individual tasks attempting to do with this playbook?

Now that we've reviewed the playbook, let's look at the Jinja2 template that will be stamped to this server at the end of the execution.

cat /root/remove_packages.j2

What are the variables being used? what are the expected outcomes?

Run the playbook to see the removal of packages from all servers.
ansible-playbook -i /root/hosts /root/remove_packages.yaml

Did the playbook execute with no issues? If it did not, run it again and see what the output is.

Verify the output of the reports that you've run.
for report in $(ls /root/*report*.txt); do echo "checking $report"; cat $report; done

Are the output reports what you expected? Why or why not?


controlplane $ cat /root/hosts
[servers]
controlplane
node01
controlplane $ cat /root/remove_packages.yaml
---

- name: Package removal for environment
  hosts: servers
  vars:
  gather_facts: True
  become: True
  tasks:

  - name: Remove Apps for security reasons
    apt:
      pkg: 
      - containerd
      - bzip2
      purge: yes
      state: absent
    register: removed_apps

  # - name: Debug Removed Apps
  #   debug:
  #     var: removed_apps
      
  - name: Verify that the packages were removed
    shell: "dpkg -l | grep -i {{item}}"
    loop:
      - bzip2
      - containerd
    failed_when: false
    register: verify_apps

 # - name: Debug Removed Apps
 #   debug:
 #     var: verify_apps

  - name: Report out all removals to just main server
    template:
      src: /root/remove_packages.j2
      dest: "/root/remove_packages_report.{{ansible_date_time.iso8601_basic_short}}.txt"
    run_once: yes
    delegate_to: localhost
    controlplane $ cat /root/remove_packages.j2
This is a system Validation at {{ ansible_date_time.time }} on {{ ansible_date_time.date }}

Unreachable systems:
----------------------------------------------
{% for host in ansible_play_hosts_all %}
{% if host not in ansible_play_hosts %}
{{ host }}
{% endif  %}
{% endfor %}

Removed packages:
----------------------------------------------
{% for host in ansible_play_hosts_all %}
{% if hostvars[host].verify_apps is defined %}
{% for package in hostvars[host].verify_apps.results %}
{% if package.rc == 1 %}
{{ host }} - Package {{ package.item }} was removed.
{% endif %}
{% endfor %}
{% endif %}
{% endfor %}
controlplane $ ansible-playbook -i /root/hosts /root/remove_packages.yaml

PLAY [Package removal for environment] ***********************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [Remove Apps for security reasons] **********************************************************************************************************************************************************************************
changed: [node01]
changed: [controlplane]

TASK [Verify that the packages were removed] *****************************************************************************************************************************************************************************
changed: [node01] => (item=bzip2)
changed: [controlplane] => (item=bzip2)
changed: [node01] => (item=containerd)
changed: [controlplane] => (item=containerd)

TASK [Report out all removals to just main server] ***********************************************************************************************************************************************************************
changed: [controlplane -> localhost]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node01                     : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

controlplane $ ansible-playbook -i /root/hosts /root/remove_packages.yaml

PLAY [Package removal for environment] ***********************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will 
default to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 
2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [Remove Apps for security reasons] **********************************************************************************************************************************************************************************
ok: [node01]
ok: [controlplane]

TASK [Verify that the packages were removed] *****************************************************************************************************************************************************************************
changed: [node01] => (item=bzip2)
changed: [controlplane] => (item=bzip2)
changed: [node01] => (item=containerd)
changed: [controlplane] => (item=containerd)

TASK [Report out all removals to just main server] ***********************************************************************************************************************************************************************
changed: [controlplane -> localhost]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
controlplane               : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node01                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

controlplane $ for report in $(ls /root/*report*.txt); do echo "checking $report"; cat $report; done
checking /root/remove_packages_report.20240117T154746.txt
This is a system Validation at 15:47:46 on 2024-01-17

Unreachable systems:
----------------------------------------------

Removed packages:
----------------------------------------------
controlplane - Package bzip2 was removed.
controlplane - Package containerd was removed.
node01 - Package bzip2 was removed.
node01 - Package containerd was removed.
checking /root/remove_packages_report.20240117T154823.txt
This is a system Validation at 15:48:23 on 2024-01-17

Unreachable systems:
----------------------------------------------

Removed packages:
----------------------------------------------
controlplane - Package bzip2 was removed.
controlplane - Package containerd was removed.
node01 - Package bzip2 was removed.
node01 - Package containerd was removed.


Look at you, learning Ansible! You used Ansible to remove packages that were not security compliant in your organization.
```
