# Jetson Cluster
Repo for instructions on setting up a micro compute cluster with the [NVIDIA Jetson Nano](https://developer.nvidia.com/embedded/jetson-nano-developer-kit) boards and potentially Ansible playbooks for configuration and setup.

## Initial Steps

These steps will be needed to get the board initially up and working regardless. Once you have gotten the OS/system initially setup we'll move on to either step by step instructions or how to use the Ansible playbooks to get your cluster setup. I will recommend the Ansible approach unless you have experience with Linux sysadmin as it hopefully will simplify getting started for you.

### Hardware

The first step would be to purchase a [NVIDIA Jetson Nano](https://developer.nvidia.com/embedded/jetson-nano-developer-kit) board and accessories.

* [NVIDIA Jetson Nano Board](https://www.amazon.com/NVIDIA-Jetson-Nano-Developer-Kit/dp/B07PZHBDKT/) - Amazon Link
* [MicroSDXC Card](https://www.amazon.com/Samsung-MicroSDXC-Adapter-MB-ME128GA-AM/dp/B06XWZWYVP/) - Amazon Link to a $19.99 128GB Samsung UHS Speed Class U3 card
* Power Supplies - The board doesn't come with a power supply and can take both a micro-USB or a barrel connector power supply.
  * [Heyday USB Wall Charger](https://www.target.com/p/heyday-153-usb-wall-charger/-/A-53454899) - Found this at Target for $4.99, since I have multiple USB-A to micro-USB cables and this supports 5V/2.3A it can adequately power the Jetson board.
  * [Adafruit Power Supply](https://www.adafruit.com/product/1995) - This is the recommended Micro USB supply that NVIDIA lists.
  * [Blog about Barrel Connector](https://desertbot.io/blog/jetson-nano-power-supply-barrel-vs-micro-usb) - There is a discussion here about why you would want a barrel connector and the fact you can get 5V/4A... this might be highly recommended if you are going to keep a monitor, keyboard, mouse connected and get a fan powered by the board.
* Optional Case - There are a few cases for the boards on Amazon, a few come with active cooling fans, which might be useful especially if you are going to move/carry the boards around or need active cooling.  

## Step-by-Step Instructions

**TODO Add the steps**

## Ansible instructions

**TODO Add the Ansible playbooks and instructions**
