```
04 Ansible Intro

Ad Hoc to copy and edit

Lab Activities
Verify your /root/hosts file and /root/configfile.cfg
Create a directory on your servers for deployments named /opt/deployment
Push over your config file
Change the values of var1 to equal 111111 only in the config file in /opt/deployment.


Solution
cat /root/hosts
cat /root/configfile.cfg
Create a Directory on each server named /opt/deployment

ansible servers -i /root/hosts -m file -a 'path=/opt/deployment state=directory'
Copy over your /root/configfile.cfg to that directory

ansible servers -i /root/hosts -m copy -a 'src=/root/configfile.cfg dest=/opt/deployment'
Let's fix a bad configuration line from 000000 to 111111 with the lineinfile module

ansible servers -i /root/hosts -m lineinfile -a "path=/opt/deployment/configfile.cfg regexp='^var1' line='var1=111111'"
Quick verification that it looks good on all servers

ansible servers -i /root/hosts -m shell -a 'cat /opt/deployment/configfile.cfg'


Look at you, learning Ansible! You solved this challenge!
```
