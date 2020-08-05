# ansible-playbook

#### Run docker

```
docker run -d -ti --privileged=true -p 80:80 -p 8080:8080 -p 22:22 congttl/ubuntu:18.04
```

#### Check status host ansible
 
 ```
 ansible -i host web -m ping
 ```
 
 #### Run playbook in ansible
 
 ```
 ansible-playbook -i host playbook.yml
 ```
 
# Module
### [1. File](https://docs.ansible.com/ansible/latest/modules/file_module.html)

* Create Folder/File

```
- name: create folder
  file:
  	path: /home/app/name-folder
  	state: dicrectory

- name: create file
  file: 
  	path: /home/app/file.rb
  	state: touch
```

* Link Folder/File - change ownership

```
- name: Create a symbolic link
  file:
    src: /file/to/link/to
    dest: /path/to/symlink
    state: link
    
- name: Change permisstion ownership
  file:
  	path: /home/app/folder
  	owner: app
  	group: app
```

### [2. Copy](https://docs.ansible.com/ansible/latest/modules/copy_module.html)

```
- name: Copy file with owner and permissions
  copy:
    src: /home/app/foo.conf
    dest: /etc/foo.conf
    owner: app
    group: app
    mode: '0644'
```

### [3. Template](https://docs.ansible.com/ansible/latest/modules/template_module.html)

```
- name: Template a file to /etc/files.conf
  template:
    src: /mytemplates/foo.j2
    dest: /etc/nginx/conf.d/default.conf
    owner: app
    group: app
    mode: '0644'
```

### [4. APT - Manages apt-packages](https://docs.ansible.com/ansible/latest/modules/apt_module.html)

```
- name: Install nginx  (state=present is optional)
  apt:
    name: nginx
    state: present
    
- name: Uninstall nginx
  apt:
    name: nginx
    state: absent
```

### [5. yum – Manages packages with the yum package manager](https://docs.ansible.com/ansible/latest/modules/yum_module.html)

```
- name: install the latest version of Apache
  yum:
    name: httpd
    state: latest
  
- name: ensure a list of packages installed
  yum:
    name: "{{ packages }}"
  vars:
    packages:
    - httpd
    - httpd-tools

- name: remove the Apache package
  yum:
    name: httpd
    state: absent
  
- name: upgrade all packages
  yum:
    name: '*'
    state: latest
```

### [6. Git](https://docs.ansible.com/ansible/latest/modules/git_module.html)

```
- name: git clone https://abc.com
  git: repo=https://abc.com dest=/home/app/folder
```

### [7. Lineinfile](https://docs.ansible.com/ansible/latest/modules/lineinfile_module.html)

```
- name: Add a line to a file if the file does not exist, without passing regexp
  lineinfile:
    path: /home/app/etc/hosts
    line: 127.0.0.0 development.site
    create: yes
```

### [8. blockinfile – Insert/update/remove a text block surrounded by marker lines](https://docs.ansible.com/ansible/latest/modules/blockinfile_module.html)

```
- name: Insert/Update eth0 configuration stanza in /etc/network/interfaces
        (it might be better to copy files into /etc/network/interfaces.d/)
  blockinfile:
    path: /etc/network/interfaces
    block: |
      iface eth0 inet static
          address 192.0.2.23
          netmask 255.255.255.0
```

### [9. Shell](https://docs.ansible.com/ansible/latest/modules/shell_module.html)

```
- name: Run a command that uses non-posix shell-isms (in this example /bin/sh doesn't handle redirection and wildcards together but bash does)
  shell: cat < /tmp/*txt
  args:
    executable: /bin/bash
```

### [10. Command](https://docs.ansible.com/ansible/latest/modules/command_module.html)

```
- name: return motd to registered var
  command: cat /etc/motd
  register: mymotd
```

### [11. stat – Retrieve file or file system status](https://docs.ansible.com/ansible/latest/modules/stat_module.html)

```
- name: Ansible check versions {{ ruby_version }} exists.
  stat:
    path: "{{ rbenv_root }}/versions/{{ ruby_version }}"
  register: dir_version

- name: Install ruby {{ ruby_version }}
  command: sudo -iu {{ user }} rbenv install {{ ruby_version }}
  when: dir_version.stat.exists == False 
  
================================================================================ 
 
- stat:
    path: /path/to/something
  register: sym

- debug:
    msg: "islnk isn't defined (path doesn't exist)"
  when: sym.stat.islnk is not defined
```

### [12. How to set and use sudo password for Ansible Vault](https://www.cyberciti.biz/faq/how-to-set-and-use-sudo-password-for-ansible-vault/)

* First update your inventory file as follows:

```
[cluster:vars]
k_ver="linux-image-4.13.0-26-generic"
ansible_user=vivek  # ssh login user
ansible_become=yes  # use sudo 
ansible_become_method=sudo 
ansible_become_pass='{{ my_cluser_sudo_pass }}'
 
[cluster]
www1
www2
www3
db1
db2
cache1
cache2

```

* Next create a new encrypted data file named password.yml, run the following command:

```
ansible-vault create passwd.yml
```

* Set the password for vault. After providing a password, the tool will start whatever editor you have defined with $EDITOR. Append the following

```
my_cluser_sudo_pass: your_sudo_password_for_remote_servers
```

* Save and close the file in vi/vim. Finally run playbook as follows:

```
ansible-playbook -i inventory --ask-vault-pass --extra-vars '@passwd.yml' my.yml
```

* How to edit my encrypted file again

```
ansible-vault edit passwd.yml
```

* How to change password for my encrypted file

```
ansible-vault rekey passwd.yml
```

#### Summary
In short use following options for the ansible-playbook command with vault or without vault file:

* `-i inventory` : Set path to your inventory file.
* `--ask-vault-pass` : Ask for vault password
* `--extra-vars '@passwd.yml'` – Set extra variable. In this case set path to vault file named passwd.yml.
* `--ask-become-pass` : Ask for sudo password


## Directiory Layout

[Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

```
production                # inventory file for production servers
staging                   # inventory file for staging environment

group_vars/
	group1.yml             # here we assign variables to particular groups
	group2.yml
host_vars/
   hostname1.yml          # here we assign variables to particular systems
   hostname2.yml

library/                  # if any custom modules, put them here (optional)
module_utils/             # if any custom module_utils to support modules, put them here (optional)
filter_plugins/           # if any custom filter plugins, put them here (optional)

site.yml                  # master playbook
webservers.yml            # playbook for webserver tier
dbservers.yml             # playbook for dbserver tier

roles/
    common/          # this hierarchy represents a "role"
      tasks/         #
        main.yml     #  <-- tasks file can include smaller files if warranted
	handlers/         #
	    main.yml      #  <-- handlers file
	templates/        #  <-- files for use with the template resource
	    ntp.conf.j2   #  <------- templates end in .j2
	files/            #
	    bar.txt       #  <-- files for use with the copy resource
	    foo.sh        #  <-- script files for use with the script resource
	vars/             #
	    main.yml      #  <-- variables associated with this role
	defaults/         #
	    main.yml      #  <-- default lower priority variables for this role
	meta/             #
	    main.yml      #  <-- role dependencies
	library/          # roles can also include custom modules
	module_utils/     # roles can also include custom module_utils
	lookup_plugins/   # or other types of plugins, like lookup in this case

    webtier/              # same kind of structure as "common" was above, done for the webtier role
    monitoring/           # ""
    fooapp/               # ""
Alternative Directory Layout
```
