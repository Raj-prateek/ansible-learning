# Introduction to Ansible

If you're a systems engineer or IT administrator, or just anybody working in IT, you're probably involved in doing a lot of repetitive tasks in your environment, whether it be sizing and creating new hosts or virtual machines every day, applying configurations on them, patching in hundreds of servers, migrations, deploying applications, or even performing security and compliance audits. All of these vary repetitive tasks involve execution of hundreds of commands on hundreds of different servers while maintaining the right sequence of events with system reboots and whatnot in between.

Some people develop scripts to automate these tasks, but that requires coding skills and regular maintenance of these scripts, and a lot of time to put these scripts together on the first place. That's where Ansible helps.

> Ansible is a powerful IT automation tool that you can learn quickly. It's simple enough for everyone in IT, yet powerful enough to automate even the most complex deployments.

In the past, something that took developing a complex script, now takes just a few lines of instruction in an Ansible automation playbook. Whether you want to make that happen on your localhost or on all of your database servers or on web servers or on clouds or on your DR env, all it takes is modifying one line.

## USE CASE

Imagine you have a number of hosts in your environment that you would like to restart in a particular order.Some of them are web server and others are database servers. So you would power down the web server firsts followed by the database serves, and then power up the database servers and then the web servers. You could write an Ansible playbook to get this done in a matter of minutes and simply invoke the ansible playbook every time you wish to restart your application.

## Ansible Inventory

Ansible can work with one or multiple systems in your infrastructure at the same time. In order to work with multiple servers ansible needs to establish connectivity to those servers. This is done using `ssh` for linux and `powershell` for windows.

> That's what makes ansible agentless.

Agentless means that you don't need to install any additional software on the target machines to be able to work with ansible.

A simple ssh connectivity would suffice Ansible's needs. One of the major disadvantages of most other orchestration tools is that you are required to configure an agent on the target systems before you can invoke any kind of automation. Now, information about these target systems is stored in an inventory file.
If you don't create a new inventory file, ansible uses the default inventory file located at `etc/ansible/hosts` location.
The inventory file is in an ini like format. It's simply a number of servers listed, one after the other. You can also group different servers together by defining it like this under the name of the group within square brackets, define the list of servers part of that group in the lines below.

```
server1.company.com
server2.company.com

[mail]
server3.company.com
server4.company.com

```

You can have multiple groups defined in a single inventory file.

```
server1.company.com
server2.company.com

[mail]
server3.company.com
server4.company.com

[db]
server5.company.com
server6.company.com

[web]
server7.company.com
server8.company.com

```

### Inventory Parameters

- **ansible_host**: Adding an alias for each server at the beginning of the line and assigning the address of that server to `ansible_host` parameter. `Ansible_host` is an inventory parameter used to specify the FQDN or IP address of a server.

```
web  ansible_host=server1.company.com
db   ansible_host=server2.company.com
mail ansible_host=server3.company.com
web2 ansible_host=server4.company.com
```

- **ansible_connection**: It define how ansible connect to the target server.

```
#ansible_connection - ssh/winrm/localhost

web  ansible_host=server1.company.com ansible_connection=ssh
db   ansible_host=server2.company.com ansible_connection=winrm
mail ansible_host=server3.company.com ansible_connection=ssh
web2 ansible_host=server4.company.com ansible_connection=winrm

localhost ansible_connection=localhost

```

- **ansible_port**: It define to which port ansible needed to connect. By default it connects to 22 for ssh, but if you need to change, you can set it differently using ansible_port parameter.

```
ansible_port - 22/5986
```

- **ansible_user**: It defines the user to use during creating a connection.

```
#ansible_user - root/admin

web  ansible_host=server1.company.com ansible_connection=ssh ansible_user=root
db   ansible_host=server2.company.com ansible_connection=winrm ansible_user=admin
mail ansible_host=server3.company.com ansible_connection=ssh
web2 ansible_host=server4.company.com ansible_connection=winrm

localhost ansible_connection=localhost
```

