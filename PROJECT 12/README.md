ANSIBLE REFACTORING, ASSIGNMENTS AND IMPORTS
Introduction
Refactoring in Ansible
Refactoring in Ansible involves restructuring your playbooks, roles, and tasks to improve their readability, maintainability, and efficiency without changing their functionality. This can include breaking down complex playbooks into smaller, reusable components, simplifying tasks, and improving variable management.

Common refactoring techniques in Ansible:
Splitting Playbooks: Breaking a large playbook into smaller, more manageable playbooks.
Using Roles: Organizing tasks into roles to promote reuse and modularity.
Variable Consolidation: Centralizing variables in a vars file or in group/host variable files.
Task Optimization: Combining similar tasks, removing redundant tasks, and using loops.
Example before refactoring:

- hosts: webservers
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start Nginx
      service:
        name: nginx
        state: started
Assignment in Ansible
Assignment in Ansible refers to setting values to variables, which can be used to store data such as configuration parameters, credentials, or dynamic values that can be reused throughout playbooks and roles.

Methods of assignment in Ansible:
Direct Assignment: Assigning variables directly within a playbook or task.
Using vars Files: Storing variables in separate files and including them in playbooks.
Group/Host Variables: Defining variables specific to groups of hosts or individual hosts in inventory files.
Example:

# Direct Assignment
- hosts: webservers
  vars:
    nginx_package: nginx
  tasks:
    - name: Install Nginx
      apt:
        name: "{{ nginx_package }}"
        state: present

# Using `vars` File
# vars/nginx.yml
nginx_package: nginx

# Playbook
- hosts: webservers
  vars_files:
    - vars/nginx.yml
  tasks:
    - name: Install Nginx
      apt:
        name: "{{ nginx_package }}"
        state: present
Import in Ansible
Importing in Ansible is used to include tasks, playbooks, or roles into other playbooks, allowing for better organization and reuse of code. There are two main directives for this purpose: import_tasks, import_playbook, and import_role.

Examples:

# Importing Tasks
- hosts: webservers
  tasks:
    - import_tasks: tasks/nginx_install.yml

# tasks/nginx_install.yml
- name: Install Nginx
  apt:
    name: nginx
    state: present

# Importing Playbooks
# main_playbook.yml
- import_playbook: webservers.yml
- import_playbook: databases.yml

# Importing Roles
- hosts: webservers
  tasks:
    - import_role:
        name: nginx
Importance Of Refactoring, Assignment And Imports
Effective Ansible usage is critical for organizations aiming to automate infrastructure management and application deployment. Refactoring promotes code clarity, reducing errors and easing collaboration. Assignments empower playbooks with dynamic capabilities, adapting to diverse scenarios. Imports enable the reuse of well-defined components, fostering modularity and scalability. Mastery of these concepts enhances productivity, making Ansible an indispensable asset in the DevOps toolbox.

Refactor Ansible Code By Importing Other Playbooks Into site.yml
Step 1: Jenkins Job Enhancement In the previous project, we create a jenkins build job, let us make some changes to our Jenkins job. Now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins server with each subsequent change. Let us enhance it by introducing a new Jenkins job - we will require Copy Artifact plugin.

i. Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact - we will store there all artifacts after each build.

sudo mkdir /home/ubuntu/ansible-config-artifact

ii. Change permissions to this directory, so Jenkins could save files there.

chmod -R 0777 /home/ubuntu/ansible-config-artifact

