
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

### 13. Enhance Privacy with Unbound  

Now that Pi-hole is blocking ads, let's improve security and privacy by installing **Unbound**. Currently, Pi-hole forwards DNS requests to an upstream provider (Cloudflare in this case). However, some users may prefer not to have third parties tracking their DNS queries, even if Cloudflare doesn't log them.  

**Unbound** allows Pi-hole to directly query the DNS root servers for any uncached Fully Qualified Domain Name (FQDN), adding a layer of privacy.  
By doing this, you'll route DNS queries directly from Pi-hole to the root DNS servers, keeping things more private and secure.Below listed the commands:
- sudo apt install unbound -y
-sudo nano -w /etc/unbound/unbound.conf.d/pi-hole.conf 

Copy and paste the following text below:


server:
    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to no if you don't have IPv6 connectivity
    do-ip6: yes

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    #root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # IP fragmentation is unreliable on the Internet today, and can cause
    # transmission failures when large DNS messages are sent via UDP. Even
    # when fragmentation does work, it may not be secure; it is theoretically
    # possible to spoof parts of a fragmented DNS message, without easy
    # detection at the receiving end. Recently, there was an excellent study
    # >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
    # by Axel Koolhaas, and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
    # in collaboration with NLnet Labs explored DNS using real world data from the
    # the RIPE Atlas probes and the researchers suggested different values for
    # IPv4 and IPv6 and in different scenarios. They advise that servers should
    # be configured to limit DNS messages sent over UDP to a size that will not
    # trigger fragmentation on typical network links. DNS servers can switch
    # from UDP to TCP when a DNS response is too big to fit in this limited
    # buffer size. This value has also been suggested in DNS Flag Day 2020.
    edns-buffer-size: 1232

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes

    # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10

After pasting the configuration into the file (use **SHIFT+INS** to paste in PuTTY), save and exit by pressing:
1. **CTRL+X** to exit.  
2. Press **Y** to confirm saving changes.  
3. Hit **ENTER** to save the file and exit.

- sudo service unbound restart
1. Log into the **Pi-hole Admin GUI**.  
2. Navigate to **Settings** → **DNS**.  

#### Steps:  
- Uncheck the box next to **Cloudflare** (or whichever DNS provider you selected during the installation).  
- Add a new custom DNS entry for **Unbound**.  


  ![image](https://github.com/user-attachments/assets/a64d6376-a1a2-459a-8982-386576d4421d)

  Now, Pi-hole will use Unbound as the upstream DNS provider, ensuring more privacy and security for DNS queries! 🚀  
