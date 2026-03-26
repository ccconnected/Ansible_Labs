```
01 Ansible Intro

a) Install Ansible
Your team has decided they need a tool they can use to control and operate the many servers you have in your environment. You have been tasked with evaluating Ansible.

Install Ansible and put the version output in a file called /root/version.


Solution
Install Ansible

apt -y install ansible
Check the version

ansible --version
Check version and send to file

ansible --version >> /root/version
------------------------------------------------------------------------------
b) Check modules and syntax
Check ansible-doc and see how many modules you have. Then read the documentation of modules setup and copy.

Save the number of modules you have into the file /root/modules


Solution
Check ansible-doc help

ansible-doc --help
Check all the modules and store them in a file /root/modules

ansible-doc -l | wc -l
ansible-doc -l | wc -l > /root/modules
Read the documentation on setup module (may have to hit q to exit)

ansible-doc -s setup
Read the documentation on copy module (may have to hit q to exit)

ansible-doc -s copy

```
