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


### 1: Install Splunk Universal Forwarder
1. 	Copy the Splunk Universal Forwarder file from your host to the Windows 10 VM.
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

### 2: Configure Data Inputs
1. On the Windows 10 VM, navigate to C:\Program Files\SplunkUniversalForwarder\etc\system\local
2. Open notepad and create a new file
3. Head over to [inputs.conf](https://drive.google.com/file/d/1-qYp4oCrT1BqhG1oaprQfkFhgFIHiWEm/view) and copy the entire text to paste it into the notepad file, then save the file as inputs.conf under C:\Program Files\SplunkUniversalForwarder\etc\system\local 
4. Run the Services app as admin (services.msc).
5. Find the SplunkForwarder service:
   - Right-click > Properties > Log On tab.
   - Select Local System account and click Apply.
   - Right-click the service and Restart.

### 3 Verify data in Splunk
1. Go back to your Splunk web interface and click on Search & Reporting.
2. In the search bar, type in index="mydfir-project" and set the time range to Last 24 hours

You should be able to see events from your Windows 10 VM

## 🤖 Part 4: Install n8n (Automation)

### 1: Install Docker and Docker Compose
1. SSH into your N8N-VM using it's ip address
2. Install Docker and Docker Compose using these commands:
   - sudo apt install docker.io -y
   - sudo apt install docker-compose -y

Troubleshooting note- If you get "fail to fetch" errors, you may need to edit your sources list:
- sudo nano /etc/apt/sources.list.d/ubuntu.sources
  - Find any lines with 'ir.archive.ubuntu.com' and remove the 'ir.' prefix.
  - Save (Ctrl+X, Y, Enter), then run 'sudo apt update' and try the installs again.
 
### 2: Creating the n8n Docker Compose File
1. Create a directory for your n8n configuration:
   - mkdir n8n-compose
   - cd n8n-compose
2. Create the docker-compose.yaml file by doing **sudo nano docker-compose.yaml** and then going to this link: [docker-compse.yaml](https://drive.google.com/file/d/1aTPKp5VPLuC7QwdEdx8-qB70iIiTSYkU/view) and copying the entire text into the file

**Important**: Change the N8N_HOST value to your n8n VM's IP address.

3. Save and exit

### 3: Start n8n and Fix Permissions
1. Pull the n8n Docker image and start the container:
   - sudo docker-compose pull
   - sudo docker-compose up -d

**Troubleshooting note**: You may get a "Connection Refused" error in your browser due to a permissions issue. 

To fix this:
  - The 'n8n_data' volume is owned by 'root', but needs to be 'node' (UID 1000) so run:
     - sudo chown -R 1000 n8n_data
  - Restart the container with the correct permissions by running: 
     - sudo docker-compose down
     - sudo docker-compose up -d

2. On your host browser, go to **http://[your-n8n-vm-ip]:5678**
3. You should now see the n8n setup screen. Create your owner account to complete the installation.
4. Take a snapshot of the VM and name it n8n-installed.


## 🚨 Part 5: Create a Splunk Alert

### 1. Generate Brute Force Telemetry
1. From your Host Machine, open a Remote Desktop (RDP) connection to your Windows 10 VM IP and enter a random but incorrect password
2. Repeat this process 5 times to generate multiple Event ID 4625 logs

**Note**: If you are already logged in via RDP, you can just lock the screen and type wrong passwords, or try to RDP from a second window.

### 2. Analyze Logs in Splunk
1. Go to your Splunk Search & Reporting app and run the following query: **index="mydfir-project" EventCode=4625**
2. Verify you see the 5 failed events.
3. Refine the search to display a clean table of the data we need. Update the query to: index="mydfir-project" EventCode=4625 | stats count by _time, ComputerName, user, src_ip

   - _time: When it happened.
   - ComputerName: Which machine was targeted.
   - user: The fake username attempted.
   - src_ip: Where the attack came from.

### 3. Create the Alert
1. With the refined query running, click Save As > Alert.
2. Title: test-brute-force
3. Description: Detects failed login attempts.
4. Alert Type: Scheduled.
5. Run on Cron: Select Run on Cron Schedule and enter the following to run every minute (for testing purposes): * * * * *
6. Trigger Conditions: Trigger when Number of Results is Greater than 0.
7. Trigger Actions:

   - Click + Add Actions > Add to Triggered Alerts.
   - Click + Add Actions > Webhook.

We'll set up the webhook in the next part so leave the "Save As Alert" dialog box open

## 🔗 Part 6: Connect n8n to Splunk

### 1. Configure the n8n Webhook Node
1. Open your n8n interface in your browser (http://[your-n8n-vm-ip]:5678).
2. Click Start from scratch to create a new workflow.
3. Click Add first step and search for Webhook. Select it.
<img width="208" height="181" alt="image" src="https://github.com/user-attachments/assets/b4acabce-617a-4245-915f-ae118ae03c72" />

4. Configuration:

   - HTTP Method: Change this from GET to POST.
   - Path: You can leave this as default.
5. Copy URL: Locate the Test URL (it usually looks like http://.../webhook-test/...) and click to copy it.

<img width="540" height="910" alt="image" src="https://github.com/user-attachments/assets/2fe7f907-960b-438d-b405-3572add2eaaf" />


### 2. Update Splunk Alert
1. Navigate back to your Splunk tab.
2. If you still have the "Save As Alert" dialog open, paste the URL into the Webhook URL field.

**Note**: If you closed it: Go to Settings > Searches, reports, and alerts, find test-brute-force, click Edit > Edit Alert, and scroll down to the Trigger Actions section.

3. Paste the n8n Test URL into the text box and click save.
4. Click View Alert and ensure the status is set to Enabled.

### 3. Verify Data Reception
1. Go back to n8n and click on the Listen for test event button. 

**Note**: you may need to wait for a minute

Once the screen is updated, showing "Workflow executed", you can Expand the JSON or Body section in the Output Data. You should see the telemetry fields we configured in Splunk:

- _time
- ComputerName
- user
- src_ip
- count

## 🧠 Part 7: Integrate OpenAI (ChatGPT)
