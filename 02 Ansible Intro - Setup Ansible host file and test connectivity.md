```
02 Ansible Intro

a) Setup Hosts file
Create an Ansible file called /root/hosts with the hosts in your network.

servers:
controlplane
node01


Solution
You may use ini or yaml format for ansible hosts files

Add this to file /root/hosts

ini
[servers]
controlplane
node01
------------------------------------------------------------------------------
b) Check functionality and connectivity
Verify connectivity to all hosts with ping module and save the output to /root/ping_check.


Solution
Executing ping module against both servers

ansible servers -i /root/hosts -m ping
Executing ping module against both servers and output to /root/ping_check

ansible servers -i /root/hosts -m ping > /root/ping_check
------------------------------------------------------------------------------
c) Add to inventory for future environments
Your team is going to be adding new servers to the environment that haven't been built yet. Update your inventory file at /root/hosts to reflect the new environments.
Update the /root/hosts file
use ansible-inventory commands to view the file values.


Solution
Update the /root/hosts file to have a test and prod environment.

vi /root/hosts and then esc : wq to save.

[servers]
controlplane
node01

[test_env]
controltest type=client
node01test  type=server

[prod_env]
controlprod type=client
node01prod  type=server

[non-prod:children]
servers
test_env
Now that you've updated the inventory for the new systems, view the inventory with ansible-inventory to see the host groups.

ansible-inventory -i /root/hosts --list
To see that in graph output use this command.

ansible-inventory -i /root/hosts --graph
To see that in yaml output.

ansible-inventory -i /root/hosts --list -y
Which of those output were most useful to you? Which of them do you think you'd like to see or use for your evironments?


Look at you, learning Ansible! You solved this challenge! You built and inventory file, then used it to connect to servers. You then added additional servers for new environments and viewed them with the ansible-inventory tool.
```
