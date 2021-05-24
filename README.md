## Project 11 - Ansible Configuration Management - Automate Project 7 to 10


### Ansible Client as a Jump Server (Bastion Host)

A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. In the current architecture, the webservers are inside a secured network which cannot be reached directly from the Internet. This implies that even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server - it provides better security and reduces attack surface.

On the diagram below the Virtual Private Network (VPC) is divided into two subnets - Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.

   ![image](https://user-images.githubusercontent.com/78841364/118401574-32b1cd80-b634-11eb-8ae5-b16730057fae.png)

### Task
Install and configure Ansible client to act as a Jump Server/Bastion Host

Create a simple Ansible playbook to automate servers configuration

### Step 1 - Install and configure Ansible on EC2 Instance

1. Update Name tag on existing Jenkins EC2 Instance to Jenkins-Ansible for running playbooks.

2. In GitHub account create a new repository and name it ansible-config-mgt.

3. Install Ansible
   
       sudo apt update

       sudo apt install ansible

  Check your Ansible version by running ansible --version

 ![image](https://user-images.githubusercontent.com/78841364/118402996-62fc6a80-b63a-11eb-9464-541909f6b750.png)


4. Configure Jenkins build job to save your repository content every time change is made.

   ![image](https://user-images.githubusercontent.com/78841364/118403463-6f81c280-b63c-11eb-8267-809dbbb12168.png)
   
   - Install Jenkins
   
   - Perform initial Jenkins setup
  
  - From the browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
  
  - Retrieve password from server using sudo cat /var/lib/jenkins/secrets/initialAdminPassword.
      

5. Create a new Freestyle project ansible in Jenkins and point it to ‘ansible-config-mgt’ repository.

6. Configure Webhook in GitHub and set webhook to trigger ansible build.

7. Configure a Post-build job to save all (**) files.

8. Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in   
   following folder
 
            ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/

    New setup will be as shown below:

  ![image](https://user-images.githubusercontent.com/78841364/118416923-e5f3e400-b67f-11eb-9134-758dce32336e.png)
  
Tip Every time you stop/start your Jenkins-Ansible server - you have to reconfigure GitHub webhook to a new IP address, in order to avoid it, it makes sense to allocate an Elastic IP to your Jenkins-Ansible server (you have done it before to your LB server in Project 10). Note that Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once you terminate your EC2 Instance.

### Step 2 - Prepare development environment using Visual Studio Code

1. Download and install VSCode

2. Install the Remote SSH Extension by searching for the extension “Remote-SSH” in the VSCode marketplace.

3. Open The Configuration file and choose the config file corresponding to the current user (typically the /.ssh path)
   
   Enter the following details
   
   Host - enter any name of choice to identify the server.
  
   HostName is the server’s host or IP
   
   User is the Ubuntu username.
   
   IdentityFile is the path to the private key.

### Step 3 - Begin Ansible Development

1. In ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.

2. Checkout the newly created feature branch to local machine and start building code and directory structure

        git branch

       * feature01-prj11-layout
         
         master

3. Create a directory and name it playbooks (for storage all playbook files).

4. Create a directory and name it inventory (for keeping the hosts organised).

5. Within the playbooks folder, create the first playbook, and name it common.yml

6. Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

    
    ![image](https://user-images.githubusercontent.com/78841364/118504070-34919480-b6f9-11eb-80fe-3032367f3f3f.png)


### Step 4 - Set up an Ansible Inventory

1. Edit inventory/dev.yml as follows:

[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ubuntu' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'

Save the inventory structure start configuring the development servers

Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host - for this we need to copy the private (.pem) key to the server and also change permissions to the private key chmod 400 key.pem so that EC2 will accept the key. 

2. Change permissions to private key
       
         chmod 400 key.pem
         
4. Import key into ssh-agent:
   
   Run the command below start the agent in background, and set the apropriate environment variables for the current shell instance.
         
         eval `ssh-agent -s`
         
         Agent pid 1608

   Use the command below to unlock keys (usually ~/.ssh/id_*) and load them into the agent, making them accessible to ssh or sftp connections.
   
        ssh-add <path-to-private-key>

        oeume@KRISOLIZ-SFFHPDPC MINGW64 ~/Downloads $ ssh-add /c/Users/oeume/Downloads/keypairname.pem
      
        Identity added: /c/Users/oeume/Downloads/keypairname.pem (/c/Users/oeume/Downloads/keypairname.pem)

Note, Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.


### Step 4b - Configure SSH Connections into target Servers

1. I was unable to use the ssh-copy-id command. I copied pub key to webserver-1 manually and really needed to make use of a simpler command to implement this for the rest of 

   the servers. I ended up enabling password authentication on both host and server and created a password for each after which I ran the ssh-copy-id command.

Run 
                            
     sudo vi /etc/ssh/sshd_config
     
     Set password authentication yes
     
Run 

      sudo systemctl restart sshd
      
      
Run

      sudo passwd ubuntu or sudo passwd ec2-user
      
      This will set a password
      
Run


      ssh-copy-id ec2-user@private-IP-add
      
      /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
      
      /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
      
      ec2-user@private-IP-add's password: 

      Number of key(s) added: 1

      Now try logging into the machine, with:   "ssh '@private-IP-add'"
     
      and check to make sure that only the key(s) you wanted were added.

      ssh ec2-user@private-IP-add
      
      Last login: Mon May 17 15:24:21 2021 from 73.152.119.181
      
      [ec2-user@ip-172-31-23-13 ~]$ logout
      
      Connection to 172.31.23.13 closed.
    
2. Next step is to go back to sshd config file and disable password authentication.

3. Change path to ansible config file to define inventory location as shown below

   [defaults]

   ####### some basic default values...

   inventory       = /home/ubuntu/ansible/ansible-config-mgt/inventory


### Step 5 - Create a Common Playbook


In common.yml playbook, write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure as listed in inventory/dev

Update playbooks/common.yml file with following code:

         ---
         - name: update web, nfs and db servers
           hosts: webservers, nfs, db
           remote_user: ec2-user
           become: yes
           become_user: root
           tasks:
           - name: ensure wireshark is at the latest version
             yum:
               name: wireshark
               state: latest

         - name: update LB server
           hosts: lb
           remote_user: ubuntu
           become: yes
           become_user: root
           tasks:
           - name: ensure wireshark is at the latest version
             apt:
               name: wireshark
               state: latest


The playbook above is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) 

on RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu. Other tasks include creating a 

directory and a file inside it, change timezone on all servers and run some shell script


Run the command below to confirm all hosts are reachable

      ~/ansible/ansible-config-mgt$ ansible all -m ping -u root
      
      
         172.31.23.215 | SUCCESS => {
             "ansible_facts": {
                 "discovered_interpreter_python": "/usr/libexec/platform-python"
             },
             "changed": false,
             "ping": "pong"
         }
         172.31.26.39 | SUCCESS => {
             "ansible_facts": {
                 "discovered_interpreter_python": "/usr/libexec/platform-python"
             },
             "changed": false,
             "ping": "pong"
         }


       ~/ansible/ansible-config-mgt/playbooks$ ansible-playbook common.yml 


First Playbook Run on webserver-1

   ![image](https://user-images.githubusercontent.com/78841364/118551260-b8647480-b72b-11eb-84d9-eace08ce18c4.png)



### Step 6 - Update GIT with the latest code

1.  Push changes made locally to GitHub. 

2.  Enter command here

3. Commit code into GitHub:

   use git commands to add, commit and push the branch to GitHub.

         ~/ansible/ansible-config-mgt$ git add .

         ~/ansible/ansible-config-mgt$ git status
         
         On branch feature01-prj11-layout
         
         Your branch is up to date with 'origin/feature01-prj11-layout'.

         Changes to be committed:
        
         (use "git restore --staged <file>..." to unstage)
               
                  new file:   inventory/dev.yml
                  
                  new file:   inventory/prod.yml
                
                  new file:   inventory/staging.yml
                
                  new file:   inventory/uat.yml
                
                  new file:   playbooks/common.yml


         :~/ansible/ansible-config-mgt$ git push
        
        To https://github.com/eoumenwa/ansible-config-mgt.git
          
          ! [rejected]        feature01-prj11-layout -> feature01-prj11-layout (fetch first)
         
         error: failed to push some refs to 'https://github.com/eoumenwa/ansible-config-mgt.git'
        
         hint: Updates were rejected because the remote contains work that you do
        
         hint: not have locally. This is usually caused by another repository pushing
         
         hint: to the same ref. You may want to first integrate the remote changes
        
         hint: (e.g., 'git pull ...') before pushing again.
         
         hint: See the 'Note about fast-forwards' in 'git push --help' for details.

git commit -m "Initial commit"

         ~/ansible/ansible-config-mgt$ git commit -m "Initial commit"
         
         [feature01-prj11-layout 4cfee5d] Initial commit
                    
         Committer: Ubuntu <ubuntu@ip-172-31-36-216.us-east-2.compute.internal>

~/ansible/ansible-config-mgt$ git pull

         Merge made by the 'recursive' strategy.

         README.md | 7 ++++---
          1 file changed, 4 insertions(+), 3 deletions(-)

~/ansible/ansible-config-mgt$ git push
         
         Enumerating objects: 11, done.
         
         Counting objects: 100% (11/11), done.
         
         Compressing objects: 100% (7/7), done.
         
         Writing objects: 100% (9/9), 1.30 KiB | 443.00 KiB/s, done.
         
         Total 9 (delta 1), reused 0 (delta 0)
         
         remote: Resolving deltas: 100% (1/1), done.
         
         To https://github.com/eoumenwa/ansible-config-mgt.git
            4b046bb..69ca5fd  feature01-prj11-layout -> feature01-prj11-layout
            
4. Create a Pull request (PR)

   Create  a pull request and merge to the master branch.
   
5. Confirm that Jenkins received the push notification from GitHub
   
   ![image](https://user-images.githubusercontent.com/78841364/119287168-ece89c80-bc13-11eb-8f5d-b80b386249a5.png)


6. Checkout from the feature branch into the master, and pull down the latest changes.

     ~/ansible/ansible-config-mgt/inventory$ git checkout master
        
        Switched to branch 'master'
        
        Your branch is up to date with 'origin/master'.
        
     ~/ansible/ansible-config-mgt/inventory$ git pull
     
         fatal: Unable to read current working directory: No such file or directory
    
    ~/ansible/ansible-config-mgt/inventory$ cd ..
        
    ~/ansible/ansible-config-mgt$ git pull
        
        remote: Enumerating objects: 1, done.
        
        remote: Counting objects: 100% (1/1), done.
        
        remote: Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
        
        Unpacking objects: 100% (1/1), 638 bytes | 638.00 KiB/s, done.
         
         From https://github.com/eoumenwa/ansible-config-mgt
         
         12f10e7..785094f  master     -> origin/master
      
         Updating 12f10e7..785094f
        
        Fast-forward
          
          README.md             |  7 ++++---
         
         inventory/dev.yml     | 13 +++++++++++++
         
         inventory/prod.yml    |  0
         
         inventory/staging.yml |  0
         
         inventory/uat.yml     |  0
         
         playbooks/common.yml  | 58 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
         
         6 files changed, 75 insertions(+), 3 deletions(-)
          
          create mode 100644 inventory/dev.yml
         
         create mode 100644 inventory/prod.yml
         
         create mode 100644 inventory/staging.yml
         
         create mode 100644 inventory/uat.yml
         
         create mode 100644 playbooks/common.yml


7. Confirm that Jenkins saved all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.

        ~/ansible/ansible-config-mgt/inventory$ sudo ls -l /var/lib/jenkins/jobs/ansible_automation/builds/7/archive/
      
         total 16
     
         -rw-r--r-- 1 jenkins jenkins 1072 May 16 17:35 LICENSE
     
         -rw-r--r-- 1 jenkins jenkins  628 May 24 02:10 README.md
    
         drwxr-xr-x 2 jenkins jenkins 4096 May 24 02:10 inventory
      
         drwxr-xr-x 2 jenkins jenkins 4096 May 24 02:10 playbooks

### Step 7 - Run first Ansible test

Execute ansible-playbook command and verify that the playbook actually works:

         ansible-playbook -i /var/lib/jenkins/jobs/ansible_automation/builds/7/archive/inventory/dev.yml  
         /var/lib/jenkins/jobs/ansible_automation/builds/7/archive/playbooks/common.yml

### Challenges

1. ERROR: Couldn't find any revision to build. Verify the repository and branch configuration for this job. Finished: FAILURE

   Solution:

   Created a new branch called "master" and went to settings/branches then changed default branch to master from main. This now corresponds to the branch specifier on Jenkins    
   I could have also changed the specifier in Jenkins to */main to match the default branch in GIT.

2. I was unable to use the saved Jenkins artifacts to run ansible test. The play as always halting for no reason since I could run the block of tasks successfully on   
 
   individual hosts. I however setenforce to zero and tried again before I could successfully run the laybook on all servers
   
3. Issues with SSH into the required servers. I had to enable password authentication on all servers to copy ssh ids (see step 4b)



### Summary

In this project, I learnt how to automate routine tasks using Ansible Configuration Management and writing code using declarative language such as YAML.



### Links to Implementation Video

https://drive.google.com/file/d/1nWr2Xx-Lfyy9hX8sIf6gMlxwC44vHeIQ/view?usp=sharing

https://drive.google.com/file/d/1Eck6_bmIsNjtHTwn_mScCld9Cvrgmj7Z/view?usp=sharing




## References

https://stackoverflow.com/questions/33762738/specifying-the-os-ansible

https://medium.com/@christyjacob4/using-vscode-remotely-on-an-ec2-instance-7822c4032cff

https://phoenixnap.com/kb/ssh-permission-denied-publickey

https://stackoverflow.com/questions/22530886/ssh-copy-id-no-identities-found-error

https://linuxhint.com/copy_ssh_keys/

https://unix.stackexchange.com/questions/356259/ssh-between-two-linux-boxes-permission-denied-public-key

https://www.google.com/search?q=meaning+of+lineinfile&oq=meaning+of+lineinfile&aqs=chrome..69i57j69i60l2j69i65j69i60j69i65.3975j1j7&sourceid=chrome&ie=UTF-8

https://www.redhat.com/en/topics/automation/what-is-an-ansible-playbook

https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html

https://stackoverflow.com/questions/36724870/ansible-error-the-field-hosts-is-required-but-was-not-set

https://www.tecmint.com/set-time-timezone-and-synchronize-time-using-timedatectl-command/

https://github.com/ansible/ansible/issues/30411
