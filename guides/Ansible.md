---
tags: [devops]
title: Ansible
created: '2020-05-29T23:06:56.614Z'
modified: '2020-07-30T16:12:46.412Z'
---

# Ansible

In here you will find useful notes about Ansible and how to work with it. This includes:

Inventory, Adhoc, Ansible Playbooks, Variables, Conditionals & Loops, Templates, Fact Gathering, Handlers, Tags, and Troubleshooting. 

Example code is also included within the documentation. This guide is still being updated.



## Inventory

We define hosts here in which ansible will run on. Usually you can find them in `/etc/ansible/ansible.cfg`, but you can create your own inventory file and tell ansible to use it. More on this later but quickly, it would look like this:

`ansible-playbook -i inv myPlaybook.yml`
in which the `-i` flag specified an `inv` dir that we created in which our hosts exists, then specified the playbook. More info will be given further down in the guide.

## Adhoc

Adhocs are single commands, where we provide one instruction to be executed on one host or a group of target hosts. So to create user on target user, that's an adhoc command. 

Playbooks are more like a script that target a group of hosts.


adhoc syntax:
`ansible <HOST> -b -m <MODULE> -a "<ARG1 ARG2 ARG3>"`

- host: host or group defined in inventory file
- b: become, elevates to sudo
- m: module that allows a command to be used on a module
- a: allows parameters to pass, if used without a module, it runs as a shell command on target system

example: `ansible myHost -b -m yum -a "name=httpd state=latest"` will use yum module to install a package

example: `ansible myHost -b -m service -a "name=httpd state=started"` will use service module to start service

example: `ansible -i inv hosts -m setup` will gather facts for `hosts` in your inv

example: `ansible -i inv hosts -b -m apt -a "name=nginx state=installed` will install nginx on hosts in inv 

example: `ansible -i inv hosts -b -m apt -a "name=nginx state-=installed -f 10` will install nginx on hosts in inv by running 10 forks instead of the default 5

example: `ansible localhost -a "echo $PATH > /path/to/file"` exports path var to a file

example: `ansible -i inv hosts -m file -a "name=myFile state=touch"` uses `file` module to create a file in working dir

example: `ansible -i inv hosts -m copy -a "src=myFile dest=myNewFile"` copy file to dest

example: `ansible -i inv hosts -m lineinfile -a "path= /path/to/file/to/edit line='i am an edit' state=present"`
allows you to edit file 

example: `ansible -i inv hosts -m archive -a "path= /path/to/file/to/edit format= zip dest= /dest/path/file.zip`

use `ansible-doc` to see more modules or view documentation online


## Ansible Playbooks

`ansible-playbook` is used as the command to run a yml file that provides instructions on what to do, how to configure the environment, etc. The playbook is broken down into plays that are run against a group of hosts.
So you can define one play for a one group of hosts and then another underneath it.

There are tasks in every play and you can define however many tasks as you like. `name` just gives some info for the task that you freely define, then you specify the module to use. There are many useful modules to use. Use `#` for comments.

You can define your own inv file and define the hosts in there, like `/home/ansible/inv` and add them there.
You don't need to use the default inv files. 

example playbook:
~~~
--- #bootstrap webservers

- hosts: webservers
  become: yes
  tasks:
  - name: install nginx
    apt:
      name: nginx
      state: latest
~~~

to run: `ansible-playbook -i inv playbookName.yml`

- i: allows you to specify a non-default inventory file, otherwise it'll use the default
After that you pass the name of the ansible playbook itself.

You can add these flags as well:

- `-K` to supply `become` password
- `-k` to supply `connect` passwod
- `-C` to run in check mode (a dry run)
- `--check` flag and ansible will provide a sanity check without running

When a playbook fails, it generates a retry file `playbookName.retry` where it'll show you which hosts failed. You can limit the run using `--limit` to reattempt the specific playbook against the specific hosts.

cat the failed file to see the hosts that failed


### Variables

You can use vars in commands passed using `-e` or `--extra-vars` or defined within a playbook.

In playbooks, it's good practice to wrap them like this: 
~~~
---
hosts: webservers
become: yes
vars:
  target_service: nginx
  target_state: started
tasks:
  - name: start nginx
    service:
      name: "{{target_service}}"
      state: "{{target_state}}"
~~~


In cli: 
~~~
ansible-playbook service.yml -e "target_hosts=localhost"
~~~

