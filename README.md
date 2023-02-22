>>## Project 11 and 12 -ansible-config-mnt

>###  Project Description
- Automation of previous project 7-10  
- Adding a bastion host (Jump server) to server as a entry point for client users
- Project Diagram 
![](Prj%2011-PNGs/Project%2011%20Diagram.png)

>### Project Steps broken down :

- installing Ansible and configuring build Job on server
    sudo apt install ansible 
- created freestyle project and configured webhook to trigger ansible build
- configured post-build job to save all files.
- attached an elastic Ip to the ansible server so the public ip persists upon reboot
- installed VScode and connected it to my GitHub repository for the project 
- setting up ansible directories (inventory, playbooks, common, prod, dev. uat)
![](Prj%2011-PNGs/4.%20Creating%20Directories%20in%20Vscode.png)

- setting up SSH-Agent as ansible uses ssh to reach target servers 
    - eval `ssh-agent -s`
    - ssh-add (path-to-private-key)
- confirming the key added successfully 
    - ssh-add -l 
- SSH using Jenknins ansible server using SSH-agent
    - ssh -A ubuntu@public-ip
![](Prj%2011-PNGs/6.%20SSH%20using%20Agent.png)

- updating inventory/dev.yml directory with host Ip addresses
    [nfs]
    - (NFS-Server-Private-IP-Address) ansible_ssh_user='ec2-user'

- [webservers]
    - (Web-Server1-Private-IP-Address) ansible_ssh_user='ec2-user'
    - (Web-Server2-Private-IP-Address) ansible_ssh_user='ec2-user'

    [db]
    - (Database-Private-IP-Address) ansible_ssh_user='ec2-user' 

    [lb]
    - (Load-Balancer-Private-IP-Address) ansible_ssh_user='ubuntu'
![](Prj%2011-PNGs/7.%20Editing%20inventory-dev%20directory.png)

- Updating common.yml file
    ---
    - name: update web, nfs and db servers
        - hosts: webservers, nfs, db
        - remote_user: ec2-user
        - become: yes
        - become_user: root
    - tasks:
        - name: ensure wireshark is at the latest version
        - yum:
          - name: wireshark
          - state: latest

- name: update LB server
   - hosts: lb
   - remote_user: ubuntu
   - become: yes
   - become_user: root
   - tasks:
        - name: Update apt repo
        - apt: 
          - update_cache: yes

    - name: ensure wireshark is at the latest version
        - apt:
          - name: wireshark
          - state: latest
![](Prj%2011-PNGs/8.%20Updating%20Common.yml%20directory.png)

- Pushing changes to GitHub branch 
![](Prj%2011-PNGs/9.%20Pushing%20changes%20to%20branch%20.png)

- Pushing a build job via webhook
![](Prj%2011-PNGs/10.%20Pushed%20to%20Jenkins%20via%20Webhook.png)

- Executing ansible playbook 
    - ansible-playbook -i inventory/dev.yml playbooks/common.yml
![](Prj%2011-PNGs/11.%20Successful%20SSH%20from%20controller%20server.png)

- Confirming wireshark installed on target servers after running ansible playbook
![](Prj%2011-PNGs/12.%20Checking%20wireshark%20version%20on%20NFS-server.png) 



>>### Project 12 Ansible Refactoring and Static Assignments (Imports and Roles)

># Project Description
- This project was a continuation of project 11, while making some improvements to the code (Refactoring) using imports function of ansible and roles.

>## Project Breakdown

 1. Installed the copy artifact plugin on jenkins console and created directory on server in home directory 
    - sudo mkdir /home/ubuntu/ansible-config-artifact
![](Prj%2011-PNGs/Prj-12-PNGs/1.%20Creating%20ansible-config-artifact%20directory.png)

![](Prj%2011-PNGs/Prj-12-PNGs/2.%20Installing%20copy%20artifact%20plugin.png)

2. Creating a new freestyle project 
![](Prj%2011-PNGs/Prj-12-PNGs/3.%20Creating%20new%20freestyle%20project.png)

3. Configuring Jenkins to save artifacts to created directory 
![](Prj%2011-PNGs/Prj-12-PNGs/4.%20Configuring%20builds%20to%20keep%20.png)

![](Prj%2011-PNGs/Prj-12-PNGs/5.%20configuring%20build%20triggers%20to%20ansible%20project.png)

4. Refactoring Ansible code 
    - created site.yml file and updated it with the import command 
    - 
    ---
     hosts: all
     import_playbook: ../static-assignments/common.yml
    - configured a task to remove wireshark from target servers if already installed by creating static-assignments directory and creating common-del.yml file and updating it with the code :
![](Prj%2011-PNGs/Prj-12-PNGs/12.%20using%20import_playbook%20command.png)

    - ---
  - name: update web, nfs and db servers
     -   hosts: webservers, nfs, db
     -   remote_user: ec2-user
     -   become: yes
     -   become_user: root
     -   tasks:
  - name: delete wireshark
    yum:
     -   name: wireshark
     -   state: removed

- name: update LB server
     -   hosts: lb
     -   remote_user: ubuntu
     -   become: yes
     -   become_user: root
  tasks:
  - name: delete wireshark
    apt:
     -  name: wireshark-qt
     -  state: absent
     -  autoremove: yes
     -  purge: yes
     -  autoclean: yes
5. Running the plyabook using :
    - ansible-playbook -i inventory/dev.yml playbooks/site.yaml
![](Prj%2011-PNGs/Prj-12-PNGs/11.%20Wireshark%20not%20present%20on%20server.png)

6. Launched two servers and created roles 
![](Prj%2011-PNGs/Prj-12-PNGs/13.%20creating%20roles%20directory.png)

7. Updating uat.yml file with hosts information 
![](Prj%2011-PNGs/Prj-12-PNGs/13.%20Updating%20inventory%20file%20with%20webserver%20hosts%20.png)

8. uncommneting roles path in **/etc/ansible/ansible.cfg** and pointing it to created directory **/home/ubuntu/ansible-config-mgt/roles**
![](Prj%2011-PNGs/Prj-12-PNGs/14.%20Uncommenting%20roles_path%20in%20ansible%20config%20file(1).png)

9. configured the tasks to run on the webserver in the main.yml file and pointed to the GitHub application 
![](Prj%2011-PNGs/Prj-12-PNGs/15.%20updating%20main.yml%20file%20to%20perform%20tasks%20.png)

10. referenced the role created by creating an assignment for the webservers :

        - ---
        - hosts: uat-webservers
          roles:
            - webserver
11. saved configurations and ran the command :

    - sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml

![](Prj%2011-PNGs/Prj-12-PNGs/16.%20site%20running%20on%20webserver.png)

