- **ansible_ssh_pass**: It defines the password to use during creating a connection.

```
#ansible_ssh_pass - <Password>

web  ansible_host=server1.company.com ansible_connection=ssh ansible_user=root
db   ansible_host=server2.company.com ansible_connection=winrm ansible_user=admin
mail ansible_host=server3.company.com ansible_connection=ssh ansible_ssh_pass=P@#$$
web2 ansible_host=server4.company.com ansible_connection=winrm

localhost ansible_connection=localhost
```

## Ansible Playbook

Ansible playbooks are ansible's orchestration language. It is in playbooks where we define what we want ansible to do. It is a set of instructions you provide ansible to work it's magic.

For example, it can be as simple as running a series of commands on different servers in a sequence, and restarting those servers in a particular order.

```
# Simple Ansible Playbook

- Run command1 on server1
- Run command2 on server2
- Run command3 on server3
- Run command4 on server4
- Run command5 on server5
- Run command6 on server6
- Restarting Server1
- Restarting Server2
- Restarting Server3
- Restarting Server4
- Restarting Server5
- Restarting Server6

```

- Playbook - A single YAML file.
  - Play - Defines a set of activities(tasks) to be run on hosts.
    - Task - An action to be performed on the host
      - Execute a command
      - Run a script
      - Install a package
      - Shutdown/Restart

> The host you wanna run these actions is defined at the play level. In below case, we just want to test on the localhost, which is why it is said to localhost. This could be anything from your inventory file.

```
name: Play 1
hosts: localhost
tasks: 
  - name: Execute command 'date'
    command: date
  - name: Execute script on server
    command: test_script.sh
  - name: Install httpd service
    yum:
      name: httpd
      state: present
  - name: Start web server
    service:
      name: httpd
      state: started
```

#### Modules:

The different actions run by tasks are called modules. In the above example, command, script, yum and service are ansible modules. There are hundreds of other modules available out of the box. Information about these modules is available in the Ansible documentation website, or you could simply run the `ansible-doc -l` command.

### Run/Execute Playbook

```
ansible-playbook <playbook filename>
```

## Ansible Vs Ansible-playbook command.

Sometimes you may want to use ansible for a one of the task such as to test connectivity between the ansible controller and the targets, or to run a command, for example to shut down a set of servers. In that case, you can get away without writing a playbook by running the ansible command i.e,

```
ansible <hosts> -a <command>
ansible all -a "/sbin/reboot"
ansible <hosts> -m <module>
ansible target1 -m ping
```


```
ansible-playbook <playbook name>
```

### Ansible Modules:

Ansible modules are categorized, into various groups based on their functionality. Some of them are list here:

- **System modules** - action to be performed at system level. Such as modifyin IP tables, modifying users and groups on the system, firewall configurations on the system, working with logical volume groups, mounting operation, ping, timezone, systemd & service level modification.
- Commands - Are used to excute commands or scripts on a host.
- File - use the ACL module to set and retrieve ACL information on files, Use the archive and un-archive modules to compress etc.
- Database - MongoDB, MySQL, MSSQL etc
- Cloud - Amazon, docker, google, openstack cloud provide configuration.
- Windows - Use to configure windows server.
- More ...

**command:**

Executes a command on a remote node

| First Header  | Second Header |
| ------------- | ------------- |
| chdir  | cd into this directory before running the command  |
| creates  | a filename or (since 2.0) glob pattern, when it already exists, this step will not be run. |
| executable  | change the shell used to execute the command. Should be an absolute path to the executable. |
| free_form  | the command module takes a free from command to run. There is no parameter actually named `free from` |
| removes  | a filename or (since 2.0) glob pattern, when it doesn't exists, this step will not be run. |
| warn  | if command warnings are on in ansible.cfg, do not warn about this particular line if set to no/false. |

