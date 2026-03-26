```
16 Ansible Playbooks

a) Create a playbook to push apache environment

Lab Activities
Your team was so happy with your work on Deploying a Web Server https://killercoda.com/het-tanis/course/Linux-Labs/101-configure-apache-server that 10 other teams have come and asked for you to do the same deployment for them.
Make a webserver available on node01
Set up a directive and web page that identifies each of the following web pages:
dev - port 8080
test - port 8081
qa - port 8082
Make sure each html page exists and has the name dev, test, and qa on the page. These will be provided by the individual teams and put in the /answers directory


Solution
Check your hosts file

cat /root/hosts

Check your virtual hosts and index files. Modify this command for all the files in that directory.

ls /answers/
cat /answers/dev_index.html
echo ""
cat /answers/dev_virtual_host.conf

Create each component of the /root/web_environment.yaml file. (If you get stuck you can find this file in /answers/web_environment.yaml)

Run against the [webservers] host group

---

- name: Web Environment
  hosts: webservers
  vars:
  gather_facts: True
  become: True
  tasks:

Create the section to install apache2
  - name: Install Apache2 Server
    apt:
      name: "apache2"
      state: latest

Create the directories for each environmentn
  - name: Create directories for environments
    file:
      path: "/var/www/html_{{item}}"
      state: directory
    loop:
    - dev
    - test
    - qa
    notify:
      - Restart apache

Add the listener ports to the correct configuration file
  - name: Add the Listener ports to /etc/apache2/ports.conf
    lineinfile:
      path: /etc/apache2/ports.conf
      insertafter: '^Listen'
      state: present
      line: "{{item}}"
    loop:
    - 'Listen 8080'
    - 'Listen 8081'
    - 'Listen 8082'
    notify:
      - Restart apache

Create the virtual directives for each web page
  - name: Push the Virtual Directives files into the correct place
    copy:
      src: "/answers/{{item}}"
      dest: "/etc/apache2/sites-enabled/{{item}}"
    loop:
    - dev_virtual_host.conf
    - qa_virtual_host.conf
    - test_virtual_host.conf
    notify:
      - Restart apache

Push the html files that the teams have given you into the right directories.
  - name: Push the html for each page over
    copy:
      src: "/answers/{{item.name}}"
      dest: "/var/www/html_{{item.env}}/index.html"
    loop:
    - { env: 'dev', name: 'dev_index.html'}
    - { env: 'test', name: 'test_index.html'}
    - { env: 'qa', name: 'qa_index.html'}
    notify:
      - Restart apache

In each of those blocks above you set a notification if something is changed. Now you have to create the handler that gets used in the event a notification happens.

  handlers:

  - name: Restart apache
    systemd:
      state: restarted
      name: apache2

If you need to copy the deployment file from the answers, use this.
cp /answers/web_environment.yaml /root/web_environment.yaml

Run your completed playbook to deploy all environments
ansible-playbook -i /root/hosts /root/web_environment.yaml

Run it a second time to see all the events that no longer have to happen. Did the handler run the second time? Why or why not?

Verify that each of the listeners are from the correct environments.

curl node01:8080
curl node01:8081
curl node01:8082

If this has all worked, you've completed this lab and are ready to move on.


controlplane $ cat /root/hosts
[servers]
controlplane env=prod app=db
node01 env=dev app=web

[webservers]
node01 env=dev app=web

controlplane $ ls /answers/
dev_index.html  
dev_virtual_host.conf  
individual_web_environment.yaml  
qa_index.html  
qa_virtual_host.conf  
test_index.html  
test_virtual_host.conf  
web_environment.yaml

controlplane $ cat /answers/dev_index.html
<html>
<head><title>Dev Page</title><head>
<body>Dev Environment</body>
</html>controlplane 

$ echo ""

controlplane $ cat /answers/dev_virtual_host.conf
<VirtualHost *:8080>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html_dev

        ErrorLog ${APACHE_LOG_DIR}/dev_error.log
        CustomLog ${APACHE_LOG_DIR}/dev_access.log combined

</VirtualHost>
controlplane $ cp /answers/web_environment.yaml /root/web_environment.yaml
controlplane $ cat /root/web_environment.yaml
---

- name: Web Environment
  hosts: webservers
  vars:
  gather_facts: True
  become: True
  tasks:

  - name: Install Apache2 Server
    apt:
      name: "apache2"
      state: latest
    
  - name: Create directories for environments
    file:
      path: "/var/www/html_{{item}}"
      state: directory
    loop:
    - dev
    - test
    - qa
    notify:
      - Restart apache
  
  - name: Add the Listener ports to /etc/apache2/ports.conf
    lineinfile:
      path: /etc/apache2/ports.conf
      insertafter: '^Listen'
      state: present
      line: "{{item}}"
    loop:
    - 'Listen 8080'
    - 'Listen 8081'
    - 'Listen 8082'
    notify:
      - Restart apache

  - name: Push the Virtual Directives files into the correct place
    copy:
      src: "/answers/{{item}}"
      dest: "/etc/apache2/sites-enabled/{{item}}"
    loop:
    - dev_virtual_host.conf
    - qa_virtual_host.conf
    - test_virtual_host.conf
    notify:
      - Restart apache

  - name: Push the html for each page over
    copy:
      src: "/answers/{{item.name}}"
      dest: "/var/www/html_{{item.env}}/index.html"
    loop:
    - { env: 'dev', name: 'dev_index.html'}
    - { env: 'test', name: 'test_index.html'}
    - { env: 'qa', name: 'qa_index.html'}
    notify:
      - Restart apache

  handlers:

  - name: Restart apache
    systemd:
      state: restarted
      name: apache2
  
  
controlplane $ ansible-playbook -i /root/hosts /root/web_environment.yaml

PLAY [Web Environment] ***************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]

TASK [Install Apache2 Server] ********************************************************************************************************************************************************************************************
changed: [node01]

TASK [Create directories for environments] *******************************************************************************************************************************************************************************
changed: [node01] => (item=dev)
changed: [node01] => (item=test)
changed: [node01] => (item=qa)

TASK [Add the Listener ports to /etc/apache2/ports.conf] *****************************************************************************************************************************************************************
changed: [node01] => (item=Listen 8080)
changed: [node01] => (item=Listen 8081)
changed: [node01] => (item=Listen 8082)

TASK [Push the Virtual Directives files into the correct place] **********************************************************************************************************************************************************
changed: [node01] => (item=dev_virtual_host.conf)
changed: [node01] => (item=qa_virtual_host.conf)
changed: [node01] => (item=test_virtual_host.conf)

TASK [Push the html for each page over] **********************************************************************************************************************************************************************************
changed: [node01] => (item={'env': 'dev', 'name': 'dev_index.html'})
changed: [node01] => (item={'env': 'test', 'name': 'test_index.html'})
changed: [node01] => (item={'env': 'qa', 'name': 'qa_index.html'})

RUNNING HANDLER [Restart apache] *****************************************************************************************************************************************************************************************
changed: [node01]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
node01                     : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

controlplane $ ansible-playbook -i /root/hosts /root/web_environment.yaml

PLAY [Web Environment] ***************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]

TASK [Install Apache2 Server] ********************************************************************************************************************************************************************************************
ok: [node01]

TASK [Create directories for environments] *******************************************************************************************************************************************************************************
ok: [node01] => (item=dev)
ok: [node01] => (item=test)
ok: [node01] => (item=qa)

TASK [Add the Listener ports to /etc/apache2/ports.conf] *****************************************************************************************************************************************************************
ok: [node01] => (item=Listen 8080)
ok: [node01] => (item=Listen 8081)
ok: [node01] => (item=Listen 8082)

TASK [Push the Virtual Directives files into the correct place] **********************************************************************************************************************************************************
ok: [node01] => (item=dev_virtual_host.conf)
ok: [node01] => (item=qa_virtual_host.conf)
ok: [node01] => (item=test_virtual_host.conf)

TASK [Push the html for each page over] **********************************************************************************************************************************************************************************
ok: [node01] => (item={'env': 'dev', 'name': 'dev_index.html'})
ok: [node01] => (item={'env': 'test', 'name': 'test_index.html'})
ok: [node01] => (item={'env': 'qa', 'name': 'qa_index.html'})

PLAY RECAP ***************************************************************************************************************************************************************************************************************
node01                     : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

controlplane $ curl node01:8080
<html>
<head><title>Dev Page</title><head>
<body>Dev Environment</body>

</html>controlplane $ curl node01:8081
<html>
<head><title>Test Page</title><head>
<body>Test Environment</body>

</html>controlplane $ curl node01:8082
<html>
<head><title>QA Page</title><head>
<body>QA Environment</body>

------------------------------------------------------------------------------
b) Remove environment and push only based on tags

Lab Activities
You've been extremely successful. Now the teams want the ability to turn off or re-deploy only parts of the environments as needed. They want each piece to exist independently of each other.
Break out the playbook and turn off the listener for test on port 8081.


Solution
There's no answers here. You have to do this one on your own, you can do it manually but know that you're going to be doing that a lot for all those developer teams.

Ok, there's an answer found at /answers/individual_web_environment.yaml.

Copy that file over

cp /answers/individual_web_environment.yaml /root/individual_web_environment.yaml
Inspect that file and see how it is different than the other file. What is the use of block and when conditions doing in this file?

cat /root/individual_web_environment.yaml
What variables must you provide at the deployment of the file?

ansible-playbook -i /root/hosts /root/individual_web_environment.yaml --extra-vars "deploy=absent env=test port=8081"
Play with these and see what variables you can and cannot use. To finish the lab ensure that test port 8081 is off, per the requirement.


controlplane $ cp /answers/individual_web_environment.yaml /root/individual_web_environment.yaml
controlplane $ cat /root/individual_web_environment.yaml
---

#Required variables:
#deploy = present or absent
#env = dev test or qa
#port = 8080 8081 or 8082 (must match defined spec given by teams)

- name: Web Environment
  hosts: webservers
  vars:
  gather_facts: True
  become: True
  tasks:

  - name: Install Apache2 Server
    apt:
      name: "apache2"
      state: latest
    
  - name: Block for Present deploy State
    block:

    - name: Create directories for environments
      file:
        path: "/var/www/html_{{env}}"
        state: directory
      notify:
        - Restart apache
    
    - name: Add the Listener ports to /etc/apache2/ports.conf
      lineinfile:
        path: /etc/apache2/ports.conf
        insertafter: '^Listen'
        state: present
        line: "Listen {{port}}"
      notify:
        - Restart apache

    - name: Push the Virtual Directives files into the correct place
      copy:
        src: "/answers/{{env}}_virtual_host.conf"
        dest: "/etc/apache2/sites-enabled/{{env}}_virtual_host.conf"
      notify:
        - Restart apache

    - name: Push the html for each page over
      copy:
        src: "/answers/{{env}}_index.html"
        dest: "/var/www/html_{{env}}/index.html"
      notify:
        - Restart apache

    when: deploy == "present"

  - name: Block for Absent deploy State
    block:

    - name: Delete directories for environments
      file:
        path: "/var/www/html_{{env}}"
        state: absent     
      notify:
        - Restart apache
    
    - name: Remove the Listener ports to /etc/apache2/ports.conf
      lineinfile:
        path: /etc/apache2/ports.conf
        insertafter: '^Listen'
        state: absent
        line: "Listen {{port}}"
      loop:
      notify:
        - Restart apache

    - name: Remove the Virtual Directives files into the correct place
      file:
        path: "/answers/{{env}}_virtual_host.conf"
        state: absent
      notify:
        - Restart apache

    when: deploy == "absent"

  handlers:
  - name: Restart apache
    systemd:
      state: restarted
      name: apache2
  
  
controlplane $ ansible-playbook -i /root/hosts /root/individual_web_environment.yaml --extra-vars "deploy=absent env=test port=8081"
[WARNING]: Found variable using reserved name: port

PLAY [Web Environment] ***************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default 
to using the discovered platform python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]

TASK [Install Apache2 Server] ********************************************************************************************************************************************************************************************
ok: [node01]

TASK [Create directories for environments] *******************************************************************************************************************************************************************************
skipping: [node01]

TASK [Add the Listener ports to /etc/apache2/ports.conf] *****************************************************************************************************************************************************************
skipping: [node01]

TASK [Push the Virtual Directives files into the correct place] **********************************************************************************************************************************************************
skipping: [node01]

TASK [Push the html for each page over] **********************************************************************************************************************************************************************************
skipping: [node01]

TASK [Delete directories for environments] *******************************************************************************************************************************************************************************
changed: [node01]

TASK [Remove the Listener ports to /etc/apache2/ports.conf] **************************************************************************************************************************************************************
changed: [node01]

TASK [Remove the Virtual Directives files into the correct place] ********************************************************************************************************************************************************
ok: [node01]

RUNNING HANDLER [Restart apache] *****************************************************************************************************************************************************************************************
changed: [node01]

PLAY RECAP ***************************************************************************************************************************************************************************************************************
node01                     : ok=6    changed=3    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   

controlplane $ curl node01:8080
<html>
<head><title>Dev Page</title><head>
<body>Dev Environment</body>
controlplane $ curl node01:8081
curl: (7) Failed to connect to node01 port 8081: Connection refused
controlplane $ curl node01:8082
<html>
<head><title>QA Page</title><head>
<body>QA Environment</body>
</html>controlplane $ 


Look at you, learning Ansible! You created a playbook to deploy a web server for your development teams. You then created a solution that allowed them to deploy or remove only the environments they wanted!
```
