SSH Hardening — Key-Based Authentication

What Is This?

SSH (Secure Shell) is the tool used to connect to the server remotely from another computer.
By default it uses password authentication, which can be brute-forced by attackers.

This document covers how I replaced password authentication with SSH key pairs
and locked down the SSH configuration to reduce the attack surface.

Why Key-Based Authentication?

| Method | Risk |
|--------|------|
| Password auth | Can be brute-forced, attacker tries thousands of passwords automatically
| Key-based auth | Requires the private key file, guessing is mathematically impossible |

Steps

1. Generate a key pair (run on your laptop, not on the server)

On Windows use this command in Git Bash: ssh-keygen -t ed25519 -C "lab-key"
Press Enter through all prompts to accept defaults.

This creates two files:
 `~/.ssh/id_ed25519` — **private key** — never share this, never upload this anywhere
 `~/.ssh/id_ed25519.pub` — **public key** — this goes on the server

ed25519 is the modern algorithm its fast, and more secure than the older RSA standard.

 2. Copy the public key to the server
```bash
ssh-copy-id username@192.168.x.xx 
```

Enter your server password when prompted. The public key is added to
`~/.ssh/authorized_keys` on the server.

3. Test key-based login before disabling passwords
Open a new terminal window and connect:
```bash
ssh username@192.168.x.xx
```

If it connects without asking for a password — keys are working.
**Do not proceed to step 4 until this is confirmed.**

4. Disable password authentication on the server
```bash
sudo nano /etc/ssh/sshd_config
```

Find and change these lines:
```
PasswordAuthentication no
PermitRootLogin no
MaxAuthTries 3
```

### 5. Restart SSH service
```bash
sudo systemctl restart ssh
```

### 6. Test again in a new terminal window
```bash
ssh username@192.168.x.xx
```

Confirms the new config works before closing any existing sessions.

---

sshd_config Changes Reference

| Setting                  | Value | Reason |
| `PasswordAuthentication` | no    | Eliminates brute-force attack vector |
| `PermitRootLogin`        | no    | Root has unlimited access — never expose it directly |
| `MaxAuthTries`           | 3     | Disconnects after 3 failed attempts |

---

Problems Encountered

**ssh-keygen ran on the server instead of the laptop**

The private key must live on the machine you connect FROM — your laptop.
Running it on the server is backwards. Cancelled and restarted on the laptop.

**ssh-copy-id not available in Windows CMD**
`ssh-copy-id` is a Linux command — not available in Windows Command Prompt.
Fix: Use **Git Bash** on Windows — it includes all standard SSH utilities.

---

Verification Commands

| Command                     | What It Shows |
|---------                    |--------------|
| `sudo systemctl status ssh` | Confirms SSH service is running |
| `sudo ss -tlnp \| grep ssh` | Shows which port SSH is listening on |
| `cat ~/.ssh/authorized_keys`| Lists installed public keys on the server |

---

Result

Server accepts only key-based connections.
Password authentication disabled.
Root login disabled. ✅
