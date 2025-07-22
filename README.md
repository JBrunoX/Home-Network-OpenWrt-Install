# Home Network: OpenWrt Edition

Welcome to my guide on setting up a personal home network using **OpenWrt**! Whether you're looking to upgrade your network's performance or gain more control over it, this guide will help you turn any old x86 computer into a custom router. Don’t worry if you’re new to networking or Linux, I’ve added extra prefaces and prerequisites to make this accessible to everyone.

---

## What is OpenWrt and Why Use It?

**OpenWrt** is an open-source operating system designed for routers and embedded devices. It offers advanced features like custom firewall rules, VLANs, VPNs, and more. By installing it on an x86 computer, you get a flexible, powerful router that can handle complex setups and since it will use x86 architecture even a old desktop can do the trick. This guide uses OpenWrt to create a home network with wired and wireless connectivity.

---

## Key Networking Concepts

Before we start, here’s a quick rundown of essential terms:

- **Router**: Connects your devices (LAN) and manages it's traffic to the internet (WAN).
	
- **Modem**: Connects your home network to your Internet Service Provider (ISP) enabling a internet connection.
    
- **Switch**: Adds more Ethernet ports to connect wired devices.
    
- **Access Point (AP)**: Provides Wi-Fi to your network.
    
- **WAN**: Your internet connection from your ISP.
    
- **LAN**: Your private network of devices.
    
- **DHCP**: Assigns IP addresses automatically to devices.
    

These concepts will pop up throughout the guide, so understanding them will make the process smoother.

---

## Prerequisites

To follow this guide, you’ll need:

- **An x86 computer** (e.g., an old PC) with:
    
    - **Storage**: At least 8MB-16MB for OpenWrt.
        
    - **RAM**: 64MB-128MB minimum.
        
    - **Network**: Two Ethernet ports (one for WAN, one for LAN) and a wireless access point or Wi-Fi card.
        
- **A monitor, keyboard, and mouse** for setup.
    
- **A USB drive** (at least 8GB) to install Ubuntu and OpenWrt.
    
- **Internet access** via your ISP’s router (connected to the x86 computer).
    
- **A network switch** (optional, but recommended) to connect multiple devices.
    
- **Basic Linux knowledge**:
    
    - How to open a terminal (e.g., Ctrl + Alt + T).
        
    - Running commands like lsblk (lists drives) or cd (changes directories).
        
    - Don’t worry I'll explain key commands as we go.
        
- **Access to your ISP router’s admin settings** (default username/password).
	
---

## Installation

This section walks you through installing OpenWrt on your x86 computer.

### 1. Prepare a Bootable USB

- **Why?** We’ll use Ubuntu’s live environment to install OpenWrt.
    
