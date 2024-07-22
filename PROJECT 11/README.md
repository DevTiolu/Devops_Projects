ANSIBLE AUTOMATE PROJECT
Introduction To Ansible
Ansible is an open-source automation tool used for configuration management, application deployment, and task automation. Developed by Red Hat, it aims to simplify complex tasks and make them repeatable and manageable. Here are the key points to understand Ansible:

Key Features of Ansible:
Agentless: Ansible does not require any agent software to be installed on the target machines. It communicates over SSH or Windows Remote Management (WinRM).

Simple and Human-readable: Ansible uses YAML (Yet Another Markup Language) for its configuration files, known as playbooks, making them easy to read and write.

Idempotent: Ansible ensures that operations are performed only if the system state changes are necessary, preventing redundant actions.

Extensible: Ansible can be extended with custom modules, plugins, and inventory sources to fit specific needs.

Components of Ansible:
Modules: Reusable, standalone scripts that perform specific tasks (e.g., managing packages, users, or services).

Playbooks: YAML files that define a series of tasks to be executed on remote systems. Playbooks are the heart of Ansible's configuration management.

Inventory: A list of hosts (nodes) to be managed, which can be static or dynamically generated.

Roles: A way to organize playbooks and tasks into reusable units, making complex playbooks easier to manage.

Basic Workflow:
Define Inventory: Create an inventory file listing the servers and their groups.

Write Playbooks: Write YAML-based playbooks describing the desired state and tasks to be performed.

Execute Playbooks: Use the ansible-playbook command to apply the playbook to the target inventory.

Ansible Client as a Jump Server (Bastion Host)
A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces attack surface.

On the diagram below the Virtual Private Network (VPC) is divided into two subnets – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses

alt text

General importance:
Configuration Management: Ensuring that servers are configured correctly with desired settings and software.

Application Deployment: Automating the deployment of applications across multiple servers.

Task Automation: Automating repetitive administrative tasks, such as backups or user management.

Continuous Delivery and Integration: Integrating with CI/CD pipelines to automate the build, test, and deployment processes.

Implementing Ansible For Automation
Step 1: Install and Configure Ansible on EC2 Instance
Launch an instace and name it Jenkins-Ansible. We will install and configure jenkins and ansible on this instance.

Create a new repository call ansible-config-mgt. This repository will be connected to jenkins pipeline and also store ansible files

Install Ansible

sudo apt update
sudo apt install ansible
alt text

Confirm Ansible has been successfully installed

ansible --version

alt text

step 2: Configure Jenkins Build Job To Archive Your Repository Content Every Time It Is Changed
Jenkin is a pipeline for continuous integration. To create a build job, we need to have jenkins installed and configured first.

On the same EC2 instance ansible was installed, we need to install jenkins *** Update package repositories
sudo apt update

Install JDK
sudo apt install default-jdk-headless

Install Jenkins
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
/etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
Check if jenkins has been installed, and it is up and running

sudo systemctl status jenkins

alt text

On our Jenkins-Ansible instance, create new inbound rules for port 8080 in security group
alt text

Set up Jenkins
i. Input your Jenkins-Ansible Instance ip address on your web browser i.e. http://public_ip_address:8080

ii. On your Jenkins-Ansible instance, check "/var/lib/jenkins/secrets/initialAdminPassword" to know your password.

alt text

iii. Installed suggested plugins

iv. Create a user account

v. Log in to jenkins console

alt text

Confiure Jenkins To Receive Source Code From ansible-cofig-mgt
i. Allow webhook in our github repository. In ansible-config-mgt repository, navigate to settings>webhooks and past the url of jenkins.

alt text

ii. Create a freestyle project in your jenkins web account and name it "ansible_automation"

iii. Connect jenkins to ansible-config-mgt repository by pasting the repository url in the area selected below

alt text

v. Save configuration and run "build now" to connect jenkins to our repository

alt text

v. Click "Configure" your job and add these two configurations

Configure triggering the job from GitHub webhook and Configure "Post-build Actions" to archive all the files.

alt text

click on add post-build actions and click "Archive the Artifact" then save

Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

vi. Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/

alt text

Note: Trigger Jenkins project execution only for /main (main) branch.

Now your setup will look like this:

alt text

Tip: Every time you stop/start your Jenkins-Ansible server – you have to reconfigure GitHub webhook to a new IP address, in order to avoid it, it makes sense to allocate an Elastic IP to your Jenkins-Ansible server (you have done it before to your LB server in Project 10). Note that Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once you terminate your EC2 Instance.

Step 2 – Prepare your development environment using Visual Studio Code
First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code (VSC).

After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.

Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance using remote development(extension)

alt text

Step 3 - BEGIN ANSIBLE DEVELOPMENT
In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.
Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool – include ticket number (e.g. PRJ-145) in the name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm)

Checkout the newly created feature branch to your local machine and start building your code and directory structure

Create a directory and name it playbooks – it will be used to store all your playbook files.

Create a directory and name it inventory – it will be used to keep your hosts organised.

Within the playbooks folder, create your first playbook, and name it common.yml

Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

alt text

Step 4: Setup Ansible Inventory
In Ansible, an inventory file is a simple text file that defines the hosts and groups of hosts that Ansible can manage. It serves as a source of truth for Ansible to know which hosts it should target when running tasks and playbooks. The inventory file is a fundamental concept in Ansible and is used to organize and manage your infrastructure.

We need setup our ssh keys for ansible so it can have access to our host (remote servers) when running playbooks for our hosts.

This can be done using an SSH agent. An SSH agent allows you to securely store and manage your SSH private keys and provides a convenient way for ansible to authenticate with remote hosts without repeatedly entering your SSH key.

eval `ssh-agent -s`
ssh-add <path-to-private-key>
Confirm the key has been added with the command below, you should see the name of your key

ssh-add -l

Now, ssh into your Jenkins-Ansible server using ssh-agent

ssh -A ubuntu@public-ip

alt text

Also note, that your Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.

Update your inventory/dev.yml file with this snippet of code:

i. Open the inventory/dev.yml file we created earlier

sudo nano /inventory/dev.yml

ii. Paste the below information

[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user=ec2-user 

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user=ec2-user
<Web-Server2-Private-IP-Address> ansible_ssh_user=ec2-user

[db]
<Database-Private-IP-Address> ansible_ssh_user=ec2-user 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user=ubuntu
CREATE A COMMON PLAYBOOK
Step 5 – Create a Common Playbook
It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in inventory/dev.

In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your playbooks/common.yml file with following code:

---
- name: update web, nfs and db servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

Step 6 – Update GIT with the latest code
Push all the changes made to the directories and files from local machine to Github.

Commit your code into GitHub:

use git commands to add, commit and push your branch to GitHub.

git status

git add <selected files>

git commit -m "commit message"
push changes to a remote repositories

git push

Wear the hat of another developer for a second, and act as a reviewer.

If the reviewer is happy with your new feature development, merge the code to the master branch on your local computer, commit and push changes to your remote repository.

Once your code changes appear in master branch - Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server as we configured it to in Step 2

alt text

RUN FIRST ANSIBLE TEST
Step 7 – Run first Ansible test
Now, it is time to execute ansible-playbook command and verify if your playbook actually works:

Connect to your jenkins-ansible server via VScode (configure the .ssh/config file with your jenkins server information)

ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds//archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds//archive/playbooks/common.yml

alt text

Feel free to update this playbook with following tasks:

Create a directory and a file inside it Change timezone on all servers Run some shell script

alt text

At the end of this project we have implemented a solution that is shown below