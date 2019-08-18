# Jetson Cluster
Repo for instructions on setting up a micro compute cluster with the [NVIDIA Jetson Nano](https://developer.nvidia.com/embedded/jetson-nano-developer-kit) boards and potentially Ansible playbooks for configuration and setup.

## Initial Steps

These steps will be needed to get the board initially up and working regardless. Once you have gotten the OS/system initially setup we'll move on to either step by step instructions or how to use the Ansible playbooks to get your cluster setup. I will recommend the Ansible approach unless you have experience with Linux sysadmin as it hopefully will simplify getting started for you.

### Hardware

The first step would be to purchase a [NVIDIA Jetson Nano](https://developer.nvidia.com/embedded/jetson-nano-developer-kit) board and accessories.

* [NVIDIA Jetson Nano Board](https://www.amazon.com/NVIDIA-Jetson-Nano-Developer-Kit/dp/B07PZHBDKT/) - Amazon link, costs the same from all the other retailers.
* [MicroSDXC Card](https://www.amazon.com/Samsung-MicroSDXC-Adapter-MB-ME128GA-AM/dp/B06XWZWYVP/) - Amazon Link to a $19.99 128GB Samsung UHS Speed Class U3 card
* Power Supplies - The board doesn't come with a power supply and can take both a micro-USB or a barrel connector power supply.
  * [Heyday USB Wall Charger](https://www.target.com/p/heyday-153-usb-wall-charger/-/A-53454899) - Found this at Target for $4.99, since I have multiple USB-A to micro-USB cables and this supports 5V/2.3A it can adequately power the Jetson board.
  * [Adafruit Power Supply](https://www.adafruit.com/product/1995) - This is the recommended Micro USB supply that NVIDIA lists.
  * [CanaKit Raspberry Pi Power Supply](https://smile.amazon.com/CanaKit-Raspberry-Supply-Adapter-Listed/dp/B00MARDJZ4) - This is als a 5V/2.5A RPi Supply like the Adafruit one and should work for the Jetson boards. It's on Amazon for $9.99 with free shipping. 
  * [Blog about Barrel Connector](https://desertbot.io/blog/jetson-nano-power-supply-barrel-vs-micro-usb) - There is a discussion here about why you would want a barrel connector and the fact you can get 5V/4A... this might be highly recommended if you are going to keep a monitor, keyboard, mouse connected and get a fan powered by the board.
* Optional Case - There are a few cases for the boards on Amazon, a few come with active cooling fans, which might be useful especially if you are going to move/carry the boards around or need active cooling.
* Optional Switch - Depending on your home network, you may need additional wired connections for the Jetson Nano board. I recommend getting an inexpensive gigabit switch for this. You optionally could run your own network off or your laptop through the switch if you don't have access to your router or want to use the boards on campus too. You want gigabit as that is the max the boards support and likely will be a limiting factor in our MPI applications on this cluster regardless.
  * [TRENDnet 5 port switch](https://smile.amazon.com/TRENDnet-Unmanaged-Gigabit-GREENnet-TEG-S5G/dp/B002HH0W5W/) - Found this cheap 5 port Gigabit switch on Amazon for $14.99


### Getting Started With Jetson Nano Developer Kit

Once you get your Jetson Nano board, follow the [Getting Started With Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit) instructions up through the *Setup and First Boot* step. At this point your Jetson Nano should have the OS installed with an initial user account. We now need to do a few more initial steps before we can use Ansible or move into the steps to build the cluster manually.

* I would recommend setting the hostnames at this step, I named mine *jn1* and *jn2*.

### Create second user account (Optional)

If you are working in a team, you should at this point setup an account for your partner who didn't create the initial user account in the last step. If you are working on your own you can skip this step.

```bash
# Replace <username> with the username of the new user
$ sudo adduser <username>
```

Now you should follow the prompts to set the new user password and user information. The user information isn't overly necessary. Once you've successfully added the new user to the system you should next give it access to sudo so that they can also administer the system if necessary. The following command is the simple way to do this as it will add the new user to the sudoers group.

```bash
# Replace <username> with the username of the new user
$ sudo usermod -aG sudo <username>
```

### Statically assign IP to Jetson Boards

We need the Jetson boards to exist at a persistent ip on our network. Most consumer routers allow you to do this through DHCP static mappings of media access control (MAC) address to a static ip for the network. Normally your DHCP network ranges are the upper range of bits, so like last digit being *100-254* and you'll want to use a static range outside that range. If you need help here, see me in office hours or post to Piazza.

1. The first step will be finding your mac address, you can do that via the *ip* command like follows:

```bash
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: dummy0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 0a:5b:89:29:39:ef brd ff:ff:ff:ff:ff:ff
3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:04:4b:e6:22:b6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.129/24 brd 10.100.100.255 scope global dynamic noprefixroute eth0
       valid_lft 4834sec preferred_lft 4834sec
    inet6 fe80::3ac:3ac6:1fc4:e0f3/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
4: l4tbr0: <BROADCAST,MULTICAST> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether be:88:ea:8e:d3:79 brd ff:ff:ff:ff:ff:ff
    inet 192.168.55.1/24 brd 192.168.55.255 scope global l4tbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::1/128 scope link tentative
       valid_lft forever preferred_lft forever
5: rndis0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast master l4tbr0 state DOWN group default qlen 1000
    link/ether be:88:ea:8e:d3:79 brd ff:ff:ff:ff:ff:ff
6: usb0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast master l4tbr0 state DOWN group default qlen 1000
    link/ether be:88:ea:8e:d3:7b brd ff:ff:ff:ff:ff:ff
```

This is the full list of addresses on the system. What you should pay attention to is the device *eth0*, which in this case is the 3rd adapter listed. And in the output below that adpater you'll see a line that looks like this *link/ether 00:04:4b:e6:22:b6*, the key is the *00:04:4b:e6:22:b6*, which is the MAC address for this physical adapter. This is what we'll need to statically assign an ip to this board when it's wired in via it's gigabit ethernet adapter.

2. Now update your router/network to statically assign each Jetson board its own unique static ip address. I recommend putting them close to each other as it'll make things easier. So with my example above I might assign each board 192.168.1.2 and 192.168.1.3, assuming nothing else needed those static addresses already on your network. For my specific network I'm now up in the 20s. Record and keep track of these static addresses as you'll need them later.

3. Once you've assigned the static address on your router, you'll need to refresh the lease on your Jetson board or restart your board so it asks for its new lease. I personally just restarted the machine, which you can always do with this command:

```bash
$ sudo reboot
```

But alternatively you may be able to restart the network controller as well with the following:

```bash
$ sudo service networking restart
```

### SSH via SSH Keys to each boards

If you already have ssh public/private keys, you can move on to the next step. If your id_rsa ssh keys, the default ones, are password protected, this will be a problem for use with Ansible. There is likely a way to use a secondary key so come see me in office hours if you need help here.

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

This will generate new ssh key, for example I ran this on one of my Jetson Boards for the above example. I recommend hitting enter for all of the prompts as the defaults and no passphrase is recommended for this application. Do not do this if you already have a SSH key pair.

#### Copy key to each board

Now we'll want to copy the SSH keys to all of the Jetson Nano boards. You will need to do the following to each of the Jetson Nano ip addresses. You may want to SSH to each board via password first to remove the complication of it prompting about if you want to connect during the key transfer:

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

Now you should be able to connect via ssh to the machine without password. Check that this works before moving on for all of your Jetson Nano boards.

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

This will install ansible and any other requirements into the virtual environment you have currently active. Now we need to update your inventory to match your Jetson Nano boards ips on your network. I provided you an example inventory file called *inventory*, we will be manually running this inventory vs putting the config files into the system to have ansible run by default. Here is the contents of the example Ansible inventory file:

```
#Adjust these to your actual ips for each board.

[jetson_nano_primary] #ip/Inventory for your primary Jetson board
jn1 ansible_host=192.168.1.2 ansible_python_interpreter="/usr/bin/python3"

[jetson_nano_worker] #ips/Inventory for your worker Jetson boards, if you have additional boards add them here.
jn2 ansible_host=192.168.1.3 ansible_python_interpreter="/usr/bin/python3"

[jetson_nano:children]
jetson_nano_primary
jetson_nano_worker

[jetson_nano:vars]
ansible_ssh_user=<username> #Replace <username> with the actual user you want to run commands on the Jetson Board
```

I gave the group of machines the name *jetson_nano*, which includes the children group for the primary and worker Jetson boards broken out. Replace your two boards in the inventory with the correct ip addresses. Additionally, in the last line there is the variable *ansible_ssh_user* to define what user you want the scripts to SSH into the board and run the commands as.

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

Now that you have ansible installed and working with your inventory you can use the playbooks to setup/configure your Jetson Nano boards to work as a cluster or for development. You can run the playbooks in the following order or just run playbook all to install everything.

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
  * **Caveat** - This works but only installs Rust onto the primary Jetson Nano board, so all compilation and development will need to happen there for Rust. This is likely what you will want to do regardless but due to the way Rust adds itself to the path and into your home directory it wasn't trivial to install on all the nodes with the shared home folder. If I'm able to resolve this in the future will update but for now this will work.
  * If you choose this playbook will install Rust onto the Jetson Nano boards
  * This may be useful if you want to play with rust, including implementing the MPI assignments for extra credit in rust with the experimental MPI bindings.
* **clang.yml**
  * **Optional Playbook**
  * If you choose, this will install Clang 7 onto your cluster
  * Clang is a llvm compiler and may provide a lot of novel compiler features that GCC doesn't by default.
  * This playbook isn't included in the *all* playbook, as its not necessary and the OpenMP included with clang is OpenMP 3.1 instead of OpenMP 4.5 that comes with GCC

You can run any of the playbooks with the following command structure:

```bash
#replace <playbook file> with the actual ansible playbook to run
$ ansible-playbook -i inventory --ask-become-pass <playbook file>
```

For example to run the *all* playbook that would run all of the playbooks for you in one command you can do the following:

```bash
$ ansible-playbook -i inventory --ask-become-pass all.yml
```

**Note:** the *--ask-become-pass* option will prompt you for the sudo password for the *ansible_ssh_user* on the Jetson Nano boards. This is to prevent you from hardcoding this somewhere, but provide privileges to the Ansible scripts to do all the installs.

### MPI Run Issues

There is a chance that you may get an odd error that looks like the following when you try to run MPI tasks:

```bash
[jn1][[39096,1],0][btl_tcp_endpoint.c:649:mca_btl_tcp_endpoint_recv_connect_ack] received unexpected process identifier [[39096,1],2]
```

If this is the case you'll want specify the tcp interface that communication occurs over in the mpirun command as follows:

```bash
$ mpirun --mca btl_tcp_if_include eth0 <rest of mpirun command>
```

I ran into this on one of the cluster builds after the Ansible playbook ran but not another. Everything still works, just requires a bit more information specified from you as the user to run MPI. 


## Step-by-Step Cluster Instructions

If I find time I will add these instructions, but going to focus on assignments, lectures, and other aspects of this class first.

**TODO Add the steps**

## Linpack Benchmarks

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

In order to run HPL you will need to edit a file called HPL.dat to match your system setup. We will be running HPL at a very small scale to save time as a fully optimized HPL could take a very very long time to complete. Running this small scale HPL took me 1.5 hours to complete as an example. I'm making the assumption you are running this on 2 Jetson Nano boards, if you are using more/different boards you'll have to figure out the details for the configurations yourself or come see me for help.

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
