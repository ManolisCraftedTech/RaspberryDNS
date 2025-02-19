
## Raspberry Pi 5 - Pi-hole Setup 

### 1. Prepare the microSD Card  
- Download and install **Raspberry Pi Imager** on your computer.  
- Insert a **microSD card** into a card reader (no need to format it manually).  
<img width="347" alt="1" src="https://github.com/user-attachments/assets/c9abe1d0-f138-4c68-9e5f-651bbdc19af6" />


### 2. Flash the OS  
- Open Raspberry Pi Imager.  
- Click **"CHOOSE OS"** and select **"Raspberry Pi OS Lite (64-bit)"** from the list.
<img width="338" alt="2" src="https://github.com/user-attachments/assets/ea99d06a-bdc5-4de3-ae22-b275ef7ea7b1" />  

### 3. Configure Basic Settings  
- Enable **SSH** and select **password authentication** for remote access.  
- Set a **hostname** (e.g., `pihole3.local`) to easily identify the device on the network.  
<img width="240" alt="4" src="https://github.com/user-attachments/assets/c9febbf2-6309-4f26-9784-6c024697ef42" />

### 4. Set Up Login Credentials  
- Choose a **username** (default is `pi`, but you can change it).  
- Set a **strong password** for secure access.  
<img width="350" alt="5" src="https://github.com/user-attachments/assets/0d82e56a-58e9-4715-8c1d-e43b5ea50f50" />

### 7. Alternative SSH Connection (Using PuTTY)  
If you prefer a graphical tool, you can use **PuTTY** to connect via SSH.

<img width="324" alt="image" src="https://github.com/user-attachments/assets/bfaca309-be8f-46a3-953f-b2033ccbc902" />

### 8. Update System Packages  
- Before installing anything, update your Raspberry Pi’s package list to ensure you have the latest versions.  
- sudo apt update && sudo apt upgrade

### 9. Set a Static IP on the Raspberry Pi  
To ensure your Raspberry Pi always uses the same IP address, you can configure a static IP manually.  

#### Edit the DHCP Configuration File  

Open the `dhcpcd.conf` file with the following command:
- sudo nano /etc/dhcpcd.conf

<img width="386" alt="1" src="https://github.com/user-attachments/assets/690b0900-db8e-4ee4-95fe-9e6dd774ef71" />

This allows you to define a static IP address directly on the Raspberry Pi.


### 10. Reboot and Install Pi-hole  
After setting the static IP address or reserving it via DHCP, reboot the Raspberry Pi to apply the network settings:  
- sudo reboot

Once the Raspberry Pi has rebooted and is using the new IP address (or after DHCP reservation), you’re ready to install Pi-hole.

- curl -sSL https://install.pi-hole.net | bash

Follow the on-screen instructions to complete the installation process

<img width="356" alt="2" src="https://github.com/user-attachments/assets/ad65caed-fa52-4043-b2cf-284516813e64" />

<img width="379" alt="3" src="https://github.com/user-attachments/assets/2b1a9b38-5387-4c2f-8b17-a9ecd0339259" />


### 11. Choose an Upstream DNS Provider  
The next step is to select an **upstream DNS provider** for Pi-hole. This is the DNS server Pi-hole will use to perform initial lookups (these will be cached afterward).  

For now, we’ll use **Cloudflare (1.1.1.1)** as the upstream DNS provider
Since we will be setting up **Unbound** later for local DNS resolution 
  

<img width="98" alt="4" src="https://github.com/user-attachments/assets/3967d930-e3da-48bc-9f76-5d2df3ded9ab" />


### 12. Configure Devices to Use Pi-hole  

Now that Pi-hole is set up and blocking ads, we need to configure devices to use it as their DNS server. There are two main ways to do this: manually or via DHCP.  

For **Windows 11**:  
1. Go to **Network & Internet settings**.  
2. Click **‘Change adapter options’** (or the properties of your current network adapter).  
3. Select **‘Edit’** under the IP settings section.  
4. Set the **DNS server** to the IP address of your Pi-hole (or Pi-holes if you’re using multiple for redundancy).  


![image](https://github.com/user-attachments/assets/9a8e3029-5591-4ffc-9993-d31131620492)
