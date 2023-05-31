# my-ansible-repo
This is my ansible notes from "Ansible for the DevOps Beginners &amp; System Admins"
#####Install ansible server #####

## Launch EC2 instance with AMI 2 linux
## Open ssh with secrity key
## Change the hostname at /etc/hostname and restart the node with init 6
or 
echo "ansible-server" > /etc/hostname
init 6
## Add ansadmin user
useradd ansadmin
passwd ansadmin
ansadmin

##Add ansadmin user to sudoers
echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
or we run visudo and add 
ansadmin ALL=(ALL)       NOPASSWD: ALL
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
ansadmin ALL=(ALL)       NOPASSWD: ALL

## Allow ansadmin with password authentication
sed -ie 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
service sshd reload // restart sshd service

## switch to ansadmin
## Generate SSH key
ssh-keygen
## *ssh keys will be kept at target server which is now generated at /home/ansadmin/.ssh/id_rsa & /home/ansadmin/.ssh/id_rsa.pub

## Install ansible in amazon linux2
[ansadmin@ansible-server ~]$ amazon-linux-extras | grep ansible
  0  ansible2                 available    \
[ansadmin@ansible-server ~]$ sudo amazon-linux-extras install ansible2

## There is already python installed with amazon linux2 so no need to install python here in EC2.
[ansadmin@ansible-server ~]$ python --version
Python 2.7.18
[ansadmin@ansible-server ~]$ ansible --version
ansible 2.9.23
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ansadmin/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.18 (default, Feb 28 2023, 02:51:06) [GCC 7.3.1 20180712 (Red Hat 7.3.1-15)]
[ansadmin@ansible-server ~]


##### 8. Setup RHEL as Ansible Managed node #####
useradd ansadmin
passwd ansadmin
## Allow ansadmin with password authentication
sed -ie 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
service sshd reload

##### 9. Add RHEL managed node to Ansible #####
Login to ansible-control-nodee

sudo vi /etc/ansible/hosts
172.31.35.174 // internal IP of rhel managed node
ssh-copy-id 172.31.35.174
Password: ansadmin
ansible all -m ping // ad hoc command where ping is the module, -m for module


##### 10. Ansible Ad-hoc commands #####
## command module ##
ansible all -m command -a "uptime" // whatever the command we want to run, here uptime command we want to run, a for attribute

## stat module ##
ansible all -m stat -a "path=/etc/hosts" // gives information abouth hosts file

## yum module ##
ansible all -m yum -a "name=git"
error:  "msg": "This command has to be run under the root user.",
We need to run this below command with "-b" for become root
ansible all -m yum -a "name=git" -b

## user module ##
ansible all -m user -a "name=john" -b

## setup module ##
ansible all -m setup // doesn't need any attribute



##### 11. Ansible Inventory #####
# Hosts information can be defined in following ways.
> Default Location: /etc/ansible/hosts // modifications on default inventory files is not recommended if there are multiple users of ansible server 
> Use -i option: ansible -i my_hosts
## If I create a hosts file adding the rhel managed server IP under /home/ansadmin/, I can execute this below command
ansible all -m ping -i hosts // hosts of current location
> Defined in ansible.cfg file


##### 12. Ansible configuration file - ansible.cfg #####
[defaults]

# some basic default values...

#inventory      = /etc/ansible/hosts // default inventory
#library        = /usr/share/my_modules/
#remote_tmp     = $HOME/.ansible/tmp
#local_tmp      = $HOME/.ansible/tmp
#forks          = 5 // Max 5 numbers of client system can be run from this ansible server
#poll_interval  = 15
#sudo_user      = root
#ask_sudo_pass = True
#ask_pass      = True // for login into the client system, password is needed or not
#transport      = smart
#remote_port    = 22
#module_lang    = C
#module_set_locale = False


# default module name for /usr/bin/ansible
#module_name = command // it treats command as default module

[privilege_escalation]
#become=True // true means execute as root user
#become_method=sudo
#become_user=root
#become_ask_pass=False // while becoming a root, do you need password or not


