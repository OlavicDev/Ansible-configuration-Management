
# ANSIBLE CONFIGURATION MANAGEMENT (Automating previous projects Projects)

In this project, we’ll automate some routine operations using **Ansible Configuration Management**, a powerful DevOps tool that enables you to automate server configuration and application deployment.

This project will help you:
- Understand **DevOps tools** and their benefits.
- Gain confidence in writing code using declarative languages like YAML.
- Automate repetitive tasks, thus improving efficiency and consistency across environments.

## Ansible Client as a Jump Server (Bastion Host)
A **Jump Server** (or Bastion Host) is a secure gateway to access resources inside a private network. If your architecture involves web servers inside a secured network, you wouldn’t be able to SSH into them directly. Instead, you’ll SSH into the Jump Server, which allows controlled access to those internal servers, thereby enhancing security.

## Task Overview
- Set up and configure an EC2 instance with Ansible to act as a Jump Server.
- Automate the configuration of your servers using an Ansible playbook.
![image](https://github.com/user-attachments/assets/54256d87-d26d-4a8a-b918-2a9e76e84292)


Let’s break down the steps to configure and automate server management using Ansible.

---

## Step 1: Install and Configure Ansible on Your EC2 Instance

### 1. Update EC2 Instance Name Tag
Start by renaming your Jenkins EC2 instance to `Jenkins-Ansible`, which will act as the Ansible control node for managing your infrastructure.

![image](https://github.com/user-attachments/assets/dc25771b-caab-49d9-85c0-df7dc61b0a9c)


### 2. Create a GitHub Repository
Create a new GitHub repository named `ansible-config-mgt` to store your Ansible playbooks and configurations.
![image](https://github.com/user-attachments/assets/827213ff-6add-40d2-8c18-6503113898dd)


### 3. Install Ansible on Your EC2 Instance
To install Ansible, follow these commands:

```
sudo apt update
sudo apt install ansible
```

To verify the installation, check the Ansible version:

```
ansible --version
```
![image](https://github.com/user-attachments/assets/08e72361-f1e1-4181-85bc-025554400ea9)


### 4. Integrate GitHub with Jenkins (CI/CD Automation)
- **Configure GitHub Webhook**: Set up a webhook on your GitHub repository to trigger Jenkins builds when code is pushed.

- **Create a Jenkins Job**: Set up a Freestyle project in Jenkins to pull code from your GitHub repository, run Ansible playbooks, and archive the build artifacts.

#### Steps to Configure Jenkins Job:
- In GitHub, go to **Settings > Webhooks > Add Webhook**.
![image](https://github.com/user-attachments/assets/99e97b3e-aacb-44ae-bc8d-ad6a3114776e)


- In Jenkins, create a **Freestyle project** named `ansible` and connect it to the GitHub repository.
  ![image](https://github.com/user-attachments/assets/97d0bb09-c9a2-46dc-b792-af947b98b131)

  
```
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
![image](https://github.com/user-attachments/assets/24605ed2-da08-4913-91e7-3a47026bdfc8)

### 5. Assign an Elastic IP to Jenkins-Ansible
Allocate and associate an Elastic IP to your `Jenkins-Ansible` EC2 instance to avoid reconfiguring the webhook each time the instance restarts.
![image](https://github.com/user-attachments/assets/a8032177-fecd-4fe8-a510-3f53cbec0916)

---

## Step 2: Set Up Development Environment in Visual Studio Code

### 1. Install Visual Studio Code
Download and install [Visual Studio Code](https://code.visualstudio.com/download), a versatile text editor that will allow you to easily edit YAML files for Ansible.

### 2. Clone the GitHub Repository
Once VS Code is set up, clone your `ansible-config-mgt` GitHub repository to your local machine or EC2 instance:

```
git clone https://github.com/<your-username>/ansible-config-mgt.git
```

---

## Step 3: Start Ansible Development

### 1. Create a New Feature Branch
Create a new branch for your Ansible configuration:

```bash
git checkout -b feature/prj-11-ansible-config
```

### 2. Create Folders for Playbooks and Inventory
Organize your project by creating the following directories:

```bash
mkdir playbooks
mkdir inventory
```

### 3. Create Your First Playbook
In the `playbooks` directory, create your first playbook:

```bash
touch playbooks/common.yml
```

### 4. Create Environment-Specific Inventory Files
Create inventory files for different environments (dev, staging, production):

```bash
touch inventory/dev.yml inventory/staging.yml inventory/prod.yml
```

---

## Step 4: Configure the Ansible Inventory

An **Ansible inventory** file defines the hosts and groups on which commands and tasks will run. Here's a sample inventory configuration for the `dev.yml` file:

```yaml
all:
  children:
    webservers:
      hosts:
        <web-server1-private-ip>:
          ansible_ssh_user: ec2-user
        <web-server2-private-ip>:
          ansible_ssh_user: ec2-user
    db:
      hosts:
        <db-server-private-ip>:
          ansible_ssh_user: ubuntu
    lb:
      hosts:
        <lb-server-private-ip>:
          ansible_ssh_user: ubuntu
```

To set up SSH access to your servers, start the SSH agent and add your private key:

```bash
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```

---

## Step 5: Create the Ansible Playbook

A playbook defines tasks that Ansible will perform on your servers. For example, the `common.yml` playbook below installs Wireshark on both RHEL 9 and Ubuntu servers:

```yaml
---
- name: Update web and NFS servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: true
  tasks:
    - name: Ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: Update LB and DB servers
  hosts: lb, db
  remote_user: ubuntu
  become: true
  tasks:
    - name: Update apt repo
      apt:
        update_cache: yes

    - name: Ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

### Additional Tasks (Optional):
You can extend the playbook to:
- Create directories and files.
- Change timezones.
- Run shell scripts.

---

## Step 6: Push Code to GitHub and Create a Pull Request

Once your changes are ready, push the code to GitHub:

```bash
git add .
git commit -m "Add common.yml playbook"
git push origin feature/prj-11-ansible-config
```

Create a **Pull Request (PR)** in GitHub and have it reviewed by your team. Once approved, merge it into the `main` branch.

---

## Step 7: Running the Ansible Playbook

### 1. Connect VS Code to EC2 for Remote Development
Install the **Remote - SSH** extension in VS Code and configure it to connect to your Jenkins-Ansible EC2 instance.

### 2. Run the Ansible Playbook
To execute your playbook, run the following command from the `Jenkins-Ansible` server:

```bash
ansible-playbook -i inventory/dev.yml playbooks/common.yml
```

### 3. Verify the Changes
On each server (web, NFS, database, etc.), verify that Wireshark is installed:

```bash
which wireshark
wireshark --version
```

---

## Step 8: Automating the Cycle
Repeat the cycle of updating your playbook, committing changes, creating a PR, and running the Ansible playbook to manage your servers automatically.

