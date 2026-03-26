```
19 Ansible Playbooks

a) Create roles and templates
Management has requested that you provide system information about all of the systems in your HPC cluster and give it to them in a format they can digest via Excel. You have decided to build an Ansible playbook that can report out in .csv format and make them happy.

Lab Activities
Verify your /root/hosts file exists.
Create a directory called /root/playbooks/roles
Create one galaxy inside of /root/playbooks/roles called data_gather .
Update the data_gather roles to properly do it's function, which can be referenced from /answers/ files.


Solution

cat /root/hosts

make the directory structure as required.

mkdir -p /root/playbooks/roles
mkdir -p /root/playbooks/reports
cd /root/playbooks/roles

Note: You're now moved into that directory and can create the required role using ansible-galaxy command

ls -l
tree
ansible-galaxy init data_gather

and check

ls -l
tree

Copy the data_gather.yaml from /answers into data_gather.yaml so that it executes correctly.
cp /answers/data_gather.yaml /root/playbooks/data_gather.yaml

Now that you've done that examine the playbook to see what is going to happen and how the roles are inherited.
cat /root/playbooks/data_gather.yaml

Copy the main.yml from /answers into main.yml so that it executes correctly.
cp /answers/main.yml /root/playbooks/roles/data_gather/tasks/main.yml

Now that you've done that examine the playbook to see what is going to happen.
cat /root/playbooks/roles/data_gather/tasks/main.yml

Why do we have to run two different shell commands?
What is the iso8601_basic_short value going to do when ansible_date_time is called?

Copy the Jinja2 template from /answers into data_gather.j2 so that it can be reported out.
cp /answers/data_gather.j2 /root/playbooks/roles/data_gather/templates/data_gather.j2

Now that you've done that examine the Jinja2 to see what is going to be reported out
cat /root/playbooks/roles/data_gather/templates/data_gather.j2

You are now ready to move on to the next part of the lab


controlplane $ cat /root/hosts
[servers]
controlplane env=prod app=db
node01 env=dev app=web
controlplane $ mkdir -p /root/playbooks/roles
controlplane $ mkdir -p /root/playbooks/reports
controlplane $ cd /root/playbooks/roles
controlplane $ ls -l
total 0
controlplane $ tree
.

0 directories, 0 files
controlplane $ ansible-galaxy init data_gather
- Role data_gather was created successfully
controlplane $ tree
.
`-- data_gather
    |-- README.md
    |-- defaults
    |   `-- main.yml
    |-- files
    |-- handlers
    |   `-- main.yml
    |-- meta
    |   `-- main.yml
    |-- tasks
    |   `-- main.yml
    |-- templates
    |-- tests
    |   |-- inventory
    |   `-- test.yml
    `-- vars
        `-- main.yml

9 directories, 8 files
controlplane $ cp /answers/data_gather.yaml /root/playbooks/data_gather.yaml
controlplane $ cat /root/playbooks/data_gather.yaml
---

- name: Data Gather Playbook
  hosts: servers
  vars:
  gather_facts: True
  become: True
  tasks:

  roles:
  - data_gather


controlplane $ cp /answers/main.yml /root/playbooks/roles/data_gather/tasks/main.yml
controlplane $ cat /root/playbooks/roles/data_gather/tasks/main.yml
---
# tasks file for data_gather
# What are we doing?
# Gather a bunch of data about the system and
# show it via a CSV format
# Server Name: Facts
# Server Make: Facts
# Server Model: Facts
# Proc Type: Command
# Socket #: Command

- name: gather processor type
  shell: "lscpu | grep 'Model name' | cut -d: -f2 | awk -F, '{print $1}' | awk '{$1=$1};1'"
  register: proc_type

- name: gather socket number
  shell: "lscpu | grep 'Socket' | cut -d: -f2 | awk '{$1=$1};1'"
  register: socket

- name: Debug everything
  debug:
    var: "{{ item }}"
    verbosity: 2
  loop:
    - ansible_hostname
    - ansible_system_vendor
    - ansible_product_name
    - proc_type.stdout
    - socket.stdout

- name: Deploy the Template
  template:
    src: /root/playbooks/roles/data_gather/templates/data_gather.j2
    dest: /root/playbooks/reports/report.{{ansible_date_time.iso8601_basic_short}}.csv
  run_once: true
  delegate_to: localhostcontrolplane $ cp /answers/data_gather.j2 /root/playbooks/roles/data_gather/templates/data_gather.j2
controlplane $ cat /root/playbooks/roles/data_gather/templates/data_gather.j2
Date,Node Name,Server Make,Server Model,Processor Type and Model,Socket #
{% for host in ansible_play_hosts_all %}
{% if hostvars[host].socket is defined %}
{{ansible_date_time.date}} {{ansible_date_time.time}},{{hostvars[host].ansible_hostname}},{{hostvars[host].ansible_system_vendor}},{{hostvars[host].ansible_product_name}},{{hostvars[host].proc_type.stdout}},{{hostvars[host].socket.stdout}}
{% endif %}
{% endfor %}

------------------------------------------------------------------------------
b) Playbook execution that gathers CSV Data

Lab Activities
So now that we have the role and playbook correct, let's execute and see it work.
Test the playbook data_gather .


Solution

Run the Playbook to see it work.
ansible-playbook -i /root/hosts /root/playbooks/data_gather.yaml

When runs, does it show the debug output? How do you know?

Check the output of the report.

Is it a properly formatted .csv? How can you tell?

ls -l /root/playbooks/reports/*.csv
cat /root/playbooks/reports/*.csv

Run it again as below, and see if you get the debug output.
ansible-playbook -vv -i /root/hosts /root/playbooks/data_gather.yaml

Did you see the debug output of all the variables? What caused this?
Check the output of the reports again to see that there are now more than one.

ls -l /root/playbooks/reports/*.csv
cat /root/playbooks/reports/*.csv


controlplane $ ansible-playbook -i /root/hosts /root/playbooks/data_gather.yaml

PLAY [Data Gather Playbook] ****************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior 
Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]

TASK [data_gather : gather processor type] *************************************************************************************************************************
changed: [node01]
changed: [controlplane]

TASK [data_gather : gather socket number] **************************************************************************************************************************
changed: [node01]
changed: [controlplane]

TASK [data_gather : Debug everything] ******************************************************************************************************************************
skipping: [controlplane] => (item=ansible_hostname) 
skipping: [node01] => (item=ansible_hostname) 
skipping: [controlplane] => (item=ansible_system_vendor) 
skipping: [node01] => (item=ansible_system_vendor) 
skipping: [controlplane] => (item=ansible_product_name) 
skipping: [node01] => (item=ansible_product_name) 
skipping: [controlplane] => (item=proc_type.stdout) 
skipping: [node01] => (item=proc_type.stdout) 
skipping: [node01] => (item=socket.stdout) 
skipping: [node01]
skipping: [controlplane] => (item=socket.stdout) 
skipping: [controlplane]

TASK [data_gather : Deploy the Template] ***************************************************************************************************************************
changed: [controlplane -> localhost]

PLAY RECAP *********************************************************************************************************************************************************
controlplane               : ok=4    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
node01                     : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

controlplane $ ls -l /root/playbooks/reports/*.csv
-rw-r--r-- 1 root root 230 Jan 18 12:26 /root/playbooks/reports/report.20240118T122613.csv
controlplane $ cat /root/playbooks/reports/*.csv
Date,Node Name,Server Make,Server Model,Processor Type and Model,Socket #
2024-01-18 12:26:13,controlplane,KubeVirt,None,Intel Xeon E312xx (Sandy Bridge,1
2024-01-18 12:26:13,node01,KubeVirt,None,Intel Xeon E312xx (Sandy Bridge,1
controlplane $ ansible-playbook -vv -i /root/hosts /root/playbooks/data_gather.yaml
ansible-playbook 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 3.8.10 (default, Nov 22 2023, 10:22:35) [GCC 9.4.0]
Using /etc/ansible/ansible.cfg as config file

PLAYBOOK: data_gather.yaml *****************************************************************************************************************************************
1 plays in /root/playbooks/data_gather.yaml

PLAY [Data Gather Playbook] ****************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************
task path: /root/playbooks/data_gather.yaml:3
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host node01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior 
Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [node01]
[DEPRECATION WARNING]: Distribution ubuntu 20.04 on host controlplane should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [controlplane]
META: ran handlers

TASK [data_gather : gather processor type] *************************************************************************************************************************
task path: /root/playbooks/roles/data_gather/tasks/main.yml:12
changed: [node01] => {"changed": true, "cmd": "lscpu | grep 'Model name' | cut -d: -f2 | awk -F, '{print $1}' | awk '{$1=$1};1'", "delta": "0:00:00.007683", "end": "2024-01-18 12:28:53.647153", "rc": 0, "start": "2024-01-18 12:28:53.639470", "stderr": "", "stderr_lines": [], "stdout": "Intel Xeon E312xx (Sandy Bridge", "stdout_lines": ["Intel Xeon E312xx (Sandy Bridge"]}
changed: [controlplane] => {"changed": true, "cmd": "lscpu | grep 'Model name' | cut -d: -f2 | awk -F, '{print $1}' | awk '{$1=$1};1'", "delta": "0:00:00.006711", "end": "2024-01-18 12:28:53.713741", "rc": 0, "start": "2024-01-18 12:28:53.707030", "stderr": "", "stderr_lines": [], "stdout": "Intel Xeon E312xx (Sandy Bridge", "stdout_lines": ["Intel Xeon E312xx (Sandy Bridge"]}

TASK [data_gather : gather socket number] **************************************************************************************************************************
task path: /root/playbooks/roles/data_gather/tasks/main.yml:16
changed: [node01] => {"changed": true, "cmd": "lscpu | grep 'Socket' | cut -d: -f2 | awk '{$1=$1};1'", "delta": "0:00:00.004889", "end": "2024-01-18 12:28:53.972720", "rc": 0, "start": "2024-01-18 12:28:53.967831", "stderr": "", "stderr_lines": [], "stdout": "1", "stdout_lines": ["1"]}
changed: [controlplane] => {"changed": true, "cmd": "lscpu | grep 'Socket' | cut -d: -f2 | awk '{$1=$1};1'", "delta": "0:00:00.014618", "end": "2024-01-18 12:28:54.018808", "rc": 0, "start": "2024-01-18 12:28:54.004190", "stderr": "", "stderr_lines": [], "stdout": "1", "stdout_lines": ["1"]}

TASK [data_gather : Debug everything] ******************************************************************************************************************************
task path: /root/playbooks/roles/data_gather/tasks/main.yml:20
ok: [controlplane] => (item=ansible_hostname) => {
    "ansible_hostname": "controlplane",
    "ansible_loop_var": "item",
    "item": "ansible_hostname"
}
ok: [controlplane] => (item=ansible_system_vendor) => {
    "ansible_loop_var": "item",
    "ansible_system_vendor": "KubeVirt",
    "item": "ansible_system_vendor"
}
ok: [controlplane] => (item=ansible_product_name) => {
    "ansible_loop_var": "item",
    "ansible_product_name": "None",
    "item": "ansible_product_name"
}
ok: [controlplane] => (item=proc_type.stdout) => {
    "ansible_loop_var": "item",
    "item": "proc_type.stdout",
    "proc_type.stdout": "Intel Xeon E312xx (Sandy Bridge"
}
ok: [controlplane] => (item=socket.stdout) => {
    "ansible_loop_var": "item",
    "item": "socket.stdout",
    "socket.stdout": "1"
}
ok: [node01] => (item=ansible_hostname) => {
    "ansible_hostname": "node01",
    "ansible_loop_var": "item",
    "item": "ansible_hostname"
}
ok: [node01] => (item=ansible_system_vendor) => {
    "ansible_loop_var": "item",
    "ansible_system_vendor": "KubeVirt",
    "item": "ansible_system_vendor"
}
ok: [node01] => (item=ansible_product_name) => {
    "ansible_loop_var": "item",
    "ansible_product_name": "None",
    "item": "ansible_product_name"
}
ok: [node01] => (item=proc_type.stdout) => {
    "ansible_loop_var": "item",
    "item": "proc_type.stdout",
    "proc_type.stdout": "Intel Xeon E312xx (Sandy Bridge"
}
ok: [node01] => (item=socket.stdout) => {
    "ansible_loop_var": "item",
    "item": "socket.stdout",
    "socket.stdout": "1"
}

TASK [data_gather : Deploy the Template] ***************************************************************************************************************************
task path: /root/playbooks/roles/data_gather/tasks/main.yml:32
changed: [controlplane -> localhost] => {"changed": true, "checksum": "ff7db67a57a23f00a3299fe9512c9c60042cea7e", "dest": "/root/playbooks/reports/report.20240118T122852.csv", "gid": 0, "group": "root", "md5sum": "7093053e66e52d2bfff2907c392295da", "mode": "0644", "owner": "root", "size": 230, "src": "/root/.ansible/tmp/ansible-tmp-1705580934.225866-50602370224168/source", "state": "file", "uid": 0}
META: ran handlers
META: ran handlers

PLAY RECAP *********************************************************************************************************************************************************
controlplane               : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node01                     : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

controlplane $ ls -l /root/playbooks/reports/*.csv
-rw-r--r-- 1 root root 230 Jan 18 12:26 /root/playbooks/reports/report.20240118T122613.csv
-rw-r--r-- 1 root root 230 Jan 18 12:28 /root/playbooks/reports/report.20240118T122852.csv
controlplane $ cat /root/playbooks/reports/*.csv
Date,Node Name,Server Make,Server Model,Processor Type and Model,Socket #
2024-01-18 12:26:13,controlplane,KubeVirt,None,Intel Xeon E312xx (Sandy Bridge,1
2024-01-18 12:26:13,node01,KubeVirt,None,Intel Xeon E312xx (Sandy Bridge,1
Date,Node Name,Server Make,Server Model,Processor Type and Model,Socket #
2024-01-18 12:28:52,controlplane,KubeVirt,None,Intel Xeon E312xx (Sandy Bridge,1
2024-01-18 12:28:52,node01,KubeVirt,None,Intel Xeon E312xx (Sandy Bridge,1


controlplane $ cat /etc/ansible/ansible.cfg
# config file for ansible -- https://ansible.com/
# ===============================================

# nearly all parameters can be overridden in ansible-playbook
# or with command line flags. ansible will read ANSIBLE_CONFIG,
# ansible.cfg in the current working directory, .ansible.cfg in
# the home directory or /etc/ansible/ansible.cfg, whichever it
# finds first

[defaults]

# some basic default values...

#inventory      = /etc/ansible/hosts
#library        = /usr/share/my_modules/
#module_utils   = /usr/share/my_module_utils/
#remote_tmp     = ~/.ansible/tmp
#local_tmp      = ~/.ansible/tmp
#plugin_filters_cfg = /etc/ansible/plugin_filters.yml
#forks          = 5
#poll_interval  = 15
#sudo_user      = root
#ask_sudo_pass = True
#ask_pass      = True
#transport      = smart
#remote_port    = 22
#module_lang    = C
#module_set_locale = False

# plays will gather facts by default, which contain information about
# the remote system.
#
# smart - gather by default, but don't regather if already gathered
# implicit - gather by default, turn off with gather_facts: False
# explicit - do not gather by default, must say gather_facts: True
#gathering = implicit

# This only affects the gathering done by a play's gather_facts directive,
# by default gathering retrieves all facts subsets
# all - gather all subsets
# network - gather min and network facts
# hardware - gather hardware facts (longest facts to retrieve)
# virtual - gather min and virtual facts
# facter - import facts from facter
# ohai - import facts from ohai
# You can combine them using comma (ex: network,virtual)
# You can negate them using ! (ex: !hardware,!facter,!ohai)
# A minimal set of facts is always gathered.
#gather_subset = all

# some hardware related facts are collected
# with a maximum timeout of 10 seconds. This
# option lets you increase or decrease that
# timeout to something more suitable for the
# environment.
# gather_timeout = 10

# Ansible facts are available inside the ansible_facts.* dictionary
# namespace. This setting maintains the behaviour which was the default prior
# to 2.5, duplicating these variables into the main namespace, each with a
# prefix of 'ansible_'.
# This variable is set to True by default for backwards compatibility. It
# will be changed to a default of 'False' in a future release.
# ansible_facts.
# inject_facts_as_vars = True

# additional paths to search for roles in, colon separated
#roles_path    = /etc/ansible/roles

# uncomment this to disable SSH key host checking
#host_key_checking = False

# change the default callback, you can only have one 'stdout' type  enabled at a time.
#stdout_callback = skippy


## Ansible ships with some plugins that require whitelisting,
## this is done to avoid running all of a type by default.
## These setting lists those that you want enabled for your system.
## Custom plugins should not need this unless plugin author specifies it.

# enable callback plugins, they can output to stdout but cannot be 'stdout' type.
#callback_whitelist = timer, mail

# Determine whether includes in tasks and handlers are "static" by
# default. As of 2.0, includes are dynamic by default. Setting these
# values to True will make includes behave more like they did in the
# 1.x versions.
#task_includes_static = False
#handler_includes_static = False

# Controls if a missing handler for a notification event is an error or a warning
#error_on_missing_handler = True

# change this for alternative sudo implementations
#sudo_exe = sudo

# What flags to pass to sudo
# WARNING: leaving out the defaults might create unexpected behaviours
#sudo_flags = -H -S -n

# SSH timeout
#timeout = 10

# default user to use for playbooks if user is not specified
# (/usr/bin/ansible will use current user as default)
#remote_user = root

# logging is off by default unless this path is defined
# if so defined, consider logrotate
#log_path = /var/log/ansible.log

# default module name for /usr/bin/ansible
#module_name = command

# use this shell for commands executed under sudo
# you may need to change this to bin/bash in rare instances
# if sudo is constrained
#executable = /bin/sh

# if inventory variables overlap, does the higher precedence one win
# or are hash values merged together?  The default is 'replace' but
# this can also be set to 'merge'.
#hash_behaviour = replace

# by default, variables from roles will be visible in the global variable
# scope. To prevent this, the following option can be enabled, and only
# tasks and handlers within the role will see the variables there
#private_role_vars = yes

# list any Jinja2 extensions to enable here:
#jinja2_extensions = jinja2.ext.do,jinja2.ext.i18n

# if set, always use this private key file for authentication, same as
# if passing --private-key to ansible or ansible-playbook
#private_key_file = /path/to/file

# If set, configures the path to the Vault password file as an alternative to
# specifying --vault-password-file on the command line.
#vault_password_file = /path/to/vault_password_file

# format of string {{ ansible_managed }} available within Jinja2
# templates indicates to users editing templates files will be replaced.
# replacing {file}, {host} and {uid} and strftime codes with proper values.
#ansible_managed = Ansible managed: {file} modified on %Y-%m-%d %H:%M:%S by {uid} on {host}
# {file}, {host}, {uid}, and the timestamp can all interfere with idempotence
# in some situations so the default is a static string:
#ansible_managed = Ansible managed

# by default, ansible-playbook will display "Skipping [host]" if it determines a task
# should not be run on a host.  Set this to "False" if you don't want to see these "Skipping"
# messages. NOTE: the task header will still be shown regardless of whether or not the
# task is skipped.
#display_skipped_hosts = True

# by default, if a task in a playbook does not include a name: field then
# ansible-playbook will construct a header that includes the task's action but
# not the task's args.  This is a security feature because ansible cannot know
# if the *module* considers an argument to be no_log at the time that the
# header is printed.  If your environment doesn't have a problem securing
# stdout from ansible-playbook (or you have manually specified no_log in your
# playbook on all of the tasks where you have secret information) then you can
# safely set this to True to get more informative messages.
#display_args_to_stdout = False

# by default (as of 1.3), Ansible will raise errors when attempting to dereference
# Jinja2 variables that are not set in templates or action lines. Uncomment this line
# to revert the behavior to pre-1.3.
#error_on_undefined_vars = False

# by default (as of 1.6), Ansible may display warnings based on the configuration of the
# system running ansible itself. This may include warnings about 3rd party packages or
# other conditions that should be resolved if possible.
# to disable these warnings, set the following value to False:
#system_warnings = True

# by default (as of 1.4), Ansible may display deprecation warnings for language
# features that should no longer be used and will be removed in future versions.
# to disable these warnings, set the following value to False:
#deprecation_warnings = True

# (as of 1.8), Ansible can optionally warn when usage of the shell and
# command module appear to be simplified by using a default Ansible module
# instead.  These warnings can be silenced by adjusting the following
# setting or adding warn=yes or warn=no to the end of the command line
# parameter string.  This will for example suggest using the git module
# instead of shelling out to the git command.
# command_warnings = False


# set plugin path directories here, separate with colons
#action_plugins     = /usr/share/ansible/plugins/action
#become_plugins     = /usr/share/ansible/plugins/become
#cache_plugins      = /usr/share/ansible/plugins/cache
#callback_plugins   = /usr/share/ansible/plugins/callback
#connection_plugins = /usr/share/ansible/plugins/connection
#lookup_plugins     = /usr/share/ansible/plugins/lookup
#inventory_plugins  = /usr/share/ansible/plugins/inventory
#vars_plugins       = /usr/share/ansible/plugins/vars
#filter_plugins     = /usr/share/ansible/plugins/filter
#test_plugins       = /usr/share/ansible/plugins/test
#terminal_plugins   = /usr/share/ansible/plugins/terminal
#strategy_plugins   = /usr/share/ansible/plugins/strategy


# by default, ansible will use the 'linear' strategy but you may want to try
# another one
#strategy = free

# by default callbacks are not loaded for /bin/ansible, enable this if you
# want, for example, a notification or logging callback to also apply to
# /bin/ansible runs
#bin_ansible_callbacks = False


# don't like cows?  that's unfortunate.
# set to 1 if you don't want cowsay support or export ANSIBLE_NOCOWS=1
#nocows = 1

# set which cowsay stencil you'd like to use by default. When set to 'random',
# a random stencil will be selected for each task. The selection will be filtered
# against the `cow_whitelist` option below.
#cow_selection = default
#cow_selection = random

# when using the 'random' option for cowsay, stencils will be restricted to this list.
# it should be formatted as a comma-separated list with no spaces between names.
# NOTE: line continuations here are for formatting purposes only, as the INI parser
#       in python does not support them.
#cow_whitelist=bud-frogs,bunny,cheese,daemon,default,dragon,elephant-in-snake,elephant,eyes,\
#              hellokitty,kitty,luke-koala,meow,milk,moofasa,moose,ren,sheep,small,stegosaurus,\
#              stimpy,supermilker,three-eyes,turkey,turtle,tux,udder,vader-koala,vader,www

# don't like colors either?
# set to 1 if you don't want colors, or export ANSIBLE_NOCOLOR=1
#nocolor = 1

# if set to a persistent type (not 'memory', for example 'redis') fact values
# from previous runs in Ansible will be stored.  This may be useful when
# wanting to use, for example, IP information from one group of servers
# without having to talk to them in the same playbook run to get their
# current IP information.
#fact_caching = memory

#This option tells Ansible where to cache facts. The value is plugin dependent.
#For the jsonfile plugin, it should be a path to a local directory.
#For the redis plugin, the value is a host:port:database triplet: fact_caching_connection = localhost:6379:0

#fact_caching_connection=/tmp

# retry files
# When a playbook fails a .retry file can be created that will be placed in ~/
# You can enable this feature by setting retry_files_enabled to True
# and you can change the location of the files by setting retry_files_save_path

#retry_files_enabled = False
#retry_files_save_path = ~/.ansible-retry

# squash actions
# Ansible can optimise actions that call modules with list parameters
# when looping. Instead of calling the module once per with_ item, the
# module is called once with all items at once. Currently this only works
# under limited circumstances, and only with parameters named 'name'.
#squash_actions = apk,apt,dnf,homebrew,pacman,pkgng,yum,zypper

# prevents logging of task data, off by default
#no_log = False

# prevents logging of tasks, but only on the targets, data is still logged on the master/controller
#no_target_syslog = False

# controls whether Ansible will raise an error or warning if a task has no
# choice but to create world readable temporary files to execute a module on
# the remote machine.  This option is False by default for security.  Users may
# turn this on to have behaviour more like Ansible prior to 2.1.x.  See
# https://docs.ansible.com/ansible/become.html#becoming-an-unprivileged-user
# for more secure ways to fix this than enabling this option.
#allow_world_readable_tmpfiles = False

# controls the compression level of variables sent to
# worker processes. At the default of 0, no compression
# is used. This value must be an integer from 0 to 9.
#var_compression_level = 9

# controls what compression method is used for new-style ansible modules when
# they are sent to the remote system.  The compression types depend on having
# support compiled into both the controller's python and the client's python.
# The names should match with the python Zipfile compression types:
# * ZIP_STORED (no compression. available everywhere)
# * ZIP_DEFLATED (uses zlib, the default)
# These values may be set per host via the ansible_module_compression inventory
# variable
#module_compression = 'ZIP_DEFLATED'

# This controls the cutoff point (in bytes) on --diff for files
# set to 0 for unlimited (RAM may suffer!).
#max_diff_size = 1048576

# This controls how ansible handles multiple --tags and --skip-tags arguments
# on the CLI.  If this is True then multiple arguments are merged together.  If
# it is False, then the last specified argument is used and the others are ignored.
# This option will be removed in 2.8.
#merge_multiple_cli_flags = True

# Controls showing custom stats at the end, off by default
#show_custom_stats = True

# Controls which files to ignore when using a directory as inventory with
# possibly multiple sources (both static and dynamic)
#inventory_ignore_extensions = ~, .orig, .bak, .ini, .cfg, .retry, .pyc, .pyo

# This family of modules use an alternative execution path optimized for network appliances
# only update this setting if you know how this works, otherwise it can break module execution
#network_group_modules=eos, nxos, ios, iosxr, junos, vyos

# When enabled, this option allows lookups (via variables like {{lookup('foo')}} or when used as
# a loop with `with_foo`) to return data that is not marked "unsafe". This means the data may contain
# jinja2 templating language which will be run through the templating engine.
# ENABLING THIS COULD BE A SECURITY RISK
#allow_unsafe_lookups = False

# set default errors for all plays
#any_errors_fatal = False

[inventory]
# enable inventory plugins, default: 'host_list', 'script', 'auto', 'yaml', 'ini', 'toml'
#enable_plugins = host_list, virtualbox, yaml, constructed

# ignore these extensions when parsing a directory as inventory source
#ignore_extensions = .pyc, .pyo, .swp, .bak, ~, .rpm, .md, .txt, ~, .orig, .ini, .cfg, .retry

# ignore files matching these patterns when parsing a directory as inventory source
#ignore_patterns=

# If 'true' unparsed inventory sources become fatal errors, they are warnings otherwise.
#unparsed_is_failed=False

[privilege_escalation]
#become=True
#become_method=sudo
#become_user=root
#become_ask_pass=False

[paramiko_connection]

# uncomment this line to cause the paramiko connection plugin to not record new host
# keys encountered.  Increases performance on new host additions.  Setting works independently of the
# host key checking setting above.
#record_host_keys=False

# by default, Ansible requests a pseudo-terminal for commands executed under sudo. Uncomment this
# line to disable this behaviour.
#pty=False

# paramiko will default to looking for SSH keys initially when trying to
# authenticate to remote devices.  This is a problem for some network devices
# that close the connection after a key failure.  Uncomment this line to
# disable the Paramiko look for keys function
#look_for_keys = False

# When using persistent connections with Paramiko, the connection runs in a
# background process.  If the host doesn't already have a valid SSH key, by
# default Ansible will prompt to add the host key.  This will cause connections
# running in background processes to fail.  Uncomment this line to have
# Paramiko automatically add host keys.
#host_key_auto_add = True

[ssh_connection]

# ssh arguments to use
# Leaving off ControlPersist will result in poor performance, so use
# paramiko on older platforms rather than removing it, -C controls compression use
#ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s

# The base directory for the ControlPath sockets.
# This is the "%(directory)s" in the control_path option
#
# Example:
# control_path_dir = /tmp/.ansible/cp
#control_path_dir = ~/.ansible/cp

# The path to use for the ControlPath sockets. This defaults to a hashed string of the hostname,
# port and username (empty string in the config). The hash mitigates a common problem users
# found with long hostnames and the conventional %(directory)s/ansible-ssh-%%h-%%p-%%r format.
# In those cases, a "too long for Unix domain socket" ssh error would occur.
#
# Example:
# control_path = %(directory)s/%%h-%%r
#control_path =

# Enabling pipelining reduces the number of SSH operations required to
# execute a module on the remote server. This can result in a significant
# performance improvement when enabled, however when using "sudo:" you must
# first disable 'requiretty' in /etc/sudoers
#
# By default, this option is disabled to preserve compatibility with
# sudoers configurations that have requiretty (the default on many distros).
#
#pipelining = False

# Control the mechanism for transferring files (old)
#   * smart = try sftp and then try scp [default]
#   * True = use scp only
#   * False = use sftp only
#scp_if_ssh = smart

# Control the mechanism for transferring files (new)
# If set, this will override the scp_if_ssh option
#   * sftp  = use sftp to transfer files
#   * scp   = use scp to transfer files
#   * piped = use 'dd' over SSH to transfer files
#   * smart = try sftp, scp, and piped, in that order [default]
#transfer_method = smart

# if False, sftp will not use batch mode to transfer files. This may cause some
# types of file transfer failures impossible to catch however, and should
# only be disabled if your sftp version has problems with batch mode
#sftp_batch_mode = False

# The -tt argument is passed to ssh when pipelining is not enabled because sudo 
# requires a tty by default. 
#usetty = True

# Number of times to retry an SSH connection to a host, in case of UNREACHABLE.
# For each retry attempt, there is an exponential backoff,
# so after the first attempt there is 1s wait, then 2s, 4s etc. up to 30s (max).
#retries = 3

[persistent_connection]

# Configures the persistent connection timeout value in seconds.  This value is
# how long the persistent connection will remain idle before it is destroyed.
# If the connection doesn't receive a request before the timeout value
# expires, the connection is shutdown. The default value is 30 seconds.
#connect_timeout = 30

# The command timeout value defines the amount of time to wait for a command
# or RPC call before timing out. The value for the command timeout must
# be less than the value of the persistent connection idle timeout (connect_timeout)
# The default value is 30 second.
#command_timeout = 30

[accelerate]
#accelerate_port = 5099
#accelerate_timeout = 30
#accelerate_connect_timeout = 5.0

# The daemon timeout is measured in minutes. This time is measured
# from the last activity to the accelerate daemon.
#accelerate_daemon_timeout = 30

# If set to yes, accelerate_multi_key will allow multiple
# private keys to be uploaded to it, though each user must
# have access to the system via SSH to add a new key. The default
# is "no".
#accelerate_multi_key = yes

[selinux]
# file systems that require special treatment when dealing with security context
# the default behaviour that copies the existing context or uses the user default
# needs to be changed to use the file system dependent context.
#special_context_filesystems=nfs,vboxsf,fuse,ramfs,9p,vfat

# Set this to yes to allow libvirt_lxc connections to work without SELinux.
#libvirt_lxc_noseclabel = yes

[colors]
#highlight = white
#verbose = blue
#warn = bright purple
#error = red
#debug = dark gray
#deprecate = purple
#skip = cyan
#unreachable = red
#ok = green
#changed = yellow
#diff_add = green
#diff_remove = red
#diff_lines = cyan


[diff]
# Always print diff when running ( same as always running with -D/--diff )
# always = no

# Set how many context lines to show in diff
# context = 3


Look at you, learning Ansible! You created a role that pulled information and then reported it out in a .csv format to keep the management fed full of tasty information!
```
