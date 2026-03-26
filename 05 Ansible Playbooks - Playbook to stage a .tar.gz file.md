```
05 Ansible Playbooks

Create a playbook to stage an application

Lab Activities
Verify your hosts file
Create a deploy playbook called /root/deploy.yml to push the file /root/deploy.tar.gz over to all servers in the /opt directory.

Tip
If you get stuck, the answer file is found in /answers/deploy.yml
cp /answers/deploy.yml /root/deploy.yml


Solution
cat /root/hosts

Get sha1 checksum of the archive.
sha1sum /root/deploy.tar.gz

Yaml for playbook
---
- name: Start of Deployer playbook
  hosts: servers
  vars:
  gather_facts: True
  become: False
  tasks:
    - name: Copy deploy.tar.gz over at {{ ansible_date_time.iso8601_basic_short }}
      copy:
        src: /root/deploy.tar.gz
        dest: /opt/deploy.tar.gz
        checksum: c6cd21b75a4b300b9228498c78afc6e7a831839e

Run Playbook and verify that everything pushed correctly
ansible-playbook -i /root/hosts /root/deploy.yml

Manual verify for all
ansible servers -i /root/hosts -m shell -a 'ls -l /opt/deploy.tar.gz'


Look at you, learning Ansible! You solved this challenge!
```
