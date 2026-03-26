```
07 Ansible Playbooks

Create a playbook to deploy a jinja2 template

Lab Activities
Verify your hosts file
Create a Jinja2 template file called /root/template.j2 to stamp values on each of the hosts as you execute the playbook.
Create a deploy playbook called /root/template.yml to push the template /root/template.j2 over to all servers as the file /root/template.txt. Verify the file is populated with values as expected.

Tip
If you get stuck, the answer file is found in /answers/template.yml
cp /answers/template.yml /root/template.yml
cp /answers/template.j2 /root/template.j2


Solution
cat /root/hosts

Yaml for playbook
---
- name: Start of Jinja2 Template Push
  hosts: servers
  vars:
  gather_facts: True
  become: False
  tasks:
    - name: Copy template over to all hosts
      template:
        src: /root/template.j2
        dest: "/root/template.txt"

Run Playbook and verify that everything pushed correctly
ansible-playbook -i /root/hosts /root/template.yml

Manual verify for all
ansible servers -i /root/hosts -m shell -a 'cat /root/template.txt'

controlplane $ cat /root/template.j2
This is an auto generated file by ansible at {{ ansible_date_time.date }} {{ ansible_date_time.time }}.
Hostname: {{ ansible_nodename }}
System: {{ ansible_os_family }}
Proc: {{ ansible_processor_count }}


Look at you, learning Ansible templates! You solved this challenge!
```
