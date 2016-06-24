#### psgsansiblewindows
######5
create windows vm
```
config.vm.define "webserver02" do |web|
  web.vm.box= ""
  web.vm.hostname= ""
  web.vm.communicator="winrm"
  web.winrm.username=""
  web.winrm.passwors=""
  web.vm.network "private_network", ip: "11.1.1.1"
  web.vm.provider "virtualbox" do |vb|
    vb.memory = 2048
    vb.cpus=2
  end
end
```
######6
```
Test-WSMan 192.168.57.3
```
######9 installing ansible on control server
this time we use pip
```
pip install markupsafe xmltodict pywinrm ansible
```
then
```
mkdir test
cd test
vim inventory.yml
```
edit
```
---
[web]
192.168.57.3
```

in test/, create folder
```
mkdir group_vars
vim group_vars/web.yml
```
edit
```
ansible_user: ke
ansible_password: root
ansible_port: 5985
ansible_connection: winrm
ansible_winrm_cert_validation: ignore
```
test
```
ansible web -i inventory.yml -m win_ping
```
######10 modules playbooks roles
```
cd test
anaible web -i inventory.yml -m setup
```
and
```
vim ansible.cfg
```
edit
```
[defaults]
inventory = ~/path/inventory.yml
```
test other commands: run raw command `dir`
```
ansible  web -m raw -a "dir"
```
service
```
ansible  web -m win_service -a "name=spooler"
ansible  web -m win_service -a "name=spooler state=stopped"
ansible  web -m win_service -a "name=spooler state=started"
```
install something
```
ansible  web -m win_feature -a "name=Telnet-Client state=persent"
```
######11 modules with playbooks
```
---
- hosts: web
  tasks:
  - name: Install IIS
    win_feature:
      name: Web-Server
      state: present
    when: ansible_os_family == "Windows"  /"Linux"
  - name: deploy
    template:
      src: start.j2
      desc: c:\inetpub\wwwroot\iisstart.htm
    
```
######12 creating roles
```
mkdir -p roles/webserver
cd roles/webserver
mkdir templates
mkdir tasks
touch templates/iisstart.j2
```
```
---
- hosts: web
  roles:
    - webserver
```
######13
bucause it is multiple machine, we cannot use vagrant ssh. use
```
ssh vagrant@ip
```
instead.  

create another var file in group_vars
```
mv group_vars/web.yml group_vars/all.yml
```
ansible vault encrypt
```
ansible-vault encrypt group_vars/all.yml
```
use encrypted files:
```
ansible db -m win_ping --ask-vault-pass
```
######14 finishing the globomantics
```
mkdir -p roles/common/tasks
vim roles/common/tasks/main.yml
```
edit
```
---
- name: install note++
  win_chocolatey:
    name: notepadplusplus
    state: present
- name: create local admin
  win_user:
    name: localadmin
    password: "123"
    groups: ["Administrators"]
```