ansible all -a "uptime" // by default it is taking command module
## we can create our own customed ansible.cfg file in our current working directory


##### 13. Ansible Modules #####
https://docs.ansible.com/ansible/2.9/user_guide/modules_intro.html
## to check the installed modules in ansible server
ansible-doc -l
ansible-doc -l | wc // to module count


##### 14. Create your first Ansible playbook #####
https://docs.ansible.com/ansible/2.9/user_guide/playbooks_intro.html
cd /opt/
sudo mkdir ansible
sudo chown ansadmin:ansadmin ansible/
[ansadmin@ansible-control-node opt]$ sudo vi create-user.yml
[ansadmin@ansible-control-node opt]$ cat create-user.yml 
---
- hosts: rhel
  become: true
  tasks:
  - user: name=john
ansible-playbook create-user.yml --check


##### 15. Setup additional managed nodes #####
# Launch 2 EC2 instances with amazon linux and ubuntu
# changed hostname from root access
# add user ansadmin
useradd ansadmin
passwd ansadmin
# add to sudoers
visudo
ansadmin ALL=(ALL)       NOPASSWD: ALL
# add password authentication
vi /etc/ssh/sshd_config
PasswordAuthentication yes
#PermitEmptyPasswords no
#PasswordAuthentication no
service sshd reload

for ubuntu
**** for ubuntu managed node, username would be ubuntu
useradd ansadmin -m -d /home/ansadmin
# visudo // ubuntu visudo is stupid, be careful
ansadmin ALL=(ALL)       NOPASSWD: ALL

## Add hosts in ansible-control-node
cd /opt/ansible

## copy ssh keys to managed servers from control node to managed nodes
ssh-copy-id 172.31.5.46 // amazon-managed-node internal IP
yes
ansadmin // first time asks for password
# same for ubuntu-managed-node
ssh-copy-id 172.31.2.221 // ubuntu-managed-node internal IP

# Once ssh keys are copied to managed node from control node, you shall be able to login to managed nodes without password authentication
ssh 172.31.5.46
ssh 172.31.2.221
# check the ping status
ansible all -m ping -i hosts


##### 16. Run a Ansible playbook #####
[ansadmin@ansible-control-node ansible]$ cat create-user.yml 
---
- name: this playbook is to create user on managed nodes
  hosts: all
  become: true
  tasks:
  - name: creating user jim
    user: 
     name: jim
  - name: creating user dic
    user:
     name: dic
[ansadmin@ansible-control-node ansible]$ 
# I have added two task to add two different users


##### 17. Yum Module - Install packages #####
[ansadmin@ansible-control-node ansible]$ cat install-packages.yml 
---
- name: this playbook installs packages
  become: true
  hosts: webservers
  tasks:
  - name: installing yum
    yum: 
     name: git
     state: installed
[ansadmin@ansible-control-node ansible]$ 

[ansadmin@ansible-control-node ansible]$ cat hosts 
[webservers]
172.31.35.174
172.31.5.46

[appservers]
172.31.5.46

[dbservers]
172.31.2.221

ansible-playbook -i hosts install-packages.yml // run the playbook to install package

## if hash out the become from playbook yml, the above command will not run, so we can use below command with "-b" represents become root
ansible-playbook install-packages.yml -i hosts -b


##### 18. File Module - create/remove a file/directory #####
# Another way to check hosts with ansible command
ansible all -i hosts --list-hosts
[ansadmin@ansible-control-node ansible]$ cat create-file.yml 
---
- name: this playbook creates file
  hosts: all
  become: true
  tasks:
  - name: creatin file
    file:
      path: /home/ansadmin/dir2
      state: directory
      owner: ansadmin
      group: ansadmin
[ansadmin@ansible-control-node ansible]$ 

## if we hashout become: true by default also the owner and group will be "ansadmin"
## [ansadmin@ansible-control-node ansible]$ cat create-file.yml 
---
- name: this playbook creates file
  hosts: all
  become: true
  tasks:
  - name: creatin file
    file:
      path: /home/ansadmin/dir2
      state: absent
      owner: ansadmin
      group: ansadmin
