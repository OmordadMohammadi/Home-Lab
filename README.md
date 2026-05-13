# Home-Lab
I'm studying for CompTIA Network+ and wanted to actually build something instead of just reading about it. This is my home lab — a physical server I built, configured, and documented from scratch.

What Is This Project?
This document tells the complete story of how I built a personal IT lab server from scratch, from inspecting the hardware, to installing and securing a Linux operating system, to connecting to it remotely from another computer.

A "home lab" is a personal computer set up to behave like the servers used in real companies. Instead of just reading about how servers work, I built one, configured it, broke things, fixed them, and learned hands-on. Every decision made along the way is explained here — not just what I did, but why.

The Hardware

Component                 Specification                             Notes
Motherboar            MSI Z170-A PRO MS-7971               LGA1151 socket, supports 6th/7th Gen Intel
CPU                   Intel Core i7-7700K @ 4.20 GHz       Quad-core, capable for lab workloads
RAM                   GB DDR4 2400MHz 2×8GB Corsair        Dual-channel configuration
Storage primary       Samsung 870 EVO 500GB SSD            installed ubuntu server here
Storage secondary     Crucial CT120M 120GB SSD             Available for future use
Operating System      Ubuntu Server 24.04.4 LTS            Noble Numbat




Phase 1 — Hardware Inspection and Fixes
What I Did?
Before installing any software, I opened the machine and inspected the hardware. Two problems I found immediately.
Problem 1: RAM Was in the Wrong Slots
What that means for a non-technical reader:
A computer's memory, RAM (random access memory) works fastest when its sticks are paired correctly — like how a car engine runs better with cylinders firing in the right order. This is called "dual-channel." The two RAM sticks were plugged into Slots next to each other(slots 3 and 4), which forces them to run in single-channel, essentially cutting memory performance almost in half.

The fix:
I moved the sticks to slots 2 and 4 (DIMM2 and DIMM4 on the MSI Z170-A PRO). These are the correct slots for dual-channel operation on this motherboard. Verified in BIOS under Overclocking MEMORY-Z, which confirmed both sticks are recognised on separate channels.
Why it matters?
Dual-channel memory doubles the bandwidth available to the CPU. For a server running multiple services at same time, this is meaningful. It also showed that attention to detail in physical hardware setup — not just software — is part of the job.

Problem 2: The BIOS Was Nearly 10 Years Out of Date
What that means for a non-technical reader:
BIOS = Basic Input/Output System, is the firmware that wakes up a computer before the operating system loads. It controls how all the hardware components start up and communicate before the operating system takes over.This machine's BIOS was from December 2016 — almost a decade old. Manufacturers release BIOS updates to fix security vulnerabilities, improve hardware compatibility, and fix bugs. Running outdated firmware is a security and stability risk.

The fix:
I downloaded the latest BIOS version (E7971IMS.1K0, build date 07/10/2018) from MSI's official website
Copied the file to the root of a FAT32-formatted USB drive
Used the motherboard's built-in M-FLASH utility to flash the new firmware
Waited about 3 minutes without touching anything — interrupting a BIOS flash can permanently damage the motherboard.

Result:
BIOS updated from build date 12/21/2016 to 07/10/2018. ✅
Important note:
The BIOS update reset all custom settings, including XMP (the memory speed profile). After updating, XMP had to be re-enabled manually under Overclocking settings to restore RAM to its rated 2400MHz speed.

Phase 2 — Choosing the Operating System
Why Ubuntu Server (Not Windows, Not Desktop)
Three options were considered:
Windows — No. because requires a paid licence. More importantly, the Swedish IT job market is Linux-heavy. Practising on Windows would build the wrong muscle memory for the roles I am targeting.
Ubuntu Desktop — rejected. because comes with a graphical interface (windows, icons, mouse-driven). Real servers don't have graphical interfaces — administrators connect remotely via terminal and type commands. Using a desktop environment would be practising the wrong skills.
Ubuntu Server — chosen. No graphical interface. Everything is done through a text-based terminal. This forces genuine skill-building. learning to navigate, configure, and troubleshoot using only commands. This is how production servers in companies, cloud platforms (Azure, AWS), and data centres operate.

