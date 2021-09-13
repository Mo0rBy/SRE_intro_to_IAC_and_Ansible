# Using Ansible - Infrastructure as Code
## What is Ansible?
Ansible is an automation tool that allows you to change settings on multiple machines while only using 1 controller machine. In our example, we will only be using 3 machines, but in realisty, Ansible can be used to change settings on 10's or even hundreds of machines.

![](https://github.com/Mo0rBy/SRE_intro_to_IAC_and_Ansible/blob/main/img/Ansible_Diagram.png)

### Setting up the machines

The `VagrantFile` will setup 3 machines:
1. Controller
2. Web
3. db

After initialising these machines, we want to set them up using:
1. `sudo apt-get update -y`
2. `sudo apt-get upgrade -y`

### Installing Ansible
We only need to install Ansible on 1 machine and not on the machines that are controlled by Ansible. This is known as "agentless".

*Defintion of Agentless (from Whatis.com) = Agentless, in computing, refers to operations where no service, daemon or process (AKA an agent) needs to run in the background on the machine the action is being performed on.*

To install Ansible, execute the following commands:
```
sudo apt-get install software-properties-common -y
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get install ansible -y
```

We can confirm that Ansible is installed by using `ansible --version`. This command will give information on the installed Ansible version and the location of the default configuration directories.
This machine is now the Ansible controller machine.

### Manual SSH with Ansible
To SSH into the agent machines, simply run `ssh vagrant@[machine-ip]`, where `[machine-ip]` is substituted for the machines IP address.

Can test the the connection works by running any command. To exit back to the controller machine's terminal, use `Exit`.

### Ansible configuration file, pinging and SSH login
We can ping the machines using the Ansible command > `ansible all -m ping`

When we execute this command, we will see this output:
```
[WARNING]: provided hosts list is empty, only localhost is available. Note that the        
implicit localhost does not match 'all'
```

#### Define the agent machines and ping
We haven't defined what machines we want to control. So, we must now go into the Ansible config file and tell Ansible what machines to communicate with.

Go to the Ansible config directory > `cd /etc/ansible/`
Edit the `hosts` file and add the IP of your machines.

Execute the `ansible all -m ping` again. Now we get this output:

![](https://github.com/Mo0rBy/SRE_intro_to_IAC_and_Ansible/blob/main/img/Ansible_ping_denied.PNG)

Edit the `/etc/ansible/hosts` file again and add the following variables after the machines IP address:
```
192.168.33.10 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant
```

This tells Ansible to connect using SSH, and gives the connection a username and password.

Now if we execute the `ansible all -m ping`, we get the output:

![](https://github.com/Mo0rBy/SRE_intro_to_IAC_and_Ansible/blob/main/img/Ansible_ping_accepted.PNG)

__*You may see a depreciation warning, but this isn't anything to worry about.*__

We can ping the same machine using `ansible web -m ping` because we gave the web machine the `[web]` heading in the `hosts` file.

**Repeat the exact same process with the `db` machine _(and make sure to put it under the `[db]` header)_**

### Ad hoc commands
We can execute Adhoc commands using Ansible to obtain information about the agent machines. For example, we can find the name of our `web` machine like this > `ansible web -a "uname -a"` and we will see this output:

![](https://github.com/Mo0rBy/SRE_intro_to_IAC_and_Ansible/blob/main/img/adhoc_uname.PNG)

This is carrying out an "ad hoc" command on a single machine, but we can run the command on **all** our machines using only 1 command > `ansible all -a "uname -a"`

The basic idea is, we can use Ansible to execute any command we would normally use within the Linux terminal and Ansible will then show us the output of those commands from each machine.

---
## To-Do Check-List
- IaC - Configuration Management and Orchestration
- Which tools are used for "Push" config and "Pull" config

- What is Ansible and benefit of it
- Why we should use Ansible
- Create a diagram for Ansible on-prom, hybrid and public architecture
- Installation and setting up Ansible controller with 2 agent nodes > Include commnads in your README.md
- What is the default deirectory structure for Ansible
- What is the Inventory/hosts file and the purpose of it
- What should be added to the hosts file to establish a secure connection between Ansible controller and agent nodes?(include code block)
- What are Ansible ad-hoc commands
- add a structure of creating adhoc commands > e.g. `ansible all -m ping` (-m means module) [break down and explain the syntax]
- include all the ad-hoc commands we have used today in this documentation

---
## Ansible Playbooks
Ansible playbooks are a completely different way to use Ansible when compared to ad-hoc commands. The new goal is install the applications we need for the `web` and `db` machines. Instead of doing this manually for both machines, we can create playbooks.

Ansible playbooks are `.yml` files *(can also be written as `.yaml`)*. `YAML` stands for **Y**et **A**nother **M**arkup **L**anguage.

### Installing nginx using an Ansible Playbook
Playbooks always start with 3 dashes > `---`

**INSERT YAML FILE HERE**

- The `- hosts: web` tells Ansible to perform the following tasks on all the machines that come under the `[web]` heading.
- The `gather_facts: yes` tells Ansible to print what it is doing to the terminal. *It basically just prints the task that it is currently completeing*
- The `become: true` line gives the YAML file admin access. *This is the same logging in as the root user, it is required to install applications*
- The next part is `tasks:`. This tells the YAML file that the following lines will be tasks.
- `- name: Install Nginx` creates a new task with the name *Install Nginx* and the following lines within the code block are part of that task
- `apt: pkg=nginx state=present` tells YAML to install the nginx package using the `apt` package manager. `state=present` tells YAML to ensure that the application is running.

The YAML file needs to be present within the `/etc/ansible` directory. To execute the playbook, the command is > `ansible-playbook nginx_playbook.yml`

After the playbook has finsihed running and installing nginx, we can check that it is running using and ad-hoc command > `ansible web -a "systemctl status nginx"`

We should see this output:

![](./img/ansible_nginx_status_check.PNG)

### 