# Configure Nodes

## To Start

If you haven't already, make sure you've created your three GCP compute instances to act as our nodes for this demonstration. If you need instructions go to the *[Create Instance Instructions](create_instance.md)* page. Once you have instances up you should see something like the following on GCP instances page:

![GCP Instances Up](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/instancesup.png "Figure 22: GCP Instances Up")

Take note of all the internal IP addresses as we will need this for the configuration management tool later.

### Firewall

So to do NFS on GCP we may need to enable a firewall rule to allow traffic for NFS between all nodes on our internal network subnet. The first step to get there from the instances window is to go to the network details dialog from the triple dot dialog:

![GCP to Network Details](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/instance_networkdetails.png "Figure 23: GCP to Network Details")

Once there click on *Firewall* on the left side dialog under the *VPC Network* details:

![GCP to Firewall](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/tofirewallinterface.png "Figure 24: GCP to Firewall")

Once on the Firewall page, click *Create Firewall Rule* to launch the firewall rule creation interface:

![GCP Create Firewall Rule](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/create_firewall_rule.png "Figure 25: GCP Create Firewall Rule")

Then fill out your filewall rule with your internal network subnet similar to mine, where my source ip range was *10.128.0.0/24* since all my internal static IPs start with *10.128.0*.

![GCP Firewall](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/firewall_rules.png "Figure 26: GCP Firewall")

### SSH via SSH Keys to each node

If you already have ssh public/private keys, and want to run the plabooks locally you can move on to the next step. If your id_rsa ssh keys, the default ones, are password protected, this will be a problem for use with Ansible. So I would choose to run anisble from the primary node instead.

#### Generate New Key

Use the following command to create your ssh RSA public/private key pair:

```bash
bryandixon@ccsc1:~$ ssh-keygen -t rsa -b 4069
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bryandixon/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/bryandixon/.ssh/id_rsa
Your public key has been saved in /home/bryandixon/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:jVdXf2GYX793ke2NwpITflfc2aBB0c3jdIB7vmRf1Is bryandixon@ccsc1
The key's randomart image is:
+---[RSA 4068]----+
|           .o++*.|
|            oo++B|
|            .+=o#|
|         o..o..*@|
|        S.o+ o.==|
|         .= +E*o*|
|           + = o+|
|              . .|
|                 |
+----[SHA256]-----+
```

This will generate new ssh key, for example I ran this on one of my nodes for the above example. I recommend hitting enter for all of the prompts as the defaults and no passphrase is recommended for this application. **Do not do this** if you already have a SSH key pair.

#### Copy key to each node

Now we'll want to copy the SSH key from the primary node to all of the GCP Nodes. We'll have to do this manually since we do not have passwords onto the GCP instances and everything is done via SSH keys. On the primary node get the contents of the SSH key we just generated, so in my case:

```bash
bryandixon@ccsc1:~$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB/Q7kcCejvb4KewZaMej7HZDc2mi271PCbYj4FKZr4mA/6I4xcVqlfUTtrePwE5/M/diOPYNqyuPkxun9HnVdM0J1K3Hj58HRgFNchh7t/wCf+NylMWTfk0vsjxyvbNUWDOpUyqReTETVx5EbwckhWfHMF2NbJT6zt7BoLqVB3sVIREeJQXBBtEz0xndY1coPdqNJieFDQ2V75Hn7+7k1tQDFyeVkcw1LNSM6mi1oxnkEGGbZ6BtibWhLwG9iat2gzFGm0Wlzr42Lig14lLZc0yDB/z0+3HzaPyQow+uqXDDRd2aQZXCh78DwhlNhXCXZQZs3texa1KPdYOZw+uMBiwlGmh8WN+2hcIpBgCV1X8Y0BC5bpH9dHmxmnsGB/zIYbP5bbYxH83NLJKu0WBP8UyFfYIQPhpbmU9sRl1IhBpK/3tMuuZtGdMVwlNYWp7vtn8nxRng/HC+FOWGMdIe1/D+liTBSUsnjnSb5ynnLfhcGAh6tw1Nk7tsn0JiTYtjoM6FtWYkz78iMQZLLDX8GDvOfZ9Zx4kxI59neyCtYCLPfwYZIb2WRakn/kzAzDIDokeB6D/qndde9KVTdAQQQUSdVft07fDJgb7W1AcXEx/poPJ1IO1guZjWBTF4+W2w9mO565rqbLxaYMoJGXPppYL/P5AixCfCvRTtiVyHT bryandixon@ccsc1
```