[ansadmin@ansible-control-node ansible]$ 
## absent to delete created file or directory
## and state to be touch for creating a file


##### 19. Copy Module - Copy a file on to managed nodes #####

vi index.html

[ansadmin@ansible-control-node ansible]$ cat copy-file.yml 
---
- name: this playbook copies file from control to managed nodes
  hosts: all
  become: true
  tasks:
  - name: copying file
    copy:
      src: /opt/ansible/index.html
      dest: /home/ansadmin
      owner: ansadmin
      group: ansadmin
      mode: 0700

## to check if there is any syntax error in playbook file
ansible-playbook -i hosts copy-file.yml --syntax-check
## to check whether playbook will run or not
ansible-playbook -i hosts copy-file.yml --check


##### 20. Install Apache on RHEL #####
[ansadmin@ansible-control-node ansible]$ cat install-httpd.yml 
---
- name: this playbook installs httpd
  become: true
  hosts: webservers
  tasks:
  - name: installing package
    yum: 
     name: httpd
     state: installed
  - name: start httpd service
    service:
     name: httpd
     state: started

# added ansible.cfg under /opt/ansible/ not to use "-i hosts" in every ansible-playbook command
vi ansible.cfg
[defaults]

# some basic default values...

inventory      = /opt/ansible/hosts

# in new asible.cfg /opt/ansible/ provide current working directory where hosts are added, /opt/ansible/hosts

# check the httpd packages are installed or not
rpm -qa | grep httpd
ps -ef | grep httpd

ansible-playbook install-httpd.yml

## now uninstall httpd package, first stop the service then remove
[ansadmin@ansible-control-node ansible]$ cat uninstall-httpd.yml 
---
- name: this playbook uninstalls httpd
  become: true
  hosts: webservers
  tasks:
  - name: stop httpd service
    service:
     name: httpd
     state: stopped
  - name: uninstalling package
    yum:
     name: httpd
     state: removed


ansible-playbook uninstall-httpd.yml


##### 21. Install Apache on Ubuntu #####
ps -ef | grep apache2 // check apache processes on ubuntu managed node

[ansadmin@ansible-control-node ansible]$ ansible-playbook install-apache2.yml --syntax-check

playbook: install-apache2.yml
[ansadmin@ansible-control-node ansible]$ cat install-apache2.yml 
---
- name: install apache2 on ubuntu managed node
  hosts: dbservers
  become: true
  tasks:
    - name: install package
      apt:
        name: apache2
        state: present
    - name: start apache2
      service:
        name: apache2
        state: started

ansible-playbook install-apache2.yml 

## check in ubuntu-managed-node if apache2 installation and service start have been successful
ps -ef | grep apache2

Expected outcome:
$ ps -ef | grep apache2
root        1603       1  0 10:15 ?        00:00:00 /usr/sbin/apache2 -k start
www-data    1605    1603  0 10:15 ?        00:00:00 /usr/sbin/apache2 -k start
www-data    1606    1603  0 10:15 ?        00:00:00 /usr/sbin/apache2 -k start
www-data    1726       1  0 10:15 ?        00:00:00 /usr/bin/htcacheclean -d 120 -p /var/cache/apache2/mod_cache_disk -l 300M -n
ansadmin    1947     852  0 10:15 pts/0    00:00:00 grep apache2

We can check if gui is accesible vi amazon public IPV4 DNS 
ec2-13-232-223-89.ap-south-1.compute.amazonaws.com


##### 22. Notify and Handlers in a playbook #####
[ansadmin@ansible-control-node ansible]$ cat install-httpd.yml 
---
- name: this playbook installs httpd
  become: true
  hosts: webservers
  tasks:
  - name: installing package
    yum: 
     name: httpd
     state: installed
    notify: start apache

  handlers:
  - name: start apache
    service:
     name: httpd
     state: started