Ubuntu 24.04 LTS over 26.04 LTS:
Ubuntu 26.04 LTS had just been released. While newer is sometimes better, a brand new operating system release often has early bugs that take months to fix. Ubuntu 24.04.4 LTS is battle-tested, has extensive documentation, and every tutorial and Stack Overflow answer works with it. For a study environment, stability matters more than cutting edge.

Phase 3 — Creating the Bootable USB
What That Means
Installing an operating system requires booting the computer from a USB drive that contains the OS installer — like a digital version of the installation disks computers used to come with.

Steps
Downloaded ubuntu-24.04.4-live-server-amd64.iso from ubuntu.com (3.2GB)
Verified the SHA256 checksum to confirm the file wasn't corrupted in transit — a basic security habit
Used Rufus 4.14 to write the ISO to a USB drive
 .Partition scheme: GPT
 .Target system: UEFI
 .Mode: DD Image

The problem I faced: BIOS/Legacy Boot Error
First boot attempt showed: ERROR: BIOS/LEGACY BOOT OF UEFI-ONLY MEDIA
What that means:
The USB was created for UEFI (modern) boot mode, but the BIOS was set to boot in Legacy (old) mode. These two modes are incompatible.
Fix:
Enabled UEFI boot mode in the BIOS under Settings → Advanced → Windows OS Configuration. Rebooted and the installer loaded correctly.

Phase 4 — Installing Ubuntu Server
The Installation Process
The Ubuntu Server installer walks through a series of configuration screens. Key decisions made at each step:
Installation type:
Selected "Ubuntu Server" (full), not "Ubuntu Server (minimized)." The minimized version strips out tools and utilities to save space — useful for automated containerized environments, not for a learning lab where you need those tools.

Network:
The installer detected the Realtek Ethernet controller automatically and received an IP address via DHCP (***.***.*.**). Confirmed network connectivity before proceeding.

Storage:
Selected the Samsung 870 EVO 500GB SSD as the target drive
Chose "Use an entire disk" — no dual-boot required
Enabled LVM (Logical Volume Manager)
What is LVM? Think of LVM as a flexible partition system. Instead of committing to fixed partition sizes at install time, LVM lets you resize storage volumes later with a single command. It is standard practice on production servers.

Encryption: Not enabled. For a home lab this would mean a mandatory passphrase on every boot with no remote recovery option — unnecessary complexity at this stage.

Profile setup:
server hostname  lab-server-01  
Username   ********
Password used a Strong password (write it down and put somewhere offline)

Why lab-server-01? 
Professionals using server even other admin understand. In real environments, servers follow structured naming schemes so teams can identify machines at a glance. Using this convention from day one builds the habit.

OpenSSH:
Installed during setup. This is the software that allows remote terminal access over the network. Without it, you must physically sit at the server keyboard for every task — that is not how servers are managed in practice.
Featured snaps:
None selected. Servers should run only what is needed. Every additional installed package is a potential attack surface. Remove unsed packages close used port for securty.

Phase 5 — First Boot and System Updates
Logging In
After installation and reboot, the server presented a login prompt:
Ubuntu 24.04.4 LTS lab-server-01 tty1
lab-server-01 login:
Logged in with username and password set during install.
System Information Confirmed
OS:           Ubuntu 24.04.4 LTS (Noble Numbat)
Kernel:       Linux 6.8.0-111-generic x86_64
Storage:      97.87GB available
Memory usage: 1% (practically nothing running yet)
IP address:   192.168.*.** (DHCP)
Updating the System
The first command run on any new Linux server:
bash sudo apt update && sudo apt upgrade -y

What this does: apt is Ubuntu's package manager — the tool that installs, updates, and removes software. update refreshes the list of available packages. upgrade installs all available updates. The -y flag automatically confirms all prompts. This cleared 37 pending updates.

