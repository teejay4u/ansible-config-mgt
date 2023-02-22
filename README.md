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
![]()