ansible-playbook install-httpd.yml --syntax-check

playbook: install-httpd.yml

ansible-playbook install-httpd.yml

PLAY [this playbook installs httpd] *********************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
[DEPRECATION WARNING]: Distribution rhel 9.2 on host 172.31.35.174 should use /usr/libexec/platform-python, but is using 
/usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default to using the 
discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be 
removed in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [172.31.35.174]
[WARNING]: Platform linux on host 172.31.5.46 is using the discovered Python interpreter at /usr/bin/python, but future
installation of another Python interpreter could change this. See
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
ok: [172.31.5.46]

TASK [installing package] *******************************************************************************************************
changed: [172.31.35.174]
changed: [172.31.5.46]

RUNNING HANDLER [start apache] **************************************************************************************************
changed: [172.31.35.174]
changed: [172.31.5.46]

PLAY RECAP **********************************************************************************************************************
172.31.35.174              : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.5.46                : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

# manual httpd service start
sudo systemctl start httpd

# for ubuntu
sudo service apache2 restart
service apache2 status
service apache2 stop
apt remove apache2

##### 23. How gather facts works #####

# gathering facts task is used get status, information about the remote node.
# to avoid gather facts from the playbook, we need to add "gather_facts: no" in playbook. Means don't gather the facts of remote system.
# gathering facts comes under 1 task. 
[ansadmin@ansible-control-node ansible]$ cat create-file.yml 
---
- name: this playbook creates file
  hosts: all
  become: true
    #gather_facts: no
  tasks:
  - name: creating file
    file:
      path: /home/ansadmin/dir2
      state: directory
      owner: ansadmin
      group: ansadmin

##### 24. How When condition works #####
# to check remote node's details
ansible all -m setup
[ansadmin@ansible-control-node ansible]$ cat install-httpd-apache.yml
---
- name: this playbook installs httpd
  hosts: all
  become: true
  tasks:
  - name: installing package
    yum: 
     name: httpd
     state: installed
    when: ansible_os_family == "Redhat"


  - name: start apache
    service:
     name: httpd
     state: started
    when: ansible_os_family == "Redhat"

  - name: install apache2
    apt:
     name: apache2
     state: present
    when: ansible_os_family == "Debian"

  - name: start apache2
    service:
     name: apache2
     state: started
    when: ansible_os_family == "Debian"

## removing apache2 from ubuntu / checking httpd/apache status in ubuntuu
service apache2 stop
apt remove apache2
service apache2 status


##### 25. Uninstall Apache using when condition #####
---
- name: this playbook uninstall httpd
  hosts: all 
  become: true
  tasks:
  - name: stop httpd service
    service:
      name: httpd
      state: stopped
    when: ansible_os_family == "RedHat" 

  - name: uninstall httpd
    yum: 
      name: httpd
      state: removed
    when: ansible_os_family == "RedHat"

  - name: stop apache2 services
    service: 
      name: apache2
      state: stopped
    when: ansible_os_family == "Debian"

  - name: uninstall apache2
    apt: 
      name: apache2
      state: absent 
    when: ansible_os_family == "Debian"

ansible-playbook new-uninstall-httpd-apache2.yml


##### 26. Adding copy task to Apache playbook #####
[ansadmin@ansible-control-node ansible]$ cat install-copy-index.yml 
---

- name: this playbook install httpd and apache2

  hosts: all

  become: true

  tasks:

  - name: install httpd package

    yum:

      name: httpd

      state: installed

    when: ansible_os_family == "RedHat"

  - name: start httpd service

    service :

      name: httpd

      state: started

    when: ansible_os_family == "RedHat"

  - name: install apache2

    apt:

      name: apache2

      state: present

    when: ansible_os_family == "Debian"



  - name: start apache2

    service:

      name: apache2

      state: started

    when: ansible_os_family == "Debian"

  - name: copy index.html

    copy:

      src: /opt/ansible/index.html

      dest: /var/www/html

      mode: 0666

ansible-playbook install-copy-index.yml


#### 27. Lists and With_items ####
---
- name: this playbook installs packages
  become: true
  hosts: webservers
  tasks:
  - name: installing yum
    yum: 
     name: ['git', 'make', 'gcc', 'wget', 'gzip', 'telnet']
     state: installed

ansible-playbook package-list-install.yml


##### 28. Ansible variables #####
Process 1
---
- name: this playbook is to create user on managed nodes
  hosts: all
  become: true
  vars:
    user: lopa
  tasks:
  - name: creating user {{ user }}
    user: 
     name: "{{ user }}"

When the sentence starts with variable "" is mandatory
For example: "{{ user }}" is being added

Process 2: get variable from extarnal source or extarnal file
## create a variable file, the extension could be anything, here we have used yml
vi user.yml
user: tom
## now modifications on create-variable-user.yml playbook file
[ansadmin@ansible-control-node ansible]$ cat create-variable-user.yml 
---
- name: this playbook is to create user on managed nodes
  hosts: all
  become: true
  vars_files:
    - user.yml

  tasks:
  - name: creating user {{ user }}
    user: 
     name: "{{ user }}"

ansible-playbook create-variable-user.yml

Process 3: high priorty, executing through the command line will be highest priority
ansible-playbook create-variable-user.yml --extra-vars "user=sonia"

or

ansible-playbook create-variable-user.yml -e "user=sonia"


#### 29. Convert shell commands into a Ansible playbook ####
https://github.com/yankils/ansible/blob/master/tomcat_installation.md
## setup tomcat
[ansadmin@ansible-control-node ansible]$ cat setup-tomcat.yml 
---
- name: setup tomcat
  hosts: all
  become: true
  tasks:
  - name: install java
    yum:
      name: java
      state: installed
    when: ansible_os_family == "RedHat"
  
  - name: install java on ubuntu
    apt:
      name: default-jdk
      state: present
    when: ansible_os_family == "Debian"

  - name: download tomcat packages
    get_url:
      url: https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
      dest: /opt

  - name: untar apache packages
    unarchive:
      src: /opt/apache-tomcat-9.0.75.tar.gz
      dest: /opt
      remote_src: yes

  - name: add execution permission on startup.sh file
    file:
      path: /opt/apache-tomcat-9.0.75/bin/startup.sh
      mode: 0777

  - name: start tomcat services
    shell: nohup ./startup.sh
    args:
      chdir: /opt/apache-tomcat-9.0.75/bin


ansible-playbook setup-tomcat.yml


#### 30. Using tags in a playbook #####
# tags are useful to segregate specific task from playbook

[ansadmin@ansible-control-node ansible]$ cp install-copy-index.yml use-of-tag.yml
[ansadmin@ansible-control-node ansible]$ vi use-of-tag.yml 
[ansadmin@ansible-control-node ansible]$ cat use-of-tag.yml 
---

- name: this playbook install httpd and apache2

  hosts: all

  become: true

  tasks:

  - name: install httpd package

    yum:

      name: httpd

      state: installed

    when: ansible_os_family == "RedHat"
    tags: install_apache

  - name: start httpd service

    service :

      name: httpd

      state: started

    when: ansible_os_family == "RedHat"
    tags: start_apache

  - name: install apache2

    apt:

      name: apache2

      state: present

    when: ansible_os_family == "Debian"
    tags: install_apache


  - name: start apache2

    service:

      name: apache2

      state: started

    when: ansible_os_family == "Debian"
    tags: start_apache

  - name: copy index.html

    copy:

      src: /opt/ansible/index.html

      dest: /var/www/html

      mode: 0666

ansible-playbook use-of-tag.yml --tags "install_apache" // only do task which has "install_apache" tag
ansible-playbook use-of-tag.yml --tags "start_apache" // only do task which has "start_apache" tag


##### 31. Error handling in a playbook #####
# to generate the error, we have a some line in RHEL managed node's sudo vi /etc/httpd/conf/httpd.conf, like testing on top
# now when we try to restart the httpd service in RHEL managed node, it will fail

