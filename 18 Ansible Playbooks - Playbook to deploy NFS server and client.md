```
18 Ansible Playbooks

Playbook execution deploy server

Lab Activities
You deployed a NFS server by hand over in Configure NFS Share https://killercoda.com/het-tanis/course/Linux-Labs/100-configure-nfs-share . Now your team needs to deploy the NFS server and client mount points to multiple new servers through an automated process.
Verify your /root/hosts file and see that you have defined variables for type in this deployment.
Create a galaxy structure and write or copy in the answers of the tasks and handlers for deploying the NFS server.


Solution
cat /root/hosts

Create a structure for a role based deployment with Ansible Galaxy.
mkdir -p /root/ansible/roles
cd /root/ansible/roles

create your ansible galaxy structures

ansible-galaxy init nfs_server
ansible-galaxy init nfs_client

Copy in the ansible playbook with the inherited roles.
cp /answers/nfs_deploy.yaml /root/ansible/nfs_deploy.yaml

Inspect the playbook. Does it make sense that the roles are going to be inherited if certain conditions about the servers are met? Those can come from an inventory or an environment file to define what servers inherit what roles.
cat /root/ansible/nfs_deploy.yaml

Copy in the playbook tasks and handlers for the NFS server deploy
cp /answers/nfs_server_task_main.yaml /root/ansible/roles/nfs_server/tasks/main.yml
cp /answers/nfs_server_handler_main.yaml /root/ansible/roles/nfs_server/handlers/main.yml

Inspect the files you have created to see what they should do.

cat /root/ansible/roles/nfs_server/tasks/main.yml
echo "----------"
cat /root/ansible/roles/nfs_server/handlers/main.yml

Execute the playbook and see the deployment of the server.
ansible-playbook -i /root/hosts /root/ansible/nfs_deploy.yaml

What output do you see? What happens if you run this again?

Check that the server has been started on node01 and a filesystem has been shared out.

ssh node01 'hostname; showmount -e'


controlplane $ cat /root/hosts
[servers]
controlplane type=client
node01 type=server
controlplane $ mkdir -p /root/ansible/roles
controlplane $ cd /root/ansible/roles
controlplane $ ansible-galaxy init nfs_server
- Role nfs_server was created successfully
controlplane $ ansible-galaxy init nfs_client
- Role nfs_client was created successfully
controlplane $ cp /answers/nfs_deploy.yaml /root/ansible/nfs_deploy.yaml
controlplane $ cat /root/ansible/nfs_deploy.yaml
- hosts: all
  gather_facts: true
  vars:
  tasks:

  roles:
  - { role: nfs_server, when: type == "server" }
  - { role: nfs_client, when: type == "client" }
  controlplane $ cp /answers/nfs_server_task_main.yaml /root/ansible/roles/nfs_server/tasks/main.yml
controlplane $ cp /answers/nfs_server_handler_main.yaml /root/ansible/roles/nfs_server/handlers/main.yml
controlplane $ cat /root/ansible/roles/nfs_server/tasks/main.yml
---
# tasks file for nfs_server

- name: Deploy the NFS server
  apt:
    name: "nfs-kernel-server"
    state: latest

- name: Create share for nfs server
  file:
    path: "/share"
    state: directory
    owner: nobody
    group: nogroup

- name: Setup share in /etc/exports
  lineinfile:
    path: /etc/exports
    line: '/share *(rw,sync,no_subtree_check)'
    create: yes
  notify:
    - Restart nfs
    controlplane $ echo "----------"
----------
controlplane $ cat /root/ansible/roles/nfs_server/handlers/main.yml
---
# handlers file for nfs_server

- name: Restart nfs
  systemd:
    state: restarted
    name: nfs-server
    controlplane $ ansible-playbook -i /root/hosts /root/ansible/nfs_deploy.yaml

PLAY [all] *********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior 
Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [nfs_server : Deploy the NFS server] **************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

TASK [nfs_server : Create share for nfs server] ********************************************************************************************************************
skipping: [controlplane]
changed: [node01]

TASK [nfs_server : Setup share in /etc/exports] ********************************************************************************************************************
skipping: [controlplane]
changed: [node01]

RUNNING HANDLER [nfs_server : Restart nfs] *************************************************************************************************************************
changed: [node01]

PLAY RECAP *********************************************************************************************************************************************************
controlplane               : ok=1    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
node01                     : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

controlplane $ ssh node01 'hostname; showmount -e'
node01
Export list for node01:
/share *
controlplane $ ansible-playbook -i /root/hosts /root/ansible/nfs_deploy.yaml

PLAY [all] *********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior 
Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [nfs_server : Deploy the NFS server] **************************************************************************************************************************
skipping: [controlplane]
ok: [node01]

TASK [nfs_server : Create share for nfs server] ********************************************************************************************************************
skipping: [controlplane]
ok: [node01]

TASK [nfs_server : Setup share in /etc/exports] ********************************************************************************************************************
skipping: [controlplane]
ok: [node01]

PLAY RECAP *********************************************************************************************************************************************************
controlplane               : ok=1    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
node01                     : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

controlplane $ ssh node01 'hostname; showmount -e'
node01
Export list for node01:
/share *

------------------------------------------------------------------------------
b) Playbook execution deploy client

Lab Activities
You deployed a NFS server and now need to configure the client portion of the playbook to properly install NFS client and then mount the shared directory.
Write or copy over the tasks/main.yml for the role nfs_client and use it to create the client mount points with Ansible.
Verify that the mount point is working properly.


Solution

Copy in the playbook tasks for the NFS client and mount point deployment.
cp /answers/nfs_client_main.yaml /root/ansible/roles/nfs_client/tasks/main.yml

Inspect the files you have created to see what they should do.
cat /root/ansible/roles/nfs_client/tasks/main.yml

Deploy the ansible playbook and see the client deployment.
ansible-playbook -i /root/hosts /root/ansible/nfs_deploy.yaml

Is this the exact same deployment from earlier? Why or why not? Is the command that you're executing exactly the same?

Check that the mount point has been properly mounted on your local system.

mount | grep -i nfs | grep -i share
cat /etc/fstab

Is the filesystem mounted? How does the /etc/fstab entry look to you? Would you have built it that same way?


controlplane $ cp /answers/nfs_client_main.yaml /root/ansible/roles/nfs_client/tasks/main.yml
controlplane $ cat /root/ansible/roles/nfs_client/tasks/main.yml
---
# tasks file for nfs_client

- name: Install the nfs client packages
  apt:
    name: nfs-common
    state: latest

- name: mount the nfs server to local system at /mnt
  mount:
    path: /mnt
    src: 'node01:/share'
    fstype: nfs
    state: mounted

controlplane $ ansible-playbook -i /root/hosts /root/ansible/nfs_deploy.yaml

PLAY [all] *********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior 
Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [nfs_server : Deploy the NFS server] **************************************************************************************************************************
skipping: [controlplane]
ok: [node01]

TASK [nfs_server : Create share for nfs server] ********************************************************************************************************************
skipping: [controlplane]
ok: [node01]

TASK [nfs_server : Setup share in /etc/exports] ********************************************************************************************************************
skipping: [controlplane]
ok: [node01]

TASK [nfs_client : Install the nfs client packages] ****************************************************************************************************************
skipping: [node01]
changed: [controlplane]

TASK [nfs_client : mount the nfs server to local system at /mnt] ***************************************************************************************************
skipping: [node01]
changed: [controlplane]

PLAY RECAP *********************************************************************************************************************************************************
controlplane               : ok=3    changed=2    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
node01                     : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   

controlplane $ cat /root/ansible/nfs_deploy.yaml
- hosts: all
  gather_facts: true
  vars:
  tasks:

  roles:
  - { role: nfs_server, when: type == "server" }
  - { role: nfs_client, when: type == "client" }
  controlplane $ mount | grep -i nfs | grep -i share
node01:/share on /mnt type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=172.30.1.2,local_lock=none,addr=172.30.2.2)
controlplane $ cat /etc/fstab
LABEL=cloudimg-rootfs   /        ext4   defaults        0 1
LABEL=UEFI      /boot/efi       vfat    umask=0077      0 1
node01:/share /mnt nfs defaults 0 0
controlplane $ 


Look at you, learning Ansible! You used Ansible to deploy NFS Server and client via Ansible Playbook. Keep working in this lab to increase functionality of your deployment tool.
```
