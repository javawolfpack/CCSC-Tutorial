# CCSC Tutorial
Instructions on setting up a micro compute cluster with GCP.

### Redeem GCP Coupon

This will be handed out at the start of the tutorial. You'll need to go to the following link to redeem the coupon:

[https://console.cloud.google.com/education](https://console.cloud.google.com/education)

I do not recommend redeeming this on your campus gmail if that is what you have as I know my campus google services are configured to not allow google cloud usage by default.

#### Redeeming Coupon

Clicking on that link will povide a dialog that looks like below:

![GCP Redemption](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/GCP_application.png "Figure 1: GCP Redemption")

In this form you should make sure you have your name and enter the GCP coupon that was handed out at the start of this tutorial.


### SSH via SSH Keys to each node

If you already have ssh public/private keys, and want to run the plabooks locally you can move on to the next step. If your id_rsa ssh keys, the default ones, are password protected, this will be a problem for use with Ansible. So I would choose to run anisble from the primary node instead.

#### Generate New Key

Use the following command to create your ssh RSA public/private key pair:

```bash
$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bryan/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/bryan/.ssh/id_rsa.
Your public key has been saved in /home/bryan/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:kFyU0PLIKO7efZdNj7rM4GLD6PdcBzDFkuMAyaiSkKc bryan@jn1
The key's randomart image is:
+---[RSA 4096]----+
| . o.o.+o+.      |
|o o o.oo*..      |
|.=   o+*oo       |
|E . . o.oo       |
|.. .    S .      |
|  .        ..    |
| .   o  . .+.o   |
|  ....*o.=o.o .  |
| ...oo.=+.=o     |
+----[SHA256]-----+
```

This will generate new ssh key, for example I ran this on one of my nodes for the above example. I recommend hitting enter for all of the prompts as the defaults and no passphrase is recommended for this application. Do not do this if you already have a SSH key pair.

#### Copy key to each board

Now we'll want to copy the SSH keys to all of the GCP Nodes. You will need to do the following to each of the GCP nodes internal ip addresses. You may want to SSH to each board via password first to remove the complication of it prompting about if you want to connect during the key transfer:

```bash
$ ssh-copy-id bryan@192.168.1.2
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/bryan/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
bryan@192.168.1.2's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'bryan@192.168.1.2'"
and check to make sure that only the key(s) you wanted were added.
$
```

Now you should be able to connect via ssh to the machine without password. Check that this works before moving on for all of your GCP Nodes.

## Ansible instructions

I recommend this approach, as hopefully it'll setup your cluster automatically for you. There are a couple bits you'll need to configure initially but then the provided playbooks should setup the boards for you automatically. If you chose I hopefully have also included [the step-by-step manual instructions later in this tutorial](#step-by-step-cluster-instructions)

I recommend setting up Ansible on your personal computer not the nodes; however, you could technically install it on one board and have it configure itself and the other board. Ansible in our use case will be going over SSH and running configuration steps for us automatically. If you have not setup SSH to work with SSH keys do this before moving on to this.

### Get Ansible installed

Ansible is built on [Python](https://www.python.org/), I recommend using Python 3.6 or newer for this. As it can be installed via the Package Installer for Python or pip, to make life simpler, we'll want to use a Python virtual environment, which is a module that comes with Python 3.6 or newer now. To initially create a virtual environment do the following:

```bash
$ python3 -m venv venv
```

This will create a new folder where you run the command above, I recommend running this inside the Ansible folder of this repo after you've cloned it to your personal computer. Now run the following command to activate the virtual environment:

```bash
$ source venv/bin/activate
(venv)$
```

Once the you've activated the python virtual environment you should see the name of it similarly to as shown above in parenthesis to indicate it's active. You can always use the command *deactivate* to exit the virtual environment. The benefit of the virtual environment is it will only install packages locally in that virtual environment so you can remove that folder and its gone from your system vs installing it on your system and then it's difficult to uninstall packages. Also useful if you want to use multiple versions of a given package on the same system. And with Python3 we can now just use the command python in the virtual environment we created with python3 since it is sim-linked accordingly while in the virtual environment.

Now we can install Ansible in the virtual environment with the following from the working directory of the Ansible folder in this repo:

```bash
$ pip install -r requirements.txt
```

This will install ansible and any other requirements into the virtual environment you have currently active. Now we need to update your inventory to match your GCP internal ips on your network. I provided you an example inventory file called *inventory*, we will be manually running this inventory vs putting the config files into the system to have ansible run by default. Here is the contents of the example Ansible inventory file:

```
#Adjust these to your actual ips for each board.

[gcp_primary] #ip/Inventory for your primary GCP Node
gcp1 ansible_host=192.168.1.2 ansible_python_interpreter="/usr/bin/python3"

[gcp_worker] #ips/Inventory for your worker GCP Nodes, if you have additional boards add them here.
gcp2 ansible_host=192.168.1.3 ansible_python_interpreter="/usr/bin/python3"

[gcp:children]
gcp_primary
gcp_worker

[gcp:vars]
ansible_ssh_user=<username> #Replace <username> with the actual user you want to run commands on the GCP Nodes
```

I gave the group of machines the name *gcp*, which includes the children group for the primary and worker GCP Nodes broken out. Replace your two nodes in the inventory with the correct ip addresses. Additionally, in the last line there is the variable *ansible_ssh_user* to define what user you want the scripts to SSH into the board and run the commands as.

Now check that your Ansible configuration works with your modified inventory file with the following that will ping all the nodes in your inventory:

```bash
$ ansible all -m ping -i inventory
jn1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
jn2 | SUCCESS => {
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
  * Playbook to setup NFS and share the home folder from the primary node
  * Creates an SSH public/private key pair for your ansible_ssh_user
  * Makes the ansible_ssh_user public key an authorized key for all the nodes
  * Mounts the NFS home share as the home folder for all the worker nodes
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
* **spark.yml**
  * **Optional Playbook**
  * If you install Apache Spark, you should install the Python playbook first.
  * If you choose this will setup the following
    * OpenJDK8
    * Apache Spark
    * .bashrc bindings for Spark for ansible_ssh_user
    * Start the master/workers for Spark
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
$ ansible-playbook -i inventory --ask-become-pass <playbook file>
```

For example to run the *all* playbook that would run all of the playbooks for you in one command you can do the following:

```bash
$ ansible-playbook -i inventory --ask-become-pass all.yml
```

**Note:** the *--ask-become-pass* option will prompt you for the sudo password for the *ansible_ssh_user* on the GCP Nodes. This is to prevent you from hardcoding this somewhere, but provide privileges to the Ansible scripts to do all the installs.

### MPI Run Issues

There is a chance that you may get an odd error that looks like the following when you try to run MPI tasks:

```bash
[gcp1][[39096,1],0][btl_tcp_endpoint.c:649:mca_btl_tcp_endpoint_recv_connect_ack] received unexpected process identifier [[39096,1],2]
```

If this is the case you'll want specify the tcp interface that communication occurs over in the mpirun command as follows:

```bash
$ mpirun --mca btl_tcp_if_include eth0 <rest of mpirun command>
```

I ran into this on one of the cluster builds after the Ansible playbook ran but not another. Everything still works, just requires a bit more information specified from you as the user to run MPI.



## Linpack Benchmarks (Optional)

Now that you have a functioning cluster, you should benchmark it so that you can see how you stack against the [top 500 supercomputer list](https://www.top500.org/). To do this you'll need to compile and run the High Performance Linpack (HPL) benchmark.

### Steps
* On the primary node, login as your **regular user.**
* Download and extract the HPL tarball.

```bash
~$ wget https://www.netlib.org/benchmark/hpl/hpl-2.3.tar.gz
~$ tar -zxvf hpl-2.3.tar.gz
```
* Prep a makefile for your system.
```bash
~$ cd hpl-2.3
~/hpl-2.3/$ cp setup/Make.Linux_PII_CBLAS Make.Linux
```

* Edit your *Make.Linux* file to have the correct install locations

```Makefile
#### edit only the following lines, leave everything else the same: ####
ARCH         = Linux
TOPdir       = $(HOME)/hpl-2.3
MPdir        = /usr/lib/aarch64-linux-gnu/openmpi
MPinc        = -I$(MPdir)/include
MPlib        = $(MPdir)/lib/libmpi.so
LAdir        = /usr/lib/aarch64-linux-gnu/blas
LAlib        = $(LAdir)/libblas.so
LINKER       = /usr/bin/gfortran
```

* Edit the *Makefile* to point to your arch file by changing one line:

```Makefile
arch = Linux
```

* Compile HPL

```bash
~/hpl-2.3/$ make arch=Linux
```

  * If all goes well this should complete within a minute
  * The binary it installs is called *xhpl* and it should be in the *hpl-2.3/bin/Linux*
  * If you see failures, **read them carefully** likely you made a typo in your Makefile. Also make sure your ansible or setup instructions completed correctly.

### Run HPL on your Cluster

In order to run HPL you will need to edit a file called HPL.dat to match your system setup. We will be running HPL at a very small scale to save time as a fully optimized HPL could take a very very long time to complete. Running this small scale HPL took me 1.5 hours to complete as an example. I'm making the assumption you are running this on 2 Jetson Nano boards, if you are using more/different machines you'll have to figure out the details for the configurations yourself or come see me for help.

```bash
$ cd $HOME/hpl-2.3/bin/Linux
Linux$ ls
HPL.dat xhpl
```

Create a hostfile in this directory so for your 2 node Jetson Nano cluster your file should look like this:

```
192.168.1.2 slots=4
192.168.1.3 slots=4
```

Your ips may differ depending on your network setup, but should reflect the *jn1* and *jn2* boards. Alternatively, and this may be added in the future you could use the hostnames as defined in your /etc/hosts file.

Now edit your HPL.dat to look **exactly** like the following:

```
HPLinpack benchmark input file
Innovative Computing Laboratory, University of Tennessee
HPL.out      output file name (if any)
6            device out (6=stdout,7=stderr,file)
1            # of problems sizes (N)
29304         Ns
1            # of NBs
4           NBs
0            PMAP process mapping (0=Row-,1=Column-major)
1            # of process grids (P x Q)
2            Ps
4            Qs
16.0         threshold
1            # of panel fact
2            PFACTs (0=left, 1=Crout, 2=Right)
1            # of recursive stopping criterium
4            NBMINs (>= 1)
1            # of panels in recursion
2            NDIVs
1            # of recursive panel fact.
1            RFACTs (0=left, 1=Crout, 2=Right)
1            # of broadcast
1            BCASTs (0=1rg,1=1rM,2=2rg,3=2rM,4=Lng,5=LnM)
1            # of lookahead depth
1            DEPTHs (>=0)
2            SWAP (0=bin-exch,1=long,2=mix)
64           swapping threshold
0            L1 in (0=transposed,1=no-transposed) form
0            U  in (0=transposed,1=no-transposed) form
1            Equilibration (0=no,1=yes)
8            memory alignment in double (> 0)
##### This line (no. 32) is ignored (it serves as a separator). ######
0                               Number of additional problem sizes for PTRANS
1200 10000 30000                values of N
0                               number of additional blocking sizes for PTRANS
40 9 8 13 13 20 16 32 64        values of NB
```

Now you can run HPL!! You can do so with the following from the folder with your configuration files and xhpl executable:

```bash
$ mpirun -n 8 --hostfile hostfile ./xhpl
```

The run will likely take quite a long time but you will see some output that looks like the following:

```
================================================================================
HPLinpack 2.3  --  High-Performance Linpack benchmark  --   December 2, 2018
Written by A. Petitet and R. Clint Whaley,  Innovative Computing Laboratory, UTK
Modified by Piotr Luszczek, Innovative Computing Laboratory, UTK
Modified by Julien Langou, University of Colorado Denver
================================================================================

An explanation of the input/output parameters follows:
T/V    : Wall time / encoded variant.
N      : The order of the coefficient matrix A.
NB     : The partitioning blocking factor.
P      : The number of process rows.
Q      : The number of process columns.
Time   : Time in seconds to solve the linear system.
Gflops : Rate of execution for solving the linear system.

The following parameter values will be used:

N      :   29304
NB     :       4
PMAP   : Row-major process mapping
P      :       2
Q      :       4
PFACT  :   Right
NBMIN  :       4
NDIV   :       2
RFACT  :   Crout
BCAST  :  1ringM
DEPTH  :       1
SWAP   : Mix (threshold = 64)
L1     : transposed form
U      : transposed form
EQUIL  : yes
ALIGN  : 8 double precision words

--------------------------------------------------------------------------------

- The matrix A is randomly generated for each test.
- The following scaled residual check will be computed:
      ||Ax-b||_oo / ( eps * ( || x ||_oo * || A ||_oo + || b ||_oo ) * N )
- The relative machine precision (eps) is taken to be               1.110223e-16
- Computational tests pass if scaled residuals are less than                16.0

================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       29304     4     2     4            6160.86             2.7232e+00
HPL_pdgesv() start time Tue Aug 13 11:02:04 2019

HPL_pdgesv() end time   Tue Aug 13 12:44:45 2019

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   3.73306719e-03 ...... PASSED
================================================================================

Finished      1 tests with the following results:
              1 tests completed and passed residual checks,
              0 tests completed and failed residual checks,
              0 tests skipped because of illegal input values.
--------------------------------------------------------------------------------

End of Tests.
================================================================================
```

The relevant output you should care the most about is the "Gflops" column. This tells you the theoretical maximum HPL measured on your system. For comparison, to even get onto the Top 500 list, you now have to get into the Pflops values.
