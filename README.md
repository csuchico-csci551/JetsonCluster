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
  * [Blog about Barrel Connector](https://desertbot.io/blog/jetson-nano-power-supply-barrel-vs-micro-usb) - There is a discussion here about why you would want a barrel connector and the fact you can get 5V/4A... this might be highly recommended if you are going to keep a monitor, keyboard, mouse connected and get a fan powered by the board.
* Optional Case - There are a few cases for the boards on Amazon, a few come with active cooling fans, which might be useful especially if you are going to move/carry the boards around or need active cooling.
* Optional Switch - Depending on your home network, you may need additional wired connections for the Jetson Nano board. I recommend getting an inexpensive gigabit switch for this. You optionally could run your own network off or your laptop through the switch if you don't have access to your router or want to use the boards on campus too. You want gigabit as that is the max the boards support and likely will be a limiting factor in our MPI applications on this cluster regardless.
  * [TRENDnet 5 port switch](https://smile.amazon.com/TRENDnet-Unmanaged-Gigabit-GREENnet-TEG-S5G/dp/B002HH0W5W/) - Found this cheap 5 port Gigabit switch on Amazon for $14.99


### Getting Started With Jetson Nano Developer Kit

Once you get your Jetson Nano board, follow the [Getting Started With Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit) instructions up through the *Setup and First Boot* step. At this point your Jetson Nano should have the OS installed with an initial user account. We now need to do a few more initial steps before we can use Ansible or move into the steps to build the cluster manually.

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


## Step-by-Step Cluster Instructions

**TODO Add the steps**

## Ansible instructions

**TODO Add the Ansible playbooks and instructions**
