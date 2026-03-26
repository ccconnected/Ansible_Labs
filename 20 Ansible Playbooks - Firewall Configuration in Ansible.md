```
20 Ansible Playbooks

a) Create a playbook to push web and database environment.

Lab Activities
Your team has a deployment of a web server and database environment. They want to secure the environment so that only the ports that should be exposed are exposed on the correct servers. You have researched and found that UFW can do this and you can configure it with Ansible in a playbook.

Setup the servers to expose the correct port numbers.
Web Server - Port 22, 80
DB Server - Port 22, 3306

Make sure that you can connect to those servers on the exposed ports.
You may try to complete this activity yourself, or you may find the answers in the Solution below.


Solution
Check your hosts file

root@controlplane:~$ cat /root/hosts
[servers]
controlplane env=prod app=db
node01 env=dev app=web

[webservers]
node01 env=dev app=web

Check on the environment deployer, or compare it to what you tried to write. How do they compare? How do they differ?
root@controlplane:~$ cat /root/packages_install.yaml
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

  - name: Install mariadb on the db server
    apt:
      pkg: 
      - mariadb-server
      - mariadb-client
      state: present
    when: '"db" in app'

Execute the packages_install.yaml to set up your web and db environments.

Run against the [servers] host group
root@controlplane:~$ ansible-playbook -i /root/hosts /root/packages_install.yaml

PLAY [Package install for environment] ****************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************
ok: [controlplane]
ok: [node01]

TASK [Debug env variables just to see them] ***********************************************************************************************************************************************
ok: [controlplane] => {
    "app": "db"
}
ok: [node01] => {
    "app": "web"
}

TASK [Install apache2 on the web server] **************************************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

TASK [Install mariadb on the db server] ***************************************************************************************************************************************************
skipping: [node01]
changed: [controlplane]

PLAY RECAP ********************************************************************************************************************************************************************************
controlplane               : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
node01                     : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

When this executes, do you notice that it skips servers? Why is it doing this, and is it expected?

Verify that you have all the expected ports open on both servers
root@controlplane:~$ timeout 3 nc -vz node01 80
timeout 3 nc -vz node01 22
timeout 3 nc -vz node01 40200
timeout 3 nc -vz controlplane 3306
timeout 3 nc -vz controlplane 22
timeout 3 nc -vz controlplane 40200
Connection to node01 (172.30.2.2) 80 port [tcp/http] succeeded!
Connection to node01 (172.30.2.2) 22 port [tcp/ssh] succeeded!
Connection to node01 (172.30.2.2) 40200 port [tcp/*] succeeded!
Connection to controlplane (127.0.0.1) 3306 port [tcp/mysql] succeeded!
Connection to controlplane (127.0.0.1) 22 port [tcp/ssh] succeeded!
Connection to controlplane (127.0.0.1) 40200 port [tcp/*] succeeded!

If this has all worked, you've completed this part of the lab and are ready to move on.

------------------------------------------------------------------------------
b) Setup the firewall and verify that only enabled ports can be connected to.

Lab Activities
You've been extremely successful. Now we need to secure the environment so that nothing else can connect on any of the other ports that are not allowed.
Use UFW Module in Ansible to secure the two servers as required by your GRC (Governance, Risk, and Compliance) Security Team.

Setup the UFW to only allow the correct port numbers.
Web Server - Port 22, 80
DB Server - Port 22, 3306

You may try to write the playbook yourself, or use the solution below to see the answer and watch it work in your environment.


Solution
Check the file /root/ufw_setup.yaml to see how to setup the firewall for the required exposed ports.
root@controlplane:~$ cat /root/ufw_setup.yaml
---

#Added for InspectorDiameter to remind everyone to inspect the diameter.

- name: UFW Setup
  hosts: servers
  vars:
  gather_facts: True
  become: True
  tasks:

  - name: Debug the app var on each server
    debug:
      var: app
      verbosity: 2

  - name: Enable the UFW firewall
    ufw:
      state: enabled
      policy: deny

  - name: Allow Port 22 (SSH)
    ufw:
      rule: allow
      name: OpenSSH
  
  - name: Allow Port 80, if web server
    ufw:
      rule: allow
      name: Apache
    when: "'web' in app" 

  - name: Allow Port 3306, if db server
    ufw:
      rule: allow
      port: '3306'
      proto: tcp
    when: "'db' in app" 

Inspect the ufw_setup.yaml file. What is the purpose of each of the tasks in the playbook? Which do you expect to run on both servers? Which do you expect to only run on some servers? Why?

Run the file to see what happens and verify operations.
root@controlplane:~$ ansible-playbook -i /root/hosts /root/ufw_setup.yaml

PLAY [UFW Setup] **************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************
ok: [controlplane]
ok: [node01]

TASK [Debug the app var on each server] ***************************************************************************************************************************************************
skipping: [controlplane]
skipping: [node01]

TASK [Enable the UFW firewall] ************************************************************************************************************************************************************
changed: [controlplane]
changed: [node01]

TASK [Allow Port 22 (SSH)] ****************************************************************************************************************************************************************
changed: [controlplane]
changed: [node01]

TASK [Allow Port 80, if web server] *******************************************************************************************************************************************************
skipping: [controlplane]
changed: [node01]

TASK [Allow Port 3306, if db server] ******************************************************************************************************************************************************
skipping: [node01]
changed: [controlplane]

PLAY RECAP ********************************************************************************************************************************************************************************
controlplane               : ok=4    changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
node01                     : ok=4    changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   


Now you can test and see that only the ports that you expect to be available are able to be connected to.

Verify the configurations of the firewalls are different:
root@controlplane:~$ ufw status
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
3306/tcp                   ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
3306/tcp (v6)              ALLOW       Anywhere (v6)             

root@controlplane:~$ ssh node01 'ufw status'
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Apache                     ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Apache (v6)                ALLOW       Anywhere (v6)             

Are they configured like you would expect? Why or why not?

root@controlplane:~$ timeout 3 nc -vz node01 80
Connection to node01 (172.30.2.2) 80 port [tcp/http] succeeded!
root@controlplane:~$ timeout 3 nc -vz node01 22
Connection to node01 (172.30.2.2) 22 port [tcp/ssh] succeeded!
root@controlplane:~$ timeout 3 nc -vz controlplane 3306
Connection to controlplane (127.0.0.1) 3306 port [tcp/mysql] succeeded!
root@controlplane:~$ ssh node01 'timeout 3 nc -vz controlplane 22'
Connection to controlplane (172.30.1.2) 22 port [tcp/ssh] succeeded!

The above should work, the items below should time out.

root@controlplane:~$ timeout 3 nc -vz node01 40200
root@controlplane:~$ 

root@controlplane:~$ ssh node01 'timeout 3 nc -vz controlplane 40200'
root@controlplane:~$ 

Did you see the timeouts on the ports that you expect to? Did this deployer act as you expected? What could you do to improve this operation?


Look at you, learning Ansible! You created a playbook to setup a host based firewall on multiple systems. You were able to configure different ports based on the applications on the servers.
```
