# Cyber-Range - Work in Progress
A walkthrough for building a virtualized security lab, including hypervisor setup, attacker and victim machines, a virtual firewall, and a full detection/monitoring stack.

# Network Information

## IP's for my devices

PfSense – 192.168.1.1

SecurityOnion – 192.168.1.100

Kali – 192.168.1.101

Metasploitable2 – 192.168.1.102

Ubuntu – 192.168.1.103

Remnux – 192.168.1.104

Windows Server – 192.168.1.110

Windows 11 – 192.168.1.111

## Network Diagram

<p align="center">
  <img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/86e19f08-e313-4d2c-8187-a758bb469ab4" />
</p>


# Home Security Lab Setup Guide

A walkthrough for building a virtualized security lab, including hypervisor setup, attacker and victim machines, a virtual firewall, and a full detection/monitoring stack.

# VMware Workstation Pro

1. Go to the Broadcom support portal: https://support.broadcom.com/group/ecx/my-dashboard
2. Create an account.
3. Navigate to **My Downloads**.
4. Click the **Free Software Downloads** link.
5. Select **VMware Workstation Pro** and download.

# pfSense

### Download and Prepare
1. Go to https://www.pfsense.org/download/ and select the **AMD64 ISO** image.
2. Add it to your cart, check out, create a free account, and follow the instructions to download.
3. Download and install 7-Zip (64-bit) from https://www.7-zip.org/ to extract the pfSense image.

### Import and Configure the VM
1. Import the extracted image into VMware Workstation Pro.
2. Add a second network adapter to the VM.
3. Create a dedicated LAN segment for the internal network.
4. Increase the VM's resources to at least 2 GB RAM and 2 CPUs, or installation will fail.
5. Start the VM and press Enter to begin installation.
6. Choose **Install**.
7. Assign `eth0` to WAN and `eth1` to LAN.
8. Note the DHCP subnet range for the LAN (e.g. `192.168.1.100–199`).
9. Install the Community Edition and continue through the remaining prompts.

### Initial Access
1. Once installation completes, switch to a different VM to reach the web interface, making sure that VM is connected to the internal LAN network.
2. Navigate to the pfSense IP address, `192.168.1.1`, and accept the certificate warning.
3. Log in with the default credentials: username `admin`, password `pfsense`.
4. Complete the setup wizard: set the time zone, then change the admin password.
5. Disable DHCP if it is not needed.

### Firewall Rules
1. Create an alias for common internet ports: `53, 80, 443`.
2. On the **WAN** interface, deny all inbound traffic unless a service (such as a web server) needs to be reachable from outside.
3. On the **LAN** interface:
   - Keep the anti-lockout rule enabled to avoid losing access to the pfSense web interface.
   - Delete the default "allow all" rule, since it is unsafe.
   - Create a rule allowing internet access using the TCP/UDP alias created above.
4. Test internet connectivity before and after applying the rules to confirm they work as expected.

### VMware Tools
1. Go to **System → Package Manager** and install the **vmtools** package.
2. Restart pfSense. This is needed to support graceful shutdowns from the hypervisor.



# Kali Linux

### Download and Import
1. Go to https://www.kali.org/get-kali/#kali-virtual-machines
2. Click the VMware download button.
3. Create a dedicated folder for VMs and set it as your default preferences location.
4. Create a subfolder for Kali Linux.
5. Extract the downloaded archive into that folder.
6. Create the VM in VMware Workstation Pro.

### Configure a Static IP
1. Edit the network interfaces file:
   ```
   sudo nano /etc/network/interfaces
   ```
2. Set a static address, for example:
   ```
   sudo ifconfig eth0 192.168.1.X netmask 255.255.255.0
   ```
3. Edit the resolver configuration:
   ```
   sudo nano /etc/resolv.conf
   ```
4. Save with `Ctrl + O`, then exit with `Ctrl + X`.
5. Restart networking:
   ```
   sudo systemctl restart networking
   ```


# Metasploitable2

1. Go to https://www.rapid7.com/products/metasploit/metasploitable/
2. Click **Download Now**.
3. Create a folder and extract the downloaded archive.


# REMnux

1. Go to https://docs.remnux.org/install-distro/get-virtual-appliance
2. Download the OVA file.
3. Right-click the file and choose **Open in VMware**.
4. Add a network adapter to the VM.
5. Mount the CD-ROM device if needed:
   ```
   sudo mkdir /media/cdrom
   sudo mount /media/cdrom /media/cdrom/
   ```


# Ubuntu 64-bit

1. Go to https://ubuntu.com/download/desktop and download the installer.
2. After installation, install the VMware tools package:
   ```
   sudo apt-get install open-vm-tools-desktop
   ```


# Windows 11