```
- name: Play 1
  hosts: loacalhost
  tasks:
    - name: Execute command 'date'
      command: date
    - name: Display resolv.cnf contents
      command: cat /etc/resolv.conf
    - name: Display resolv.conf contents
      command: cat resolv.conf chdir/etc
    - name: Display resolv.conf contents
      command: mkdir /folder creates=/folder
    - name: Copy files from source to destination
      copy: src=/source_file dest=/destination
```

**script:**

- Runs a local script on a remote node after transferring it.

What ansible does for us?

1. Copy script to remote systems.
2. Execute script on remote systems.

```
- name: Play 1
  hosts: localhost
  tasks:
    - name: Run a script on remote server
      script: /some/local/script.sh --arg1 arg2
```

**service:**

- Manage Services - start, stop, restart

```
- name: Start services in order
  hosts: localhost
  tasks:
    - name: start the database service
      service: name=postgresql state=started
    - name: start the httpd service
      service: name=httpd state=started
    - name: start the nginx service
      service:
        name: nginx
        state: started

```

```
- name: Start services in order
  hosts: localhost
  tasks:
    - name: start the database service
      service: 
        name:postgresql 
        state:started
      
```

Why `started` and not `start`?

`Start` the service httpd vs `Started` the service httpd

Ensure service httpd is started

if httpd is not already started => start it
if httpd is already started, => do nothing

> **Idempotent** An operation is idempotent if the result of performing it once is exactly the same as the result of performing it repeatedly without any intervening actions.

Thus started, stopped and restarted actions, instruct ansible to ensure the service, is in a particular state.
Majority of the modules in ansible, are idempotent and ansible highly recommends this.

The overall idea is that, you should be able to run the same playbook again and ansible should report, that everything is in expected state.
If something is not, ansible takes care of putting it to the expected state.

**lineinfile**

- Search for a line in a file and replace it or add it if it doesn't exit.

For example, we're given a task to add a new DNS server, into the `/etc/resolv.conf` file.

```
nameserver 10.1.250.1
nameserver 10.1.250.2
```

```
- name: Add DNS server to resolv.conf
  hosts: localhost
  tasks:
    - lineinfile:
        path: /etc/resolv.conf
        line: 'nameserver 10.1.250.10'
```
This simple ansible playbook using the lininfile task, adds the new nameserver information, into the `/etc/resolv.conf` file.
Remember, the lineinfile module is idempotent.

Comparison between simple script vs ansible playbook:
If this script is run multiple times, it's add a new entry in the file each time, which is not desired. On the other hand, if you run the ansible playbook multiple times, it will ensure there's only a single entry in the file.

### Ansible Variables

Stores information that varies with each host.
For example, let's say we are trying to perform the same operation of applying patches to hundreds of servers, we only need a single playbook for all hundred servers.
However, it's the variables that store information about the different host names, usernames or passwords that are different for each server.

**Variables in inventory:**

```
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_ssh_pass=P@ssW
db ansible_host=server2.company.com ansible_connection=winrm ansible_ssh_pass=P@s
web2 ansible_host=server3.company.com ansible_connection=ssh ansible_ssh_pass=P@ss
```

ansible_connection, ansible_host & ansible_ssh_pass are the examples of variables.

**Variables in playbook:**

```
- name: Add DNS server to resolv.conf
  hosts: localhost
  vars:
    dns_server: 10.1.250.10
  tasks:
    - lineinfile:
        path: /etc/resolv.conf
        line: 'nameserver {{ dns_server }}'
```

Uses jinja2 templating for replacing variable in playbook.

### Conditionals

Let us take two playbooks, that does the same thing, install NGINX on a host. But as you know, different OS flavours use different package managers.

```
- name: Install Nginx
  hosts: debian_hosts
  tasks:
    - name: Install Nginx on debian
      apt:
        name: nginx
        state: present
```

```
- name: Install Nginx
  hosts: redhat_hosts
  tasks:
    - name: Install nginx on redhat
      yum:
        name: nginx
        state: present
```