Setting the Timezone
in bash sudo timedatectl set-timezone Europe/Stockholm
Servers log every action with a timestamp. Incorrect timezone makes logs misleading and troubleshooting harder.

Phase 6 — Configuring a Static IP Address
Why Static IP?
For a non-technical reader:
By default, a router assigns IP addresses dynamically — the address your device gets today might be different tomorrow. That works fine for a laptop or phone. For a server that you connect to remotely, it is a problem. If the address changes, you lose access until you figure out the new one.
A static (fixed) IP address means the server always has the same address on the network, every time.
How Ubuntu Manages Network Configuration: Netplan
Ubuntu Server uses a tool called Netplan for network configuration. Configuration is written in YAML format — a structured text file.
Configuration File
Location: /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    enp3s0:
      dhcp4: no
      addresses:
        - 192.168.*.**/24
      routes:
        - to: default
          via: 192.168.*.*
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
Breaking this down for a non-technical reader:

Setting                Value                         What It Means
enp3s0            Network interface name              The name Linux gave to the Ethernet port  
dhcp4:no          Disabled                            Don't request an automatic address use what we specify
addresses         192.168.*.**                        This server's fixed IP address
Via               192.168.*.*                         The router's address — all internet traffic goes through here
nameservers       8.8.8.8                              DNS servers — translates website names into IP addresses (Google and Cloudflare)
                  1.1.1.1                               
                  

Applying the Configuration
In bash is used command, sudo chmod 600 /etc/netplan/00-installer-config.yaml  # Fix file permissions
sudo netplan apply                                      # Apply the new settings
Verification
bash ip a show enp3s0
ping -c 4 google.com
Result: Server now has permanent address 192.168.*.**  on the local network. ✅

Phase 7 — Remote Access via SSH
What Is SSH?
For a non-technical reader:
SSH (Secure Shell) is an encrypted tunnel between two computers. It lets you type commands on your laptop that run on a server in another room — or another country. It is the standard way all server administration is done professionally. Every command sent through SSH is encrypted so nobody can see or intercept it.

First Remote Connection (from laptop)
bash ssh myname@192.168.*.**
The first connection asks you to verify the server's fingerprint — a unique identifier that confirms you're connecting to the right machine. Accepted and added to known hosts.
Result: Connected to lab-server-01 from the laptop without physical access. ✅

Phase 8 — SSH Hardening (Key-Based Authentication)
The Problem with Passwords
Password authentication over SSH has a critical weakness: passwords can be brute-forced. An attacker can try thousands of password combinations automatically until one works.
SSH key authentication remove this risk entirely.
How SSH Keys Work? (Non-Technical Explanation)
Think of it like a padlock and key system:
You generate a key pair — a public key and a private key
The public key goes on the server (like a padlock installed on a door)
The private key stays on your laptop (like the physical key in your pocket)
When you connect, your laptop proves it has the matching private key — without ever sending it across the network
No private key = no access, regardless of any password

Generating the Key Pair (on the laptop — Git Bash on Windows)
bash ssh-keygen -t ed25519 -C "lab-key"

ed25519 is the algorithm used — modern, fast, and more secure than the older RSA algorithm. The key pair is saved to ~/.ssh/id_ed25519 (private) and ~/.ssh/id_ed25519.pub (public).

Installing the Public Key on the Server
bash ssh-copy-id myname@192.168.*.**
This command connects to the server and adds the public key to ~/.ssh/authorized_keys, the list of keys the server will accept.
Disabling Password Authentication
Once key-based login was confirmed working, password authentication was disabled entirely. Even if someone knows the username and password, they cannot log in without the private key.
Edit /etc/ssh/sshd_config:

PasswordAuthentication no
PermitRootLogin no
MaxAuthTries 3

Setting                         Value           Why
PasswordAuthentication no      Disabled        Eliminates brute-force attack vector
PermitRootLogin no             Disabled        The root account has unlimited system access — never expose it directly
MaxAuthTries 3                 3 attempts       Limits how many times someone can try before being disconnected

