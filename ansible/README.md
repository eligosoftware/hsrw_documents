# Ansible

Ansible is an automation tool for application deployment in IT infrastructure. Besides that it also automates cloud provisioning, configuration management and other IT tasks. Noteworthy, Ansible does not need agent software for communication with nodes and uses easy to understand YAML notation. Read more at https://www.ansible.com/overview/how-ansible-works


## Installation

Currently Ansible can be installed either by Linux distribution's package manager (yum, apt) or with pip. I couldn't install it on Windows PC with command
```
pip install ansible
```

but could install it on virtual OS (Debian) with command

```
sudo apt install ansible -y
```
Read more about installation here https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#selecting-an-ansible-artifact-and-version-to-install

## Configuration

First, I ensured that all devices are in the same WiFi network and run "ifconfig" command on Jetson Nanos to obtain their IP addresses.
Second, the host file of ansible on Debian virtual OS was edited to set the correct IP values

![image](https://user-images.githubusercontent.com/15895292/161236909-b8526dbc-5b83-468b-81c6-0d721b772d65.png)

It is important to add a group name and the list of IP addresses alongside with the root username and passwords

![image](https://user-images.githubusercontent.com/15895292/161237121-f766fe56-c9bd-41ed-b240-0aedb2ea6ae0.png)

Ansible uses SSH to connect to the nodes. At first try I got security error:
```
ERROR! Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host.
```
The workaround I found was to set environment variable value:

```
export ANSIBLE_HOST_KEY_CHECKING=False
```

## Ansible Playbooks

Ansible uses component based approach and YAML syntax in commands' composition. For each type of command Ansible has a set of prepared modules: git, bash, mail, ec2 and more. Full list is here https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html.

These commands should be written in a file, called ansible playbook, extension yml.

The example of ansible playbook file for installing numpy

```
---
- name: install numpy book
  hosts: ansible_client
  remote_user: ilgar
  become: true
  tasks:
   - name: install pip
     apt:
         name: python3-pip
         state: latest
   - name: download requirements
     get_url:
         url: http://192.168.179.13:5000/download
         dest: ./requirements.txt
   - name: install packages
     shell: pip3 install -r requirements.txt
```

Here I mention the name of playbook, the target devices group (remember I wrote it in host file during configuration?), remote_user (don't understand the meaning, because I wrote the root user in hosts file), become - True (means super user privileges should be acquired for task), and the list of tasks. For each task I write the name of task and the command. In this example I used components apt (for installing pip), get_url (for downloading requirements.txt file) and shell for installing libraries from this requirements file (actually there is pip component, but I could not configure it to work with my version of Python).

## Running playbook

Before running it is important to check for syntax errors. The main problem might be indentations or misspelling errors. Syntax check is done using command:

```
ansible-playbook <filename.yml> --syntax-check
```

It should return filename if no error was found

![image](https://user-images.githubusercontent.com/15895292/161241334-4fec2806-45a6-4080-9150-e5b83e01e33c.png)

Running the commands is done using the script
```
ansible-playbook <filename.yml> --ask-become-pass
```
--ask-become-pass flag is important for input the SUDO password. I noticed that without it the commands could not be run on nodes for Permission denied error.

The following are images of setting up Tello library:

![image](https://user-images.githubusercontent.com/15895292/161242042-b68175b2-ae03-40d3-af45-f6045c9f789c.png)

![image](https://user-images.githubusercontent.com/15895292/161242079-7cbeb82c-d44d-439f-97af-73b575e6f1be.png)