[ansadmin@rhel-managed-node ~]$ sudo service httpd restart
Redirecting to /bin/systemctl restart httpd.service
Job for httpd.service failed because the control process exited with error code.
See "systemctl status httpd.service" and "journalctl -xeu httpd.service" for details.
[ansadmin@rhel-managed-node ~]$

# Now, lets go to control node and try to execute ansible-playbook command. 
# it shall throw error like below and rest of task will not be executed due to this
TASK [start httpd service] ******************************************************************************************************
skipping: [172.31.2.221]
ok: [172.31.5.46]
fatal: [172.31.35.174]: FAILED! => {"changed": false, "msg": "Unable to start service httpd: 
Job for httpd.service failed because the control process exited with error code.\nSee \"systemctl status httpd.service\" and \"journalctl 
-xeu httpd.service\" for details.\n"}

## So, can ignore this error which is less impacting and follow along with rest of the task.
## We can add "ignore_errors: yes" in playbook file
[ansadmin@ansible-control-node ansible]$ cat ignore_error-yes.yml 
---

- name: this playbook install httpd and apache2

  hosts: all

  become: true

  tasks:

  - name: install httpd package

    yum:

      name: httpd

      state: installed

    when: ansible_os_family == "RedHat"

  - name: start httpd service

    service :

      name: httpd

      state: started

    when: ansible_os_family == "RedHat"
    ignore_errors: yes

  - name: install apache2

    apt:

      name: apache2

      state: present

    when: ansible_os_family == "Debian"



  - name: start apache2

    service:

      name: apache2

      state: started

    when: ansible_os_family == "Debian"

  - name: copy index.html

    copy:

      src: /opt/ansible/index.html

      dest: /var/www/html

      mode: 0666



## after this , we will have below output
ansible-playbook ignore_error-yes.yml 
TASK [start httpd service] ******************************************************************************************************
skipping: [172.31.2.221]
ok: [172.31.5.46]
fatal: [172.31.35.174]: FAILED! => {"changed": false, "msg": "Unable to start service httpd: 
Job for httpd.service failed because the control process exited with error code.\nSee \"systemctl status httpd.service\" and \"journalctl 
-xeu httpd.service\" for details.\n"}
...ignoring

## we can see that is it ignoring.


##### 32. Ansible vault introduction #####

# to create
ansible-vault creat vault-pass.yml
Password: root

# to view
ansible-vault view vault-pass.yml 
Vault password: root
this is a new encrypted file // content of the file

# To edit 
ansible-vault edit vault-pass.yml
Vault password: root

# to decrypt
ansible-vault decrypt vault-pass.yml

# To encrypt 
ansible-vault encrypt vault-pass.yml


##### 33. Using ansible vault with git #####
# create repo in github
https://github.com/victorkeshari/vault
git clone https://github.com/victorkeshari/vault.git

[ansadmin@ansible-control-node ansible]$ cat ansible-vault.yml 
---
- name: ansible playbook to test ansible vault
  hosts: all
  become: true
  tasks:
    - name: clone a repo
      git:
        repo: https://github.com/victorkeshari/vault
        dest: /home/ansadmin/test-vault

ansible-playbook ansible-vault.yml // this will clone the repo from github to managed nodes

ansible-playbook ansible-vault.yml --ask-vault-pass // If there is any ecrypted vault file which is encrypted with password, this will ask for password
if the vault file is needed to do run the task on playbook



##### 34. Ansible roles Introduction #####
## to create role
ansible-galaxy init setup-apache // init will initiate role with the name of setup-apache, then we can see that role structure.
[ansadmin@ansible-control-node ansible]$ cd setup-apache/
[ansadmin@ansible-control-node setup-apache]$ tree
.
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

8 directories, 8 files


##### 35. Adding more tasks to playbook - lab #####



##### 36. Convert a playbook into a role - lab #####
ansible-playbook setup-apache.yml --extra-vars "port=8088" // port=8080 has highest priority

