```
21 Ansible in the Enterprise

Playbook execution deploy server

Lab Activities
This is the Enterprise Ansible Playground.


Solution
clone the git of HPC_Deploy
root@controlplane:~$ git clone https://github.com/het-tanis/HPC_Deploy.git
Cloning into 'HPC_Deploy'...
remote: Enumerating objects: 152, done.
remote: Counting objects: 100% (152/152), done.
remote: Compressing objects: 100% (89/89), done.
remote: Total 152 (delta 46), reused 119 (delta 19), pack-reused 0 (from 0)
Receiving objects: 100% (152/152), 15.32 KiB | 1.70 MiB/s, done.
Resolving deltas: 100% (46/46), done.

Change into that directory
root@controlplane:~$ cd HPC_Deploy

Run or change the playbooks as you see fit.
root@controlplane:~/HPC_Deploy$ ansible-playbook -i hosts 01_nfs_system.yaml

PLAY [all] ********************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************
ok: [node01]
ok: [controlplane]

TASK [nfs_server : Deploy the NFS server] *************************************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

TASK [nfs_server : Create share for nfs server] *******************************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

TASK [nfs_server : Setup share in /etc/exports] *******************************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

TASK [nfs_client : Install the nfs client packages] ***************************************************************************************************************************************
skipping: [node01]
changed: [controlplane]

TASK [nfs_client : mount the nfs server to local system at /mnt] **************************************************************************************************************************
skipping: [node01]
fatal: [controlplane]: FAILED! => changed=false 
  msg: |-
    Error mounting /mnt: Created symlink /run/systemd/system/remote-fs.target.wants/rpc-statd.service → /usr/lib/systemd/system/rpc-statd.service.
    mount.nfs: access denied by server while mounting node01:/share

RUNNING HANDLER [nfs_server : Restart nfs] ************************************************************************************************************************************************
changed: [node01]

PLAY RECAP ********************************************************************************************************************************************************************************
controlplane               : ok=2    changed=1    unreachable=0    failed=1    skipped=3    rescued=0    ignored=0   
node01                     : ok=5    changed=4    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0  


This breaks the first run, but works the second. This is just a sandbox environment, but feel free to play with why that happens.

PLAY [all] ********************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************
ok: [node01]
ok: [controlplane]

TASK [nfs_server : Deploy the NFS server] *************************************************************************************************************************************************
skipping: [controlplane]
ok: [node01]

TASK [nfs_server : Create share for nfs server] *******************************************************************************************************************************************
skipping: [controlplane]
ok: [node01]

TASK [nfs_server : Setup share in /etc/exports] *******************************************************************************************************************************************
skipping: [controlplane]
ok: [node01]

TASK [nfs_client : Install the nfs client packages] ***************************************************************************************************************************************
skipping: [node01]
ok: [controlplane]

TASK [nfs_client : mount the nfs server to local system at /mnt] **************************************************************************************************************************
skipping: [node01]
changed: [controlplane]

PLAY RECAP ********************************************************************************************************************************************************************************
controlplane               : ok=3    changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
node01                     : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   

Keep working on sandbox practice


Look at you, learning Ansible! This is a sandbox for you to play with Enterprise Ansible.
```
