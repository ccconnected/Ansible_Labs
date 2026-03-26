```
08 Ansible Playbooks

Create a playbook to deploy a jinja2 report

Lab Activities
Verify your hosts file
Create a deploy playbook called /root/template.yml to check the uptime of all servers.
Create a Jinja2 template file to deploy the /root/report.txt report upon execution of the playbook.
Report back unreachable servers and servers that have been up longer than a day to /root/report.txt on the server of execution.

Tip
If you get stuck, the answer file is found in /answers/template.yml
cp /answers/template.yml /root/template.yml
cp /answers/template.j2 /root/template.j2

controlplane $ cat /root/template.j2 
This is a system Validation at {{ ansible_date_time.time }} on {{ ansible_date_time.date }}

Unreachable systems:
----------------------------------------------
{% for host in ansible_play_hosts_all %}
{% if host not in ansible_play_hosts %}
{{ host }}
{% endif  %}
{% endfor %}

Hosts who have been up longer than a day:
----------------------------------------------
{% for host in ansible_play_hosts_all %}
{% if hostvars[host].uptime is defined %}
{% if 'day' in hostvars[host].uptime.stdout %}
 {{ hostvars[host].ansible_hostname }} - has not rebooted today
{% endif %}
{% endif %}
{% endfor %}


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

    - name: Get information for uptime on all systems
      shell: uptime
      register: uptime
      
    - name: Copy template over to all hosts
      template:
        src: /root/template.j2
        dest: "/root/report.{{ansible_date_time.iso8601_basic_short}}.txt"
      run_once: yes
      delegate_to: localhost

Run Playbook and verify that everything pushed correctly
ansible-playbook -i /root/hosts /root/template.yml

Manual verify for all
cat /root/report.*.txt


Look at you, learning Ansible templates! You solved this challenge!
```
