```
03 Ansible Intro

Ad Hoc to gather data

Lab Activities
Verify your hosts file and test ad hoc commands to your servers
Verify servers uptime
Verify OS type of servers
Verify date and time of servers

Deliverables
Create a file /root/version with the Ansible Facts for the distribution of each server.
Create a file /root/date with the Ansible Facts for the current date of each server.


Solution
cat /root/hosts
Check server uptime

ansible servers -i /root/hosts -m shell -a 'uptime'
Setup module gives so much information you can use during playbook execution.

ansible servers -i /root/hosts -m setup
Cut that output down a bit so you can just check the host distribution information

ansible servers -i /root/hosts -m setup -a 'filter=ansible_distribution'
Send this output to the required file

ansible servers -i /root/hosts -m setup -a 'filter=ansible_distribution' > /root/version
Cut that output down a bit so you can just check the host time information

ansible servers -i /root/hosts -m setup -a 'filter=ansible_date_time'
Send this output to the required file

ansible servers -i /root/hosts -m setup -a 'filter=ansible_date_time' > /root/date


Look at you, learning Ansible! You solved this challenge!
```