1. Go to https://www.microsoft.com/en-us/software-download/windows11
2. Download and install the VM as usual.
3. Configure a static IP address.


# Windows Server 2022

1. Go to https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022
2. Download and install the VM.
3. Configure a static IP address.

### Active Directory Domain Services

1. Open **Server Manager** and go to **Local Server → Manage → Add Roles and Features**.
2. Click **Next**, choose **Role-based or feature-based installation**, then select **This Server**.
3. Select **Active Directory Domain Services**, add the required features, and click **Next**, then **Install**.
4. Promote the server to a domain controller, choosing **Add a new forest**, and enter a domain name.
5. Continue through the remaining prompts, then install and restart the server.

### DNS Setup for Active Directory

1. Open the DNS management console.
2. Right-click **Reverse Lookup Zones** and create a new **Primary Zone** for `192.168.1.x`.
3. Right-click the domain zone (e.g. `marshal.local`) and select **Update PTR Record**.

### Joining a Client to the Domain

1. On the client Windows PC, search for **Domain or Workgroup**.
2. Click **Change**, then enter the domain name to join it.


# Security Onion

Security Onion serves as the defender-side counterpart to Kali Linux, providing network monitoring and detection.

### Download and Import
1. Go to https://github.com/Security-Onion-Solutions/securityonion/blob/2.4/main/DOWNLOAD_AND_VERIFY_ISO.md
2. Review the hardware minimums before proceeding.
3. Import the ISO into VMware Workstation Pro with the following specs:
   - 200 GB of storage
   - Single disk image
   - 8 GB of RAM
   - 2 processors, 4 total cores
   - Attached to the LAN segment
4. Finish the import and power on the VM.

### Installation
1. Choose a standard (non-desktop) installation, since a GUI is not required.
2. Set up the local account.
3. Once setup completes, press Enter repeatedly through the remaining prompts and wait for installation to finish.

### Web Console Access
1. Log in to the web console using the configured admin account.
2. Confirm access to the dashboard.

### Elastic Fleet: Adding pfSense Logs
1. Go to **Agent Policies → so-grid-nodes_general → Add Integration → pfSense**, then add the integration and set the syslog host, saving and continuing.
2. Under **Administration → Configuration → Advanced Options → Firewall → Host Groups**, add a custom host group for `192.168.1.1`.
3. Under **Port Groups**, add a custom port group for `UDP:9001`.
4. Under **Roles → Standalone → Chain → Input**, attach the custom host group and the custom port group.
5. Synchronize the grid to apply the changes.
6. Back on pfSense, go to **Status → System Logs → Settings**, enable **Remote Logging**, select the LAN interface, choose to log everything, and save.
7. Create a firewall rule allowing UDP traffic on port 9001.

### Adding Wazuh Agents

**Security Onion side:**
1. Go to **Administration → Configuration → Add Wazuh Agents** and add the address of each endpoint to be monitored.

**Windows endpoint side:**
1. Download and install the Wazuh agent using the Windows GUI installer.
2. Download the Sysmon Modular configuration and the Sysmon binary.
3. Open PowerShell and install Sysmon with the configuration applied:
   ```
   .\sysmon64.exe -accepteula -i .\sysmonconfig.xml
   ```
4. The Wazuh agent configuration file is located at:
   ```
   C:\Program Files (x86)\ossec-agent\ossec.conf
   ```


# Network Intrusion Detection (Suricata / ET Rules)

### Install Suricata on pfSense
1. On pfSense, go to **System → Package Manager → Available Packages**.
2. Find **Suricata** and install it.

### Configure Rule Sources
1. Go to **Services → Suricata → Global Settings**.
2. Enable the **ET Open** and **Snort Community** rule sets.
3. Enable **Hide Deprecated Rules**.
4. Locate the ET rules download URL from the provider's instructions page and paste it into the appropriate field.
5. Update and download the rule sets; this can take approximately 15 minutes.

### Interface Configuration
1. Go to **Interfaces** and configure the LAN interface settings (LAN traffic matters most for internal detection, versus WAN).
2. Enable **TLS logging**.
3. Leave **File Store** and **Packet Store** disabled unless needed, since they consume significant disk space, which matters especially in a VM.
4. Leave **Block Offenders** disabled until the rule set is confirmed to be working correctly.
5. Under **Categories**, select all relevant categories for the LAN interface.
6. Review and enable the resulting LAN rule set.
7. Return to **Interfaces** and start Suricata using the play button.

### Verification
1. Monitor resource usage on pfSense using `top`.
2. Power on Kali Linux and generate traffic to trigger an alert, for example by launching `msfconsole` and running an exploit against `vsftpd`.
3. Run `whoami` to confirm the session context.
4. Review the resulting Suricata alerts, suppressing any that are confirmed to be self-generated test traffic.
