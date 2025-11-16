# SOC-Automation-Project-2.0
A Hands-on guide to building AI-driven SOC workflows with Splunk and n8n. Learn to set up VMs, ingest telemetry, create alerts, and automate responses to streamline detection and incident response.

## 📖 Overview
This project is inspired by the *“SOC Automation Project 2.0: How To Use AI in Your SOC Workflow”* video from MyDFIR. It provides scripts, documentation, and examples to help security professionals and learners replicate the project in their own environments.


## 🔑 Features
- **Splunk Setup & Telemetry**: Configure VMs, install Splunk, and send system telemetry for centralized monitoring.  
- **Alert Creation**: Build Splunk alerts that detect suspicious activity and trigger automated responses.  
- **Workflow Automation with n8n**: Connect Splunk alerts to automation pipelines, reducing manual triage.  
- **AI Integration**: Explore how AI can enhance SOC efficiency by analyzing alerts and prioritizing incidents.

## 🧰 Prerequisites
Before you begin, download the following tools and images:

### Hypervisor
- **VMware Workstation Pro** (or **VirtualBox**)

### OS Images
- **Windows 10 ISO**: From the Microsoft Download Page (using the *Create Windows 10 installation media* tool).  
- **Ubuntu Server ISO**: From the Ubuntu Website (*v24.04 used in this lab*).  

### Software
- **Splunk Enterprise**: The Linux `.deb` file from the Splunk Free Trials Page.  
- **Splunk Universal Forwarder**: The Windows `.msi` (64-bit) from the Splunk Downloads Page.

## Part 1: VM Setup

We're going to create 3 virtual machines: Windows 10, Splunk(SIEM), and n8n(SOAR). 
## 🖥️ Virtual Machine Configuration

| VM Name            | Role             | OS             | CPUs | RAM   | Disk Size        |
|--------------------|------------------|----------------|------|-------|------------------|
| Windows-10         | Victim/Endpoint  | Windows 10     | 2    | 4 GB  | 60 GB (default)  |
| Splunk             | SIEM             | Ubuntu Server  | 2    | 8 GB  | 100 GB           |
| N8N-VM             | SOAR/Automation  | Ubuntu Server  | 2    | 4 GB  | 50 GB            |


## ⚙️ VM Creation Steps

For each virtual machine:

1. Follow your hypervisor's **"New Virtual Machine" wizard**.  
2. Point the installer to the corresponding ISO (Windows or Ubuntu).  
3. Assign the resources as specified in the [VM Configuration Table](#🖥️-virtual-machine-configuration).  
4. Set the **Network Type** to **NAT** for all VMs.  
5. Power on the VMs to begin OS installation.

### A. Windows 10 (Windows-10)
1. Follow the Windows setup wizard.  
2. When prompted, select:
   - **"I don't have a product key."**
   - **Windows 10 Pro**
   - **Custom: Install Windows only**
3. During the initial setup (OOBE):
   - Set up for personal use → Offline account → Limited experience.  
   - Create a local user (e.g., `mydfir`) and set a password.  
   - Decline all privacy and telemetry options.  
4. Once at the desktop, enable Remote Desktop:
   - Start → *Remote desktop settings* → Toggle **Enable Remote Desktop** to **On**.  
5. Open `cmd` and get the VM’s IP address: ipconfig

Note: remember to take a Snapshot of the VM

### B. Ubuntu Servers (Splunk & N8N-VM)
The setup is identical for both Ubuntu VMs.
1. 	Follow the Ubuntu Server setup wizard.
2. 	Keep defaults for language, keyboard, and network (DHCP).
3. 	For storage, select Use an entire disk.
4. 	Set up your user (e.g., Name: , Server Name: , Username: ).
5. 	CRITICAL: When prompted, check the box to Install OpenSSH server.
6. 	Wait for installation to finish and reboot.
7. 	Log in to the console and get the IP address: ip a
8. 	From your host machine, SSH into the VM and run system updates: sudo apt-get update && sudo apt-get upgrade -y
9. 	Repeat steps 7–8 for the N8N-VM.

Note: Keep a record of all VM IP addresses.


## 🔬 Part 2: Splunk Installation & Configuration

### 1. Download and Install Splunk
1. 	On your host machine, go to the Splunk Enterprise download page and copy the  link for the  file.
2. 	In your SSH session for the mydfir-splunk VM, download the file: wget -O splunk.deb "https://download.splunk.com/..." (Remember your link will be different)
3. 	Install the package: sudo dpkg -i splunk.deb
4. 	Start Splunk and accept the license: sudo /opt/splunk/bin/splunk start
5.  Enable Splunk to start on boot: sudo /opt/splunk/bin/splunk enable boot-start -user splunk

### 2. Configure Splunk GUI
On your host browser, go to http://your-splunk-vm-ip:8000 and Log in with the credentials you just created.

### A. Enable Data Receiving
1. Go to Settings > Forwarding & receiving.
2. Under "Receive data," click Configure receiving.
3. Click New Receiving Port, enter 9997, and click Save.

 <img width="653" height="639" alt="image" src="https://github.com/user-attachments/assets/84b5ae48-dad3-4dcd-9614-9ae7d2dc7ff8" />


### B. Create a New Index
1. Go to Settings > Indexes.
2. Click New Index.
3. Set Index Name to mydfir-project and click Save.

### C. Install Windows Add-on
1. Go to Apps > Find More Apps.
2. Search for Windows event and install the Splunk Add-on for Microsoft Windows.
3. You will need to enter your splunk.com credentials (not your server credentials) to install.

<img width="615" height="311" alt="image" src="https://github.com/user-attachments/assets/bc52d03c-a67f-41d2-bbf1-1f1d5365b039" />


Finally, take a snapshot of the Splunk VM and name it Splunk-installed.


## 📮 Part 3: Send Windows Telemetry to Splunk
These steps connect the Windows 10 VM to your Splunk instance.

### Part 1: Install Splunk Universal Forwarder
1. 	Copy the Splunk Universal Forwarder  file from your host to the Windows 10 VM.
2. 	Run the installer inside the Windows 10 VM.
3. 	Follow the setup wizard:
   - Accept the license agreement.
   - Select Splunk Enterprise (on-premise).
   - Username:  (local user for the forwarder). Set a password.
   - Deployment Server: Leave blank.
   - Receiving Indexer:
     -	Host: Your mydfir-splunk VM IP
     -	Port: 9997
  - Click Next and Install.

### Part 2: Configure Data Inputs
1. On the Windows 10 VM, navigate to C:\Program Files\SplunkUniversalForwarder\etc\system\local
2. Open notepad and create a new file named inputs.conf
3. Head over to 