- **Steps**:
    
    - Download the latest LTS version of Ubuntu Desktop from the [Ubuntu downloads page](https://ubuntu.com/download/desktop).
        
    - Download a disk image creation tool. I will be using Balena Etcher, but Ventoy is also good and if your on windows Rufus is good as well.
        
    - Insert your USB, open Balena Etcher, click Flash from file and select the Ubuntu download file. Now click Select target and choose your USB drive. Then hit **Flash** and wait for it to finish. Unplug the USB once its done.
        

### 2. Boot into Ubuntu

- Insert the USB into your x86 computer that you will be installing OpenWrt.
    
- Start up the computer and enter the BIOS by repeatedly pressing the delete key, F2, F10 or Esc upon startup (varies among motherboards).
    
- Once in the BIOS, navigate to the boot section. You may or may not have to first go to the advanced section of the BIOS. Once your in the boot section change the boot priority to have your USB drive first. Now you can save your changes and reboot your system. It should now boot into the USB drive.
    
- Choose **Try Ubuntu** from the menu.
    

### 3. Wipe the Internal Drive

- **Why?** I've found that when flashing OpenWrt, the previous OS seems to cling on to the drive. For this reason we will simply wipe the drive.
    
- Once booted into Ubuntu select *Try or Install Ubuntu*. Now open a terminal (Ctrl + Alt + T).
    
- Run `lsblk` to list all your systems drives. Here you will determine which drive you want to install OpenWrt on. It will be something like nvme0n1, sda or sdb. Make sure it is not the USB drive and is your internal drive. Use the drive sizes to help.
    
- Wipe it:
    
    - Run `sudo wipefs -a /dev/<drive_name>` (erases all file system signatures like partition tables and headers).
        
    - Then run `sudo dd if=/dev/zero of=/dev/<device_name> bs 1M count=100` (overwrites the first 100MB of the drive with zeros).
        
    - _Caution: Double-check the drive name to avoid erasing the wrong one!_
        

### 4. Install OpenWrt

- Remaining in the terminal run `firefox` to open the Firefox browser. Go to the OpenWrt [downloads page](https://downloads.openwrt.org/).
    
- Pick the latest stable release > *x86* > *64* > *generic-ext4-combined-efi.img.gz*. Once downloaded it will pop up in the top right. Click it to automatically unzip it. Files will open.
    
- In Files delete the .zip file and keep the .gz file. Now exit Firefox and Files.
    
- In the terminal:
    
    - Run `cd Downloads && ls` to see the name of the file we downloaded.
        
    - Run `sudo dd if=<file_name> of=/dev/<drive_name> bs=1M status=progress`.
        
- Wait for it to finish, power off the system, remove the USB drive when prompted and hit enter.
    

### 5. Configure Network Interfaces

- Turn the system back on and hit OpenWrt when prompted. Once OpenWrt is done initializing it's system hit enter.
    
- Change root password with `passwd` and entering your desired password when prompted.
	
- Run `cat /etc/config/network` to check the current network configuration.
    
- Here are some things to look for:
    
    - We need to make sure we have the correct device assignments. To verify do the following:
	    - Run `ip -br a`. This shows which interfaces are up or down. Unplug the Ethernet cable running from your ISP router to your x86 machine and run the command again. The interface that goes down is the one you unplugged, thus making it your 'wan' port. Do the same for the Ethernet cable running from your network switch to your x86 machine. The port that does down will be your 'br-lan' port.
        
    - **config interface 'lan'** should look something like this. 
			option device 'br-lan'
			option proto 'static'
			option ipaddr '192.168.1.1'
			option netmask '255.255.255.0'
			option ip6assign '60'
	- You can modify DNS to this. It's Cloudflare and Google's DNS. If you have a different DNS you like or your own DNS server you can use those too.
			list dns '1.1.1.1'
			list dns '8.8.8.8'
			
	- **config interface 'wan'** should look something like this. 
			option device 'eth0'
		    option proto 'dhcp'
		    option peerdns '0'
		    list dns '1.1.1.1'
		    list dns '8.8.8.8'
	- Make sure *option device* points to the interface connected to your ISP router.
        
    - Run `vi /etc/config/network` then press `i` to enter insert mode and begin your edits if needed.
	    
    - After making changes exit insert mode by pressing `Esc` and then saving and quitting with `:wq`. Now restart the network with `/etc/init.d/network restart`.
    
- Run `ping google.com` to test for internet connection. You should see replies with no dropped packets.
    

### 6. Setup Root Partition & Root File System Expansion Automatically

- Since OpenWrt was meant for embedded devices we will be expanding our root partition and file system. There is also a known issue where every time our system is updated the expansion goes away. To avoid this we will be configuring a automatic expansion after every update.
	
- Run `opkg update` to update the package repository  then `opkg install parted losetup resize2fs blkid` to install the required packages.
    
- Download and run the expansion script:
    
    - Run `wget -U "" -O expand-root.sh "https://openwrt.org/_export/code/docs/guide-user/advanced/expand_root?codeblock=0"` to download.
        
    - Run `. ./expand-root.sh` to source the script.
        
	- Run `sh /etc/uci-defaults/70-rootpt-resize` to resize the root partition/file system.
		
	- Now the system will reboot twice on it's own.

---

## Configuration

Now, let’s set up your network.

### 1. Enable Bridge Mode on Your ISP Router

- **Why?** This lets OpenWrt handle routing instead of your ISP’s router leaving the ISP router as essentially a modem.
    
- Log in to your ISP routers admin page via it's IP address. You will be prompted login credentials. You can find it's IP and login credentials either on the physical router or by doing a google search for that specific router. 
    
- Once logged in enable **bridge mode**, save and reboot your router.
	

### 2. Set Up WAN

- Access OpenWrt’s web interface (LuCI) at 192.168.1.1 or whatever you changed your IP address too during the edit of `/etc/config/network`. The username will be `root` and the password will be whatever you set yours too when you ran `passwd`. 
    
- Go to **Network** > **Interfaces** > **wan** and hit **edit** on wan.
    
- Click Advanced Settings and if checked, uncheck `Use DNS servers advertised by peer`. Under Use custom DNS servers add `1.1.1.1` first, hit enter and `8.8.8.8` second, hit enter. Then click save. Once again if you have other DNS servers in mind you may use those.
    

### 3. Configure LAN

- Go to **Network** > **Interfaces** > **LAN** > **Edit**.
    
- Match DNS settings from WAN.
    
- If you want to adjust the DHCP server's lease range, go to the **DHCP Server** and modify the **Start** and **Limit** fields. This defines the range of IP addresses the router will assign to connected devices. Avoid including `0`, `1`, or `255` in the range:
	
	- `0` is the network identifier
	    
	- `1` is typically the router’s IP address
	    
	- `255` is the broadcast address
	    
- Example: 192.168.1.50 - 192.168.1.200
	
- Click **Save** after making changes.
    

### 4. Configure DHCP

- Go to **Network** > **DHCP and DNS** > **Static Leases**.
    
- If you wish to add a static lease click **Add**. Here you may change the hostname and IPv4 address as desired. 
	
	- The MAC address will be what OpenWrt recognizes it as. You should see the device in the dropdown. 
		
	- When changing the IPv4 address it must stay within your network. So if your network is 192.168.1.1 the devices IP must be within that. So it must be from 192.168.1.2 - 192.168.1.254.
		
- Repeat for as many devices as needed.
    

### 5. Connect a Wi-Fi Access Point

- Power on your AP/router and plug it into your network switch via Ethernet.
    
- If your device is simply a access point then your all set and can skip these next two steps. If you have a router then follow along. 
	
	- Login to the routers admin page whether its via an app or web page.  
		
	- Set it to **AP mode**. This disables the router’s routing capabilities to prevent double NAT, while keeping its wireless interface active so it can still handle wireless connections to your network.
    
- Login to LucI and check **DHCP and DNS** for a new IP lease to verify your access point have gotten an IP.
    

---

## All Set!

You now have a fully functional OpenWrt network! You can now experiment with firewall rules, VPNs, or other features as you grow comfortable.

---

# Troubleshooting

### 1. No Internet After Configuration

- **Issue**: ping google.com fails.
    
- **Fix**: Verify WAN interface connects to ISP router. Check /etc/config/network for correct wan settings (option proto 'dhcp'). Restart network: /etc/init.d/network restart.
    

### 2. Boot Failure After OpenWrt Install

- **Issue**: System doesn’t boot into OpenWrt after USB removal.
    
- **Fix**: Verify correct drive flashed `lsblk`. Re run dd with correct if/of parameters. Check BIOS boot order prioritizes internal drive after removing the USB.
    

### 3. Network Interface Misconfiguration

- **Issue**: LAN/WAN ports not working.
    
- **Fix**: Run `ip -br a` to confirm interface assignments. Edit /etc/config/network to match WAN (ISP port) and LAN (switch port). Restart network `/etc/init.d/network restart`.
    

### 4. Wi-Fi Access Point Not Detected

- **Issue**: AP not in LuCI DHCP leases.
    
- **Fix**: Ensure AP and network switch are powered on and connected to each other. Reboot AP, verify connectivity to switch. Check LuCI **DHCP and DNS** for lease.
    

### 5. Root Partition Not Expanding

- **Issue**: Root partition reverts after updates.
    
- **Fix**: Run `opkg update`, install `parted losetup resize2fs blkid`. Re-download, execute expand-root.sh. Ensure `/etc/uci-defaults/70-rootpt-resize` runs on boot.
    

### 6. LuCI Web Interface Inaccessible

- **Issue**: Can’t access 192.168.1.1.
    
- **Fix**: Confirm LAN IP in `/etc/config/network`. Ensure device IP is in same subnet (192.168.1.x). Disable firewall: `/etc/init.d/firewall stop`. Use correct root password.
    

### 7. DHCP Not Assigning IPs

- **Issue**: Devices don’t get IPs.
    
- **Fix**: In LuCI, go to **Network > Interfaces > LAN > DHCP Server**, ensure DHCP enabled, range set. Restart LAN interface.

---