Alternatively, you can create a yml file containing variables. Example:
~~~
service_list:
- nginx
- httpd

working_dir: /home/ansible/dir

share_paths:
nfs: /mnt/nfs
cifs:/mnt/cifs
~~~

The first type is a list type, then a reg var, then a dictionary or array type.
To reference an index in dict or array in a playbook: "{{ share_paths['nfs']}}"

Now you need to tell ansible to look for that yml file in which you define variables:
`ansible-playbook playbook.yml -e @vars_file.yml ` 
This will tell ansible to look for the vars there.

## Conditionals and Loops

`when` keyword allows you to apply conditional tests inside playbooks.
~~~
---
- hosts: myHosts
  vars:
    target_file: /home/ansible/myHostName
  tasks:
    - name: collect host name info
      stat:
        path: "{{ target_file }}"
      register: myHostName
    - name: rename the file
      command: mv "{{ target_file }}" /home/ansible/yourHostName
      when: hostname.stat.exists
~~~
See that last line? We used stat to apply the `when` module to check if the file exists. The dots indicate that you are accessing a dictionary. It is equivalent to: `hostname['stat']['exists']`

You could also test equivalence of things of the same type too by using `not` before the line or `==` or `<=` etc


Loop example:
~~~
---
- hosts: remote
  become: yes
  tasks:
    - name:
      apt:
        name: "{{ items }}"
        state: latest
      loop:
        - curl
        - wget
        - sudo
~~~
apt will run each time replacing the items var with the values in the loop one at a time. You could also use a variable for the loop but make sure to pass the var_files:
~~~
---
- hosts: remote
  become: yes
  var_files:
    - vars.yml
  tasks:
    - name:
      apt:
        name: "{{ items }}"
        state: latest
      loop:"{{ install_tools }}"
~~~
Where your tools to install will be defined in `vars.yml`
~~~
---
install_tools:
- curl
- wget
- sudo
~~~

### Templates

You can use preconfigured files as templates for ansible to use by using the `template` module.
Example: cfg file
~~~
name 
~~~

### Ansible Facts

   
These gather properties about a given remote system and you can filter the result. The module setup can do that for you:
`ansible hostName -m setup -a filter=*ipv4*` and it'll return ipv4 details, you can also return `hostname`

You can also save the output in a file in json form that you can manipulate later to extract info:
`ansible -i inv hosts -m setup --tree output_file_name`

You can use this in a playbook to insert the hostname:
~~~
- hosts: webservers
  become: yes
  tasks:
  - name: insert host name into file
    lineinfile:
      line: "{{ ansible_hostname }}"
      path: /path/to/file
~~~
after it gather facts, it'll add the host name into that file.

## Handlers

These allows actions to be flagged for execution when a task performs a change. For example, say you run a playbook that installs nginx then adjusts the config file. You can define a handler that restarts the daemon when that action is done by adding `notify`.

### Tags: Running Selective Parts

You can use tags to allow a play or a task to be selectively run, or be skipped entirely, based on the tags. You just have to specify the tag and what you want Ansible to do with it.

myPlaybook.yml:
~~~
---
- hosts: remote
  become: yes
  tasks:
    - name:
      apt:
        name: "{{ items }}"
        state: latest
      loop:
        - curl
        - wget
        - sudo
      tags: 
        - install
    - name: start nginx
      service:
        name: "{{target_service}}"
        state: "{{target_state}}"
      tags: 
        - runX
~~~

So as you can see, we have two tags for each of the tasks we have. Now we can pass them to the ansible command to tell it which task to run. Say we already know we installed NginX already, we can use the `runX` tag to start it!

`ansible-playbook myPlaybook.yml --tags runX`

Other flags:
- `-t`: same as --tags
- `--skip-tags`: specify which tags to skip


## Troubleshooting

You will face all sorts of errors and bugs in your playbooks. From wrong indentations in your yml syntax to forgetting to elevate permissions, and more. So you have to know how to handle it.

There is a debug module to help with that:

~~~
- debug: 
    msg:
~~~
msg: prints a message to STDOUT
var: var that is printed to STDOUT

There is also a register module that stores the task output

~~~
- hosts: webservers
  become: yes
  tasks:
  - debug:
      var: ansible_hostname
  - name: insert host name into file
    lineinfile:
      line: "{{ ansible_hostname }}"
      path: /path/to/file
    register: debug_output
  - debug:
      msg: "the hostname is {{ debug_output }}"
~~~

---

