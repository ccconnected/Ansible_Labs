```
09 Ansible Playbooks

Playbook execution to see API call with URI module

Lab Activities
Verify your /root/hosts file and /root/api.yaml
Create a playbook that calls from https://swapi.dev/api/people/ via the uri method.
Return JSON output and debug to see values returned by various calls.


Solution
cat /root/hosts
cat /root/api.yaml
---

- name: Call API and get results from a webpage
  hosts: localhost
  vars:
  gather_facts: True
  become: False
  tasks:

    - name: Pulling SWAPI and get results from the webpage
      uri:
        url: https://swapi.dev/api/people/
        return_content: yes
      register: swapi

    - name: Debug variables to view contents
      debug:
        var: swapi

    #- name: Debug variables to view contents
    #  debug:
    #    var: swapi.json.results[0]

Run the Playbook to see the output
ansible-playbook api.yaml

Edit the playbook to give you just the information on the first entry. Uncomment these lines

    #- name: Debug variables to view contents
    #  debug:
    #    var: swapi.json.results[0]

Rerun the Playbook to see the new output
ansible-playbook api.yaml


Look at you, learning Ansible! You pulled information from an API and looked at the JSON objects returned with debug. You solved this challenge!
```
