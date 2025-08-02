**Author:** Zawad Hossain
**Location:** Canada, Ontario, Mississauga
**Date Created:** Saturday, August 2, 2025 at 11:51 AM EDT
**License:** All rights reserved.

# Znsession Lab Setup: A Guided Walkthrough and Reference

This document serves as a step-by-step, beginner-friendly guide to setting up a virtual network environment using Oracle VM VirtualBox. It explains every action we performed, from creating virtual networks to configuring servers and resolving common issues, so that a non-technical user can follow along and understand the purpose of each step. This version has been updated to include a detailed breakdown of the Domain Controller setup and replication process.

### **Table of Contents**

1.  [Installing the Virtual Machines](https://www.google.com/search?q=%231-installing-the-virtual-machines)

2.  [Lab Environment Overview](https://www.google.com/search?q=%232-lab-environment-overview)

3.  [Phase 1: VirtualBox Network Configuration](https://www.google.com/search?q=%233-phase-1-virtualbox-network-configuration)

4.  [Phase 2: Virtual Machine Setup and Initial IP Configuration](https://www.google.com/search?q=%234-phase-2-virtual-machine-setup-and-initial-ip-configuration)

5.  [Phase 3: Core Server Role Installation](https://www.google.com/search?q=%235-phase-3-core-server-role-installation)

6.  [Phase 4: Final Verification and Conclusion](https://www.google.com/search?q=%236-phase-4-final-verification-and-conclusion)

7.  [Phase 5: Detailed Troubleshooting](https://www.google.com/search?q=%237-phase-5-detailed-troubleshooting)

### **1. Installing the Virtual Machines**

Before we can do anything with networking or servers, we need to create the virtual computers themselves. This process is similar for both Windows Server and Windows 11.

#### **Step-by-Step VM Creation**

1.  **Open VirtualBox and Click "New"**: On the main VirtualBox screen, find the `New` button and click it to start the virtual machine creation wizard.

2.  **Name the Virtual Machine and Select OS**:

    * **Name**: Give your virtual machine a descriptive name (e.g., `SERVER1A`, `ROUTERVM`, `Windows 11 Client`).

    * **Folder**: Choose where you want to save the VM's files.

    * **ISO Image**: Click the down arrow and then `Other` to browse to the Windows Server 2019 or Windows 11 `.iso` file on your computer. VirtualBox will automatically detect the OS Type and Version for you.

3.  **Hardware Allocation**:

    * **Base Memory**: Allocate a suitable amount of RAM. For Windows Server, 2GB (2048 MB) is a good start. For Windows 11, 4GB (4096 MB) or more is recommended for better performance.

    * **Processors**: Allocate 2 or more CPU cores to the virtual machine for better speed.

4.  **Virtual Hard Disk**:

    * The wizard will ask you to create a virtual hard disk. Accept the default `Create a virtual hard disk now` and click `Create`.

    * The default size of 50GB is usually sufficient for our lab.

    * Choose the `VDI (VirtualBox Disk Image)` file type and `Dynamically allocated` to save space on your physical hard drive.

5.  **Finish and Start the VM**: After clicking `Finish`, you will see your new virtual machine listed. Select it and click `Start` to begin the operating system installation.

#### **OS-Specific Installation Details**

* **Windows Server 2019**:

    * When the setup starts, choose your language and keyboard layout.

    * Click `Install now`.

    * You will be asked for a product key. You can skip this for now.

    * Choose the **`Desktop Experience`** version. This is the one with the graphical user interface, which is much easier for beginners to work with than the command-line-only version.

    * Accept the license terms.

    * Choose `Custom: Install Windows only (advanced)`.

    * Select the virtual hard drive you created and click `Next`.

    * The installation will begin. When it's done, you will be prompted to create an Administrator password. **Remember this password!**

* **Windows 11**:

    * **Important Note on VirtualBox Installation:** After selecting your Windows 11 ISO image, you'll see a checkbox that says "Skip Unattended Installation."
        * **Why you should check this box:** If this box is unchecked, VirtualBox will try to automate the installation process by pre-filling details like the username and password. By checking the box, you are telling VirtualBox to disable this automation and let you manually go through every installation prompt yourself, which is what we need to do for this lab.

    * When the setup starts, choose your language and keyboard layout.

    * Click `Install now`.

    * You will be asked for a product key. You can skip this.

    * Choose the version of Windows 11 you want to install (e.g., Pro, Home).

    * Accept the license terms.

    * Choose `Custom: Install Windows only (advanced)`.

    * Select the virtual hard drive and click `Next`.

    * Windows 11 will prompt you to create a user account and password during the setup process.

#### **Troubleshooting Tip: Bypassing the Windows 11 Microsoft Account Requirement**

During the initial Windows 11 setup (OOBE), the system often requires you to sign in with a Microsoft account. If you need to create a local account or join a domain, you can bypass this in one of two ways:

1.  **Join a Domain**: On the screen that asks for your Microsoft account, you may see an option to "Join a domain instead" or something similar. This will allow you to create a local account and proceed with the setup.

2.  **Use the Command Prompt**: If the domain join option isn't available, press `SHIFT + F10` to open a Command Prompt window. Type the command `oobe\bypassnro` and press Enter. The machine will restart, and when you get to the network connectivity screen again, you will have a new option: "I don't have internet." Selecting this will let you continue with the setup and create a local user account.

### **2. Lab Environment Overview**

Our goal was to create a mini-corporate network inside VirtualBox. We wanted to have two separate network segments connected by a central router, with special servers called "Domain Controllers" to manage everything.

* **Network Segments:**

    * **`WAN1` (Corporate Network):** This is our main network with the address range `192.168.132.0` to `192.168.132.255`. This network can connect to the internet.

    * **`LAN2` (Branch Office Network):** This is a separate, private network with the address range `192.168.10.0` to `192.168.10.255`. It needs to connect to the main network through our router.

* **Virtual Machines:**

    * **`SERVER1A` & `SERVER1B`**: These are the main servers for our domain, `znsession.com`, located on the `WAN1` network.

    * **`ROUTERVM`**: This server acts like a real-world router, connecting our two networks together and providing internet access to the `LAN2` network.

    * **`SERVER2A`**: This is a server on the `LAN2` network. Its job is to eventually become another Domain Controller to help manage that network segment.

    * **`CLIENT1A` & `CLIENT2A`**: These are two Windows 11 client machines, one for each network, that we will join to our domain.

### **3. Phase 1: VirtualBox Network Configuration**

Before we created any servers, we first had to create our virtual networks inside VirtualBox.

1.  **Creating the `WAN1` Network (NatNetwork)**

    * Open VirtualBox.

    * In the menu, go to `File` > `Host Network Manager`.

    * Click on the `NatNetwork` tab and then click `Create`.

    * Click `Properties` for the new network.

    * **Network Name**: We named it `WAN1 NatNetwork`.

    * **Network CIDR**: We set this to `192.168.132.0/24`. This defines the range of IP addresses for this network.

    * **Gateway**: The gateway was automatically set to `192.168.132.1`. This will be the address that connects our servers to the internet.

    * **DHCP Server**: We unchecked the box to disable the DHCP server because we wanted to assign IP addresses manually for better control.

2.  **Creating the `LAN2` Network (Host-Only Adapter)**

    * In the `Host Network Manager`, click on the `Host-only Networks` tab.

    * Click `Create` to add a new network.

    * Click `Properties` for the new network.

    * **Network Name**: We named this `LAN2 Host-Only`.

    * **Adapter Tab**: The `IPv4 Address` was set to `192.168.10.1`, and the `IPv4 Network Mask` was `255.255.255.0`. This defines our second network.

    * **DHCP Server Tab**: We disabled the DHCP server here as well.

### **4. Phase 2: Virtual Machine Setup and Initial IP Configuration**

Now we created each of our servers and gave them their first IP addresses.

#### **`SERVER1A` and `SERVER1B` Configuration**

* **Network Adapter**: For both servers, we went to their settings in VirtualBox, then to the `Network` section, and set Adapter 1 to `NAT Network`. We then selected the `WAN1 NatNetwork` we created.

* **IP Configuration (Static)**:

    1.  On each server, we opened `Control Panel` > `Network and Sharing Center` > `Change adapter settings`.

    2.  We right-clicked the network adapter, chose `Properties`, and then double-clicked `Internet Protocol Version 4 (TCP/IPv4)`.

    3.  **For `SERVER1A`**: We entered **IP address `192.168.132.130`**, Subnet mask `255.255.255.0`, and **Default gateway `192.168.132.1`**. We set the **Preferred DNS server to `192.168.132.130`** (itself) and the **Alternate DNS server to `192.168.132.131`** (the other DC).

    4.  **For `SERVER1B`**: We did the same, but with **IP address `192.168.132.131`**, and the DNS servers pointing to `192.168.132.131` (itself) and `192.168.132.130` (the other DC).

#### **`ROUTERVM` Configuration**

* **Network Adapter 1 (WAN1)**: We set this adapter to the `WAN1 NatNetwork`.

    * **IP Configuration**: We set the static IP to `192.168.132.100`, Gateway to `192.168.132.1`, and DNS to a public server like `8.8.8.8`.

* **Network Adapter 2 (LAN2)**: We set this adapter to the `LAN2 Host-Only` network.

    * **IP Configuration**: We set the static IP to `192.168.10.1`, but we **left the Default Gateway blank**. This is because this server itself is the gateway for this network. We also set its DNS to `8.8.8.8`.

#### **`SERVER2A` Configuration**

* **Network Adapter**: We connected this VM to the `LAN2 Host-Only` adapter.

* **IP Configuration (Static)**:

    * We set its static IP to `192.168.10.132`.

    * **Default Gateway**: This was set to `192.168.10.1`, which is the IP of `ROUTERVM`'s LAN2 network card.

    * **Preferred DNS**: This was set to `192.168.132.130` (`SERVER1A`).

    * **Alternate DNS**: This was set to `192.168.132.131` (`SERVER1B`).

#### **Windows 11 Client Configuration**

* **`CLIENT1A` (on the WAN1 network)**

    * **Network Adapter**: Set Adapter 1 to `NAT Network` and select the `WAN1 NatNetwork`.

    * **IP Configuration**:

        1.  On the Windows 11 desktop, right-click the network icon in the bottom-right corner and select `Open Network & Internet settings`.

        2.  Click on `Ethernet` and then `Edit` next to IP assignment.

        3.  Set it to **`Manual`**.

        4.  Toggle on `IPv4`.

        5.  **IP address**: `192.168.132.200`

        6.  **Subnet mask**: `255.255.255.0`

        7.  **Gateway**: `192.168.132.1`

        8.  **Preferred DNS**: `192.168.132.130` (our primary domain controller)

        9.  **Alternate DNS**: `192.168.132.131` (our secondary domain controller)
           *This client will now be able to communicate with the servers on the main network.*

* **`CLIENT2A` (on the LAN2 network)**

    * **Network Adapter**: Set Adapter 1 to `Host-only Adapter` and select the `LAN2 Host-Only` network.

    * **IP Configuration**:

        1.  Follow the same steps as above to get to the IP assignment settings.

        2.  Set it to **`Manual`**.

        3.  Toggle on `IPv4`.

        4.  **IP address**: `192.168.10.200`

        5.  **Subnet mask**: `255.255.255.0`

        6.  **Gateway**: `192.168.10.1` (the router's address on this network)

        7.  **Preferred DNS**: `192.168.132.130` (our primary domain controller on the other network)

        8.  **Alternate DNS**: `192.168.132.131` (our secondary domain controller on the other network)
           *This client can now reach the servers and the internet through the `ROUTERVM`.*

### **5. Phase 3: Core Server Role Installation**

This is where we gave our servers their jobs.

1.  **Installing and Configuring RRAS on `ROUTERVM`**:

    * We opened `Server Manager`, clicked `Add Roles and Features`, and installed `Network Policy and Access Services`.

    * After installation, we opened `Routing and Remote Access` from the `Tools` menu.

    * We right-clicked the server name and chose `Configure and Enable Routing and Remote Access`.

    * In the wizard, we selected `Network address translation (NAT)` and chose our `WAN1` interface (`192.168.132.100`) as the public interface. This told the router to share its internet connection with the `LAN2` network.

2.  **Detailed Steps to Promote `SERVER1A` and `SERVER1B` to Domain Controllers**:

    * **Install the AD DS Role**:

        1.  On `SERVER1A` (or `SERVER1B`), open `Server Manager` and click `Add Roles and Features`.

        2.  Click `Next` until you get to the `Server Roles` screen.

        3.  Check the box next to `Active Directory Domain Services`. When prompted to add the required management features, click `Add Features`.

        4.  Click `Next` through the remaining screens and then `Install`.

    * **Promote the Server to a DC**:

        1.  After the role installation is complete, a yellow notification flag will appear in `Server Manager`. Click the flag and then click `Promote this server to a domain controller`.

        2.  In the wizard, select the option **`Add a new forest`**. This is because `znsession.com` is a brand-new domain.

        3.  For the `Root domain name`, enter **`znsession.com`**.

        4.  Click `Next`.

        5.  On the `Domain Controller Options` screen, check the boxes for `Domain Name System (DNS) server` and `Global Catalog (GC)`.

        6.  For the `Directory Services Restore Mode (DSRM)` password, enter a strong password (and remember it!). This password is used for a special recovery mode.

        7.  Click `Next` through the remaining wizard pages, accepting the default options.

        8.  Click `Install` to begin the promotion process. The server will restart automatically.

    * **Adding the Second DC (`SERVER1B`)**: We repeated these steps on `SERVER1B` but with one key difference. In the promotion wizard, we selected **`Add a domain controller to an existing domain`** and specified `znsession.com` and the credentials of the domain administrator we just created.

#### **Joining and Promoting `SERVER2A`**

* **Once the initial domain controllers are set up**, we were ready to join `SERVER2A` to the domain.

    1.  We logged into `SERVER2A` and, in `Server Manager` > `Local Server`, we clicked the `Workgroup` link.

    2.  We clicked `Change...`, selected `Domain`, and typed `znsession.com`.

    3.  After providing the `znsession.com` Administrator password, `SERVER2A` joined the domain and restarted.

    4.  We then installed the `Active Directory Domain Services` role and promoted `SERVER2A` to be an additional Domain Controller.

    5.  **Verification**: After `SERVER2A` has restarted, log into `SERVER1A` (or `SERVER1B`), open `Active Directory Users and Computers`, and expand the `Domain Controllers` organizational unit. `SERVER2A` should be listed here, confirming its successful promotion.

#### **Joining Client Machines to the Domain**

* **`CLIENT1A` (on the WAN1 network)**

    1.  On `CLIENT1A`, right-click the Start menu and select `System`.

    2.  Under `Device specifications`, click `Rename this PC (Advanced)`.

    3.  In the `System Properties` window, click the `Change...` button.

    4.  Under `Member of`, select `Domain` and type `znsession.com`.

    5.  Enter the credentials for the domain administrator when prompted and click `OK`.

    6.  The computer will prompt you to restart. Click `Restart Now`.

* **`CLIENT2A` (on the LAN2 network)**

    1.  Follow the same steps as for `CLIENT1A`. The DNS settings we configured earlier will allow it to find the domain controllers.

* **Verification**:

    1.  After the clients have restarted, log into `SERVER1A` (or `SERVER1B`).

    2.  Open `Active Directory Users and Computers` (ADUC).

    3.  Expand the `znsession.com` domain and navigate to the `Computers` organizational unit.

    4.  Both `CLIENT1A` and `CLIENT2A` should be listed here, confirming they have successfully joined the domain.

### **3. Verifying and Testing Active Directory Replication**:

* **Check AD Replication Status**:

    1.  On `SERVER1A`, open `Command Prompt` as an Administrator.

    2.  Run the command: `repadmin /showrepl`

    3.  This command will show the status of Active Directory replication between `SERVER1A` and `SERVER1B`. You should see `SUCCESSFUL` for both servers.

* **Test AD Object Replication**:

    1.  On `SERVER1A`, open `Active Directory Users and Computers` (ADUC).

    2.  Create a new test user or group.

    3.  On `SERVER1B`, open ADUC and refresh the view. The new user or group should appear, confirming that the replication is working correctly.

* **Test DNS Replication**:

    1.  On `SERVER1A`, open `DNS Manager`.

    2.  Expand `Forward Lookup Zones` > `znsession.com`.

    3.  Create a new Host (A) record (e.g., `testpc.znsession.com` pointing to `192.168.10.200`).

    4.  On `SERVER1B`, open `DNS Manager` and refresh the `znsession.com` zone. The new A record should appear, confirming DNS zone replication.

### **6. Phase 4: Final Verification and Conclusion**

To confirm everything was working, we did one final round of tests.

* **Test Ping Connectivity Between All Machines**: A fundamental test for any network is to ensure all machines can communicate. On a machine like `CLIENT1A`, open Command Prompt and try to `ping` the IP addresses of every other machine in the lab (`SERVER1A`, `SERVER1B`, `ROUTERVM`, `SERVER2A`, and `CLIENT2A`). If all pings are successful, your network routing is working correctly.

* **Test Internet Connectivity**: A basic check to ensure all machines can reach the outside world.

    * We successfully pinged a public internet IP (`8.8.8.8`).

    * We were able to browse the internet from all machines.

* **Test DNS Resolution**: A test to ensure our domain controllers are working.

    * `nslookup znsession.com` correctly found our domain controllers.

This confirmed that our entire virtual network was running perfectly, and our servers were all in their correct roles. Our lab is now complete and ready for the next steps!

#### **Troubleshooting Connectivity Issues**

If you find that the machines are not able to ping each other or access the internet, here are some common issues and troubleshooting steps:

* **Ping Failures**: If you can't ping between machines, the issue is often a firewall.

    * **Check Firewalls**: Ensure that the Windows Defender Firewall is either disabled for testing purposes or has an inbound rule to allow ICMP (ping) traffic, as detailed in **Phase 4, Problem 2**.

    * **Verify IP Configuration**: Double-check the static IP addresses, subnet masks, and default gateways on all machines to ensure they are correct for their respective networks (`WAN1` or `LAN2`).

* **No Internet Access**: If a machine can ping its gateway but cannot reach public IPs like `8.8.8.8`, the problem is likely with the router's configuration.

    * **Review RRAS Setup**: Revisit the steps in **Phase 4, Problem 1** to ensure the `ROUTERVM`'s RRAS configuration is correct. The `WAN1` network card must be set as the public interface with NAT enabled, and the `LAN2` network card must be set as a private interface.

* **DNS Resolution Issues**: If you can ping by IP address but not by domain name (e.g., `ping znsession.com` fails), the DNS settings are the most likely culprit.

    * **Check DNS Settings**: On all domain-joined machines (`SERVER2A`, `CLIENT1A`, `CLIENT2A`), make sure the preferred and alternate DNS servers are set to the IP addresses of `SERVER1A` (`192.168.132.130`) and `SERVER1B` (`192.168.132.131`).

### **7. Phase 5: Detailed Troubleshooting**

This was the most important phase. We encountered problems and had to carefully fix them to make the network work.

#### **Problem 1: `SERVER2A` could not connect to `SERVER1A` or the Internet**

* **The Issue**: Our first try at configuring RRAS didn't work. Traffic from `SERVER2A` couldn't get past `ROUTERVM` even though its IP addresses seemed correct. We later found out the router's network cards were incorrectly labeled.

* **The Fix**:

    1.  **On `ROUTERVM`**, open `Routing and Remote Access`.

    2.  In the left pane, expand `IPv4` > `NAT`.

    3.  Right-click the **`WAN1` interface** and select `Properties`.

    4.  On the `General` tab, make sure the box that says **`Public interface connected to the Internet`** is checked. Also, `Enable NAT on this interface` should be checked.

    5.  Now, find the **`LAN2` interface** and right-click it, then select `Properties`.

    6.  On the `General` tab, make sure the box that says **`Private interface connected to private network`** is checked. `Enable NAT on this interface` should be **unchecked**.

    7.  Finally, we right-clicked `ROUTERVM (local)` at the top of the menu and chose `All Tasks` > `Restart` to apply the changes.

#### **Problem 2: One-way Ping Issue (Router couldn't ping Server)**

* **The Issue**: After fixing the router, `SERVER2A` could finally connect to everything. However, when we tried to ping `SERVER2A` *from* `ROUTERVM`, it didn't work. This meant `SERVER2A`'s firewall was blocking pings from the router.

* **The Fix**: We needed to create a new firewall rule on `SERVER2A` to allow pings.

    1.  **On `SERVER2A`**, open `Windows Defender Firewall with Advanced Security` (you can search for it in the Start Menu).

    2.  In the left pane, click on `Inbound Rules`.

    3.  In the right pane, click `New Rule...`.

    4.  In the wizard, follow these steps:

        * **Rule Type**: Choose `Custom`.

        * **Program**: Choose `All programs`.

        * **Protocol and Ports**: Set `Protocol type` to `ICMPv4`. Click the `Customize` button, select `Specific ICMP types`, and check the box for `Echo Request (Type 8, Code 0)`.

        * **Scope**: Leave both IP address sections set to `Any IP address`.

        * **Action**: Choose `Allow the connection`.

        * **Profile**: Make sure all three boxes (`Domain`, `Private`, and `Public`) are checked.

        * **Name**: Give the rule a clear name like `Allow Ping Inbound`.

        * Click `Finish`.
            