How can we convert the above 2 different playbooks into a single playbook?

```
- name: Install Nginx
  hosts: all
  tasks:
    - name: Install Nginx on debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian" and ansible_distribution_version == "16.04"  #if true than run
    - name: Install nginx on redhat
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat" or ansible_os_family == "SUSE" #if true than run
```

Operators:

- `==` to check equal condition
- `!=` to check not equal condition.
- `or` to check or statement in the condition.
- `and` to check and statement in the condition.

Conditionals in Loops

```
- name: Install Softwares
  hosts: all
  vars:
    packages:
      - name: nginx
        required: True
      - name: mysql
        required: True
      - name: apache
        required: False
  tasks:
  - name: Install "{{ item.name }}" on Debian
    apt:
      name: "{{ item.name }}"
      state: present
    when: item.required == True
    loop: "{{ packages }}"
```

Conditionals & Register


```
- name: Check the status of the service and email if it's down
  hosts: localhost
  tasks:
    - command: service httpd status
      register: result
    - mail:
        to: admin@brevo.com
        subject: Service Alert
        body: Httpd service is down
        when: result.stdout.find('down') != -1
```

### Ansible Loops

Loop is a looping directive that executes the same task multiple number of times. Each time it runs, it stores the value of each item in the loop in a variable named "item".

```
- name: create users
  hosts: localhost
  tasks:
    - user: name="{{ item }}"  state=present
      loop:
        - joe
        - ross
        - rachel
        - monica
        - bing
```

```
- name: create users
  hosts: localhost
  tasks:
    - user: name="{{ item.name }}"  state=present  uid="{{ item.uid }}"
      loop:
        - name: joe
          uid: 1010
        - name: ross
          uid: 1011
        - name: rachel
          uid: 1012
        - name: monica
          uid: 1013
        - name: bing
          uid: 1014
```

**with directives**: Look-up plugins

- *with_items*, iterates over a list of items.
  ```
  - name: create users
    hosts: localhost
    tasks:
      - user: name="{{ item }}"  state=present
        with_items:
          - joe
          - ross
          - rachel
          - monica
          - bing
  ```

- *with_file*, iterates over a multiples files.
  ```
  - name: view config files
    hosts: localhost
    tasks:
      - debug: var=item
        with_file:
          - "/etc/hosts"
          - "/etc/resolv.conf"
          - "/etc/ntp.conf"
  ```

- *with_url*, iterates over a URLs.
  ```
  - name: Get from multiple URLS
    hosts: localhost
    tasks:
      - debug: var=item
        with_file:
          - "https://web1.com/"
          - "https://web2.com/"
          - "https://web3.com/"
  ```

- *with_mongodb*, connect to multiple mongo databases.
  ```
  - name: Get from multiple URLS
    hosts: localhost
    tasks:
      - debug: msg="DB={{ item.database }} PID={{ item.pid }}"
        with_mongodb:
          - database: dev
            connection_string: "mongodb://dev.mongo/"
  ```

### Ansible Roles

In ansible world, we assign roles to blank serves to make them a database server or redis messaging server or a backup server.

In automation world, assigning a roles means doing everything you need to do to make a server a database server.

For example,

- Installing Pre-requisites
- Installing mysql packages
- Configuring mysql service
- Configuring database and users.

With the web server, it would be, again

- Installing Pre-requisites
- Installing nginx packages
- Configuring nginx service
- Configuring custom web pages.

These set of tasks, to install and perform basic configuration on a mysql database is going to remain mostly common. Once a person develops this ansible playbook it can be shared with hundreds of thousands of others trying to do the same thing install mysql. Instead of all of them rewriting this piece of code we could package it into a role and reuse it later.
Next time, you could simply assign the role you created and in a playbook like this, be it a single server or hundreds of servers, that's all you need.
So that's the primary purpose of roles, `to make your work reusable`.