You'll want to export your SSH key the same way and copy the file contents to your clipboard. Now on each of the other two nodes, in my case *ccsc2* and *ccsc3* we'll want to add this SSH key to a new line in the *authorized_keys* file for the other accounts with our *username*. If you are going to run the tutorial on the primary node, in my case *ccsc1* make sure you add your public key to the *authorized_keys* file for the node it is on as well.

If you need help with a commandline editor let me know and I can help with *nano*, *vim*, or *emacs*.

Now you should be able to connect via ssh from your primary node to the other nodes without password. Check that this works before moving on for all of your GCP Nodes.

## Ansible instructions

There are a couple bits you'll need to configure initially but then the provided playbooks should setup the boards for you automatically.

I recommend setting up Ansible on your personal computer not the nodes; however, you can install it on one GCP node and have it configure itself and the other GCP nodes, which is what I'm going to do for the tutorial just in case since may be easier for some. Ansible in our use case will be going over SSH and running configuration steps for us automatically. If you have not setup SSH to work with SSH keys do this before moving on to this.

### Get Ansible installed

Ansible is built on [Python](https://www.python.org/), I recommend using Python 3.6 or newer for this. To verify which version of Python3 is on your system:

```bash
bryandixon@ccsc1:~$ python3 --version
Python 3.10.6
```

Before we get our environment for Python setup, let's grab the tutorial playbooks:

```bash
bryandixon@ccsc1:~$ git clone https://github.com/javawolfpack/CCSC-Tutorial.git
Cloning into 'CCSC-Tutorial'...
remote: Enumerating objects: 297, done.
remote: Counting objects: 100% (97/97), done.
remote: Compressing objects: 100% (87/87), done.
remote: Total 297 (delta 41), reused 50 (delta 9), pack-reused 200
Receiving objects: 100% (297/297), 3.04 MiB | 12.75 MiB/s, done.
Resolving deltas: 100% (168/168), done.
```

As it can be installed via the Package Installer for Python or pip, to make life simpler, we'll want to use a Python virtual environment, which is a module that comes with Python 3.6 or newer now. By default Ubuntu comes with Python3 now but not the Python3 virtual environment so let's install that first:

```bash
bryandixon@ccsc1:~$ sudo apt update
bryandixon@ccsc1:~$ sudo apt install python3.10-venv
```

Let's create a virtual environment in the Ansible folder for the CCSC-Tutorial files:

```bash
bryandixon@ccsc1:~$ cd CCSC-Tutorial/Ansible/
bryandixon@ccsc1:~/CCSC-Tutorial/Ansible$ python3 -m venv venv
```

This will create a new folder where you run the command above, I recommend running this inside the Ansible folder of this repo after you've cloned it to the computer you will run the playbooks from. Now run the following command to activate the virtual environment:

```bash
bryandixon@ccsc1:~/CCSC-Tutorial/Ansible$ source venv/bin/activate
(venv) bryandixon@ccsc1:~/CCSC-Tutorial/Ansible$
```

Once the you've activated the python virtual environment you should see the name of it similarly to as shown above in parenthesis to indicate it's active. You can always use the command *deactivate* to exit the virtual environment. The benefit of the virtual environment is it will only install packages locally in that virtual environment so you can remove that folder and its gone from your system vs installing it on your system and then it's difficult to uninstall packages. Also useful if you want to use multiple versions of a given package on the same system. And with Python3 we can now just use the command python in the virtual environment we created with python3 since it is sim-linked accordingly while in the virtual environment.

Now we can install Ansible in the virtual environment with the following from the working directory of the Ansible folder in this repo:

```bash
(venv) bryandixon@ccsc1:~/CCSC-Tutorial/Ansible$ pip install -r requirements.txt
```

This will install ansible and any other requirements into the virtual environment you have currently active. Now we need to update your inventory to match your GCP internal ips on your network. I provided you an example inventory file called *inventory*, we will be manually running this inventory vs putting the config files into the system to have ansible run by default. Here is the contents of the example Ansible inventory file:

```
#Adjust these to your actual ips for each board.

[gcp_primary] #ip/Inventory for your primary GCP Node
gcp1 ansible_host=10.128.0.2 ansible_python_interpreter="/usr/bin/python3"

[gcp_worker] #ips/Inventory for your worker GCP Nodes
gcp2 ansible_host=10.128.0.3 ansible_python_interpreter="/usr/bin/python3"
gcp3 ansible_host=10.128.0.4 ansible_python_interpreter="/usr/bin/python3"

[gcp:children]
gcp_primary
gcp_worker

[gcp:vars]
ansible_ssh_user=<username> #Replace <username> with the actual user you want to run commands on the GCP Nodes
```

I gave the group of machines the name *gcp*, which includes the children group for the primary and worker GCP Nodes broken out. Replace your two nodes in the inventory with the correct ip addresses. Additionally, in the last line there is the variable *ansible_ssh_user* to define what user you want the scripts to SSH into the board and run the commands as.

Now check that your Ansible configuration works with your modified inventory file with the following that will ping all the nodes in your inventory:

```bash
(venv) bryandixon@ccsc1:~/CCSC-Tutorial/Ansible$ ansible all -m ping -i inventory
gcp1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
gcp2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
gcp3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

If your setup works, you should see something similar to the output of the command above. Now you can move on to running the playbooks.

### Run Ansible Playbooks

Now that you have ansible installed and working with your inventory you can use the playbooks to setup/configure your GCP Nodes to work as a cluster or for development. You can run the playbooks in the following order or just run playbook all to install everything.

* **initial.yml**
  * Playbook to install updates/upgrade system to started
  * Installs GCC/G++/OpemMPI development tools
  * Installs BLAS libraries
* **nfs.yml**
  * Playbook originally to setup NFS and share the home folder from the primary node
  * Playbook has been modifed to create a unified /mnt/mpi_shared directory instead
  * Creates an SSH public/private key pair for your ansible_ssh_user
  * Makes the ansible_ssh_user public key an authorized key for all the nodes
  * Mounts the NFS /mnt/mpi_shared share as the /mnt/mpi_shared folder for all the worker nodes
* **hosts.yml**
  * **Optional Playbook**
  * Playbook to setup the /etc/hosts file so you can use the inventory names *jn1* and *jn2* or whatever you gave the nodes instead of ip addresses
  * You don't have to do this, but it's a simple way to use hostnames vs ips in your hostfiles, etc.
  * You will need to make sure to ssh to each machine via the hostname in the /etc/hosts or inventory after this so that they're now in *known_hosts* for the system for MPI to work, which you have to do with the ips too.
* **python.yml**
  * **Optional Playbook**
  * If you choose this will setup the following for python3 system wide
    * pip3
    * Virtual Environments
    * OpenMPI bindings for python3
    * Numpy library for python3
* **rust.yml**
  * **Optional Playbook**
  * **Caveat** - This works but only installs Rust onto the primary GCP Node, so all compilation and development will need to happen there for Rust. This is likely what you will want to do regardless but due to the way Rust adds itself to the path and into your home directory it wasn't trivial to install on all the nodes with the shared home folder. If I'm able to resolve this in the future will update but for now this will work.
  * If you choose this playbook will install Rust onto the GCP Nodes
  * This may be useful if you want to play with rust, including implementing the MPI assignments for extra credit in rust with the experimental MPI bindings.
* **clang.yml**
  * **Optional Playbook**
  * If you choose, this will install Clang 7 onto your cluster
  * Clang is a llvm compiler and may provide a lot of novel compiler features that GCC does not by default.
  * Clang is necessary for Rust.
* **zsh.yml**
  * **Optional Playbook**
  * If you choose, this will install ZSH and the powerline fonts onto your cluster
  * It does not force it to be the default shell or add Oh My ZSH currently.

You can run any of the playbooks with the following command structure:

```bash
#replace <playbook file> with the actual ansible playbook to run
(venv) bryandixon@ccsc1:~/CCSC-Tutorial/Ansible$ ansible-playbook -i inventory --ask-become-pass <playbook file>
```

For example to run the *all* playbook that would run all of the playbooks for you in one command you can do the following:

```bash
(venv) bryandixon@ccsc1:~/CCSC-Tutorial/Ansible$ ansible-playbook -i inventory all.yml
```

If you were doing this not on the GCP you would want to do the following:

```bash
(venv) bryandixon@ccsc1:~/CCSC-Tutorial/Ansible$ ansible-playbook -i inventory --ask-become-pass all.yml
```

**Note:** the *--ask-become-pass* option will prompt you for the sudo password for the *ansible_ssh_user* on the Nodes. This is to prevent you from hardcoding this somewhere, but provide privileges to the Ansible scripts to do all the installs. This is not necessary on GCP since our user has sudo access without passwords.




