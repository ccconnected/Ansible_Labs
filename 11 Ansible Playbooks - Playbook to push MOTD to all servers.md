```
11 Ansible Playbooks

Playbook execution to push MOTD to all servers

Lab Activities
Your team has had mistakes because users don't know what server they've jumped into, specifically DEV or PROD environments. You have decided to push a message of the day file (MOTD) to all servers to help them know what systems they are on.
Verify your /root/hosts file and /root/motd_push.yaml file
Push the motd messages to all servers based on the new variables assigned in the /root/hosts file.


Solution
cat /root/hosts
Note: There are variables now assigned to each of the servers (env)

cat /root/motd_push.yaml

Run the Playbook push the MOTD
ansible-playbook -i /root/hosts motd_push.yaml

Run an adhoc command to check all the MOTD on all servers
ansible servers -i /root/hosts -m shell -a 'cat /etc/motd'

Manually check by logging into node01
ssh node01 'cat /etc/motd'

controlplane $ cat /root/hosts
[servers]
controlplane env=prod
node01 env=dev
controlplane $ cat /root/motd_push.yaml
---

- name: MOTD Push
  hosts: servers
  vars:
  gather_facts: True
  become: True
  tasks:

  - name: Debug env variables
    debug:
      var: env

  - name: Push over the file if prod
    copy:
      src: /root/prod_motd
      dest: "/etc/motd"
    when: '"prod" in env'

  - name: Push over the file if dev
    copy:
      src: /root/dev_motd
      dest: "/etc/motd"
    when: '"dev" in env'


controlplane $ cat /etc/motd 
.----------------.  .----------------.  .----------------.  .----------------.
| .--------------. || .--------------. || .--------------. || .--------------. |
| |   ______     | || |  _______     | || |     ____     | || |  ________    | |
| |  |_   __ \   | || | |_   __ \    | || |   .'    `.   | || | |_   ___ `.  | |
| |    | |__) |  | || |   | |__) |   | || |  /  .--.  \  | || |   | |   `. \ | |
| |    |  ___/   | || |   |  __ /    | || |  | |    | |  | || |   | |    | | | |
| |   _| |_      | || |  _| |  \ \_  | || |  \  `--'  /  | || |  _| |___.' / | |
| |  |_____|     | || | |____| |___| | || |   `.____.'   | || | |________.'  | |
| |              | || |              | || |              | || |              | |
| '--------------' || '--------------' || '--------------' || '--------------' |
'----------------'  '----------------'  '----------------'  '----------------'
controlplane $ ssh node01 'cat /etc/motd'
 .----------------.  .----------------.  .----------------.
| .--------------. || .--------------. || .--------------. |
| |  ________    | || |  _________   | || | ____   ____  | |
| | |_   ___ `.  | || | |_   ___  |  | || ||_  _| |_  _| | |
| |   | |   `. \ | || |   | |_  \_|  | || |  \ \   / /   | |
| |   | |    | | | || |   |  _|  _   | || |   \ \ / /    | |
| |  _| |___.' / | || |  _| |___/ |  | || |    \ ' /     | |
| | |________.'  | || | |_________|  | || |     \_/      | |
| |              | || |              | || |              | |
| '--------------' || '--------------' || '--------------' |
 '----------------'  '----------------'  '----------------'


Look at you, learning Ansible! You pushed MOTDs to all servers. You solved this challenge!
```
