# UFW Firewall Configuration — Ubuntu Server 24.04

## What Is This?

A firewall controls which network traffic is allowed in and out of a server.
Without one, every port is potentially accessible — any service running on the
server could be reached by anyone on the network.

UFW (Uncomplicated Firewall) is Ubuntu's built-in firewall tool. It wraps the
underlying `iptables` system in simple, readable commands.

---

## The Default Policy Approach

The correct way to configure a firewall:

1. **Block everything incoming by default**
2. **Allow only what you specifically need**

This is called the principle of least privilege — only open what is necessary,
close everything else.

---

## Steps

### 1. Set default policies
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 2. Allow SSH before enabling the firewall
```bash
sudo ufw allow 22/tcp
```

**Critical:** Always add the SSH rule before enabling UFW.
If you enable the firewall without allowing SSH, you will lock yourself out
and need physical access to the server to fix it.

### 3. Enable the firewall
```bash
sudo ufw enable
```

It will warn that enabling may disrupt SSH connections — type `y` and Enter.
Your SSH rule is already added so you will not be locked out.

### 4. Verify the configuration
```bash
sudo ufw status verbose
```

---

## Current Firewall Rules

| Port | Protocol | Action | Service |
|------|----------|--------|---------|
| 22 | TCP | ALLOW IN | SSH |
| All others | — | DENY IN | — |

---

## Common Rules for Future Services

As new services are added to the lab, these rules will be needed:

```bash
sudo ufw allow 80/tcp       # HTTP — web server
sudo ufw allow 443/tcp      # HTTPS — secure web server
sudo ufw allow 53           # DNS — name resolution
sudo ufw allow 67/udp       # DHCP — address assignment
```

Add rules only when the service is actually installed and running.

---

## Management Commands

| Command | What It Does |
|---------|-------------|
| `sudo ufw status verbose` | Shows all active rules and default policies |
| `sudo ufw allow 80/tcp` | Opens a port |
| `sudo ufw deny 80/tcp` | Closes a port |
| `sudo ufw delete allow 80/tcp` | Removes a rule entirely |
| `sudo ufw disable` | Turns the firewall off |
| `sudo ufw reset` | Clears all rules and starts fresh |

---

## Security Connection

Every open port is an attack surface. A service with a vulnerability running
on an open port is a direct path into the server.

UFW enforces the habit of thinking about what is exposed and what is not —
the same thinking required for Security+ and real production environments.

Port scanning with Nmap from another machine confirms what is visible:
```bash
nmap -sV 192.168.x.xx
```

A correctly configured server should show only port 22 open.
Everything else should appear closed or filtered.

---

## Result

Firewall active with default-deny policy.
Only SSH (port 22) open.
All other ports blocked. ✅
