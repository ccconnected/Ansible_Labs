```
10 Ansible Playbooks

a) Playbook execution to see extra-vars

Lab Activities
Your team has decided they need to more securely keep their secret information, instead of just passing values at the command line. You know that Ansible Vault can be used to securely store variables.
Verify your /root/hosts file and /root/variables.yaml file
Verify the functionality of the /root/variables.yaml file by executing and giving extra-vars as needed.


Solution
cat /root/hosts
cat /root/variables.yaml
---

- name: Variable output testing
  hosts: localhost
  vars:
  gather_facts: True
  become: False
  tasks:

    - name: Debug variables to view contents
      debug:
        msg: "{{ item }} is in variable list"
      with_items:
        - "{{ username }}"
        - "{{ password }}"
Run the Playbook to see the output (This should break)
ansible-playbook variables.yaml

Run the Playbook again and give the correct variables to see it exectue correctly
ansible-playbook variables.yaml --extra-vars "username=scott password=secure"

------------------------------------------------------------------------------
b) Create Ansible Vault

Lab Activities
Create your ansible vault


Solution
Create a vault using Ansible. You will be asked to give a password, make sure you remember it.
ansible-vault create vault.yaml

Add the following information:
username: user1
password: somestrongpassword

Verify that you cannot read the file.
cat vault.yaml

View the vault with ansible-vault command. You will have to add in the password
ansible-vault view vault.yaml

------------------------------------------------------------------------------
c) Run playbook to consume Ansible Vault variables

Lab Activities

Run the playbook vault_variables.yaml to see ansible run and use the vault variables.


Solution
Run the Playbook vault_variables.yaml. (You will have to put in your vault password)
ansible-playbook --ask-vault-pass vault_variables.yaml

If we were using tower, we'd store credentials securely in there and it wouldn't matter. But, as it is we have an option to keep a password file. We can use that to decrypt the vault.
echo <your password> > .passfile

Make sure it's readable only by your user.
chmod 400 .passfile

Run the ansible playbook and consume that password file to decrypt the vault
ansible-playbook --vault-password-file=.passfile vault_variables.yaml

ubuntu $ cat vault_variables.yaml
---

- name: Variable output testing
  hosts: localhost
  vars:
  vars_files:
    - vault.yaml
  gather_facts: True
  become: False
  tasks:

    - name: "Debug variables to view contents"
      debug:
        msg: "{{ item }} is in variable list"
      with_items:
        - "{{ username }}"
        - "{{ password }}"

ubuntu $ cat vault.yaml 
$ANSIBLE_VAULT;1.1;AES256
39633963333032613033303964653535613165333961343763373733353464373134313462366365
6534376361666133613931373930636462333233653733330a393561653330643866356461663939
39643039333564616437326330386434313638633334663363346634656330633732323162663030
3630636432653030380a636139363964653636326264633230343838386434623730626130653634
62613366373935303762396639616633626138643435366137636437353366393764316532303266
6161316562343963535303636616133346366336163366364 

ubuntu $ ansible-playbook vault_variables.yaml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
ERROR! Attempting to decrypt but no vault secrets found


Look at you, learning Ansible! You vaulted information and decrypted it to read variables. You solved this challenge!
```