Apply changes:
bash sudo systemctl restart ssh
Result: Server now only accepts connections from devices with the correct private key. ✅

Phase 9 — Firewall Configuration (UFW)
What Is a Firewall?
For a non-technical reader:
A firewall is a set of rules that controls which network traffic is allowed in and out of a computer. Without a firewall, every port (think of ports as doors into the computer) is potentially accessible. A firewall closes all doors except the ones you explicitly decide to open.
UFW — Uncomplicated Firewall
Ubuntu ships with UFW, a straightforward firewall tool that wraps Ubuntu's underlying iptables system in simple commands.
Configuration Applied
sudo ufw default deny incoming    # Block ALL incoming traffic by default
sudo ufw default allow outgoing                # Allow all outgoing traffic
sudo ufw allow 22/tcp                          # Open port 22 for SSH
sudo ufw enable                                # Activate the firewall

Final Firewall Status
Status: active
Default: deny (incoming), allow (outgoing), disabled (routed)
To          Action    From
--          ------    ----
22/tcp      ALLOW IN  Anywhere
22/tcp (v6) ALLOW IN  Anywhere (v6)
Result: Firewall active. All ports closed except SSH. ✅

What Was Built — Summary
Starting from a bare machine with a near-decade-old BIOS and incorrectly seated RAM, the following was completed in a single session:

Task                                                   Status
RAM dual-channel configuration verified                ✅
BIOS updated from 2016 to latest version               ✅
Ubuntu Server 24.04.4 LTS installed                    ✅
System fully updated (37 packages)                     ✅
Timezone configured (Europe/Stockholm)                 ✅
Static IP address configured (192.168.*.**)            ✅
Remote SSH access established                          ✅
SSH key-based authentication configured                ✅
Password authentication disabled                       ✅
UFW firewall enabled and configured                    ✅

Skills Demonstrated
Hardware
Physical server inspection and fault identification
RAM channel configuration
BIOS firmware update procedure (M-FLASH)
UEFI vs Legacy boot mode understanding

Linux Administration.
Ubuntu Server installation and configuration
Package management (apt)
File system navigation and editing (nano, file paths)
Service management (systemctl)
Network interface management (ip, ping)

Networking
DHCP vs static IP addressing
Subnet masks and CIDR notation (/24)
Default gateway configuration
DNS server configuration
Network interface identification

Security
SSH key-based authentication (ed25519)
Firewall configuration and rule management (UFW)
Principle of least privilege (no root login, password auth disabled)
File permission management (chmod)

Tools Used
Git Bash (Windows SSH client)
Rufus (bootable USB creation)
GNU nano (terminal text editor)
Netplan (Ubuntu network configuration)

(Next Steps
This server is the foundation. What gets built on top of it next:
 Set up SSH key on additional devices
 Re-enable XMP in BIOS (RAM speed restoration after flash)
 Deploy Apache web server and configure virtual hosts
 Configure BIND9 DNS server
 Set up isc-dhcp-server and study DORA process
 Install and configure network monitoring tools (nmap, tcpdump)
 Practice log analysis (journalctl, /var/log/)
 Set up automated security updates)


Environment Reference
Server hostname:   lab-server-01
Operating system:  Ubuntu Server 24.04.4 LTS (Noble Numbat)
Kernel:            Linux 6.8.0-111-generic x86_64
Static IP:         192.168.*.**/24
Gateway:           192.168.*.*
DNS:               8.8.8.8, 1.1.1.1
Network interface: enp3s0
MAC address:       **:**:**:**:**:**
SSH port:          22
Auth method:       ED25519 key pair (no password)
Firewall:          UFW active — deny all incoming except port 22

Built and documented as part of an ongoing home lab project for IT infrastructure study and CompTIA Network+ certification preparation. All configuration performed manually to build genuine understanding of each component.
Last updated:
