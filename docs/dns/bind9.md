# DNS Server — BIND9 Setup on Ubuntu Server 24.04

## What Is This?

A DNS (Domain Name System) server translates hostnames into IP addresses and vice versa.
Instead of remembering `192.168.x.xx`, devices can resolve `lab-server-01.lab.local`.

This document covers how I installed and configured BIND9 as an authoritative DNS server
for a local lab domain (`lab.local`), including both forward and reverse lookup zones.

---

## Why Build a DNS Server?

- Core Network+ exam topic — record types, zones, resolution process
- Eliminates hardcoded IPs — services reference hostnames instead
- Teaches how DNS actually works, not just what it does
- Direct foundation for Active Directory (AD requires DNS)

---

## DNS Concepts Used in This Build

### Two Lookup Directions
| Direction | Record Type | Example |
|-----------|-------------|---------|
| Forward | A record | `lab-server-01.lab.local` → `192.168.x.xx` |
| Reverse | PTR record | `192.168.x.xx` → `lab-server-01.lab.local` |

### Reverse Zone Naming
Reverse zones use `.in-addr.arpa` format with the network address written **backwards**.

Network `192.168.0.20` → reverse zone: `0.168.192.in-addr.arpa`

The host octet (`.20`) goes in the PTR record, not the zone name — because the zone
covers the entire network, not one host.

### SOA Record (Start of Authority)
Every zone file starts with an SOA record. It identifies the authoritative server
and contains timing values that control zone behaviour:

| Field | Value | Meaning |
|-------|-------|---------|
| Serial | `2` | Version number — **must be incremented every time the zone file changes** |
| Refresh | `604800` | How often secondary DNS servers check for updates (7 days) |
| Retry | `86400` | How long to wait before retrying a failed refresh (1 day) |
| Expire | `2419200` | When secondary stops answering if it can't reach primary (28 days) |
| Negative TTL | `604800` | How long to cache a "not found" response (7 days) |

> **Why the serial matters:** Secondary DNS servers compare their serial number against
> the primary. If the primary serial is higher, they pull the update. If you change a
> zone file without incrementing the serial, secondaries never sync.

---

## Installation

```bash
sudo apt install bind9 bind9utils bind9-doc -y
sudo systemctl enable --now bind9
sudo systemctl status bind9
```

### Force IPv4 Only
```bash
sudo nano /etc/default/named
```

Change:
```
OPTIONS="-u bind"
```
To:
```
OPTIONS="-u bind -4"
```

```bash
sudo systemctl restart bind9
```

This eliminates "network unreachable" warnings caused by BIND9 attempting
to reach IPv6 root servers on an IPv4-only network.

---

## Configuration

### Step 1 — Declare zones in named.conf.local
```bash
sudo nano /etc/bind/named.conf.local
```

```
zone "lab.local" {
    type master;
    file "/etc/bind/zones/db.lab.local";
};

zone "0.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.0";
};
```

> **Common mistake:** The file path must start with `/`. Writing `etc/bind/zones/...`
> (without the leading slash) is a relative path — BIND9 won't find the file, I did this mistake once.

### Step 2 — Create the zones directory
```bash
sudo mkdir /etc/bind/zones
```

### Step 3 — Create the forward zone file
```bash
sudo nano /etc/bind/zones/db.lab.local
```

```
$TTL    604800
@       IN      SOA     lab-server-01.lab.local. admin.lab.local. (
                        2         ; Serial
                        604800    ; Refresh
                        86400     ; Retry
                        2419200   ; Expire
                        604800 )  ; Negative Cache TTL

@               IN      NS      lab-server-01.lab.local.
lab-server-01   IN      A       192.168.x.xx
```

### Step 4 — Create the reverse zone file
```bash
sudo nano /etc/bind/zones/db.192.168.0
```

```
$TTL    604800
@       IN      SOA     lab-server-01.lab.local. admin.lab.local. (
                        2         ; Serial
                        604800    ; Refresh
                        86400     ; Retry
                        2419200   ; Expire
                        604800 )  ; Negative Cache TTL

@       IN      NS      lab-server-01.lab.local.
20      IN      PTR     lab-server-01.lab.local.
```

### Step 5 — Set correct ownership
```bash
sudo chown -R bind:bind /etc/bind/zones
```

### Step 6 — Validate zone files
```bash
sudo named-checkzone lab.local /etc/bind/zones/db.lab.local
sudo named-checkzone 0.168.192.in-addr.arpa /etc/bind/zones/db.192.168.0
```

Both must return `OK` before restarting BIND9.

### Step 7 — Restart and verify
```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

---

## Verification

### Forward lookup test
```bash
dig @localhost lab-server-01.lab.local
```

Expected ANSWER SECTION:
```
lab-server-01.lab.local.    604800    IN    A    192.168.x.xx
```

### Reverse lookup test
```bash
dig @localhost -x 192.168.x.xx
```

Expected ANSWER SECTION:
```
xx.x.168.192.in-addr.arpa.    604800    IN    PTR    lab-server-01.lab.local.
```

### Understanding dig output
| Field | Meaning |
|-------|---------|
| `status: NOERROR` | Query succeeded |
| `ANSWER: 1` | One record returned |
| `Query time: 0 msec` | Answered from local cache |
| `SERVER: 127.0.0.1#53` | BIND9 answered on port 53 |

---

## Firewall Rule
```bash
sudo ufw allow 53
```

Port 53 must be open for other devices on the network to query this DNS server.

---

## Problems Encountered

**"network unreachable" errors on startup**
BIND9 attempting IPv6 root servers on IPv4-only network.
Fix: Added `-4` flag to `/etc/default/named`

**Zone file not found on restart**
File path in `named.conf.local` was missing the leading `/`.
`etc/bind/zones/...` → `/etc/bind/zones/...`

**Zone check passing but file missing**
Forward zone file was saved as `db.local` instead of `db.lab.local`.
Fix: `sudo mv /etc/bind/zones/db.local /etc/bind/zones/db.lab.local`

---

## Security Note

By default BIND9 may allow zone transfers (`AXFR`) to any host.
A zone transfer hands an attacker your entire DNS map — every hostname and IP
in the zone. In production, restrict zone transfers to authorised secondary servers only.

This is a known reconnaissance technique and is tested on Security+.

---

## Result

- BIND9 installed and running on Ubuntu Server 24.04 ✅
- Forward zone `lab.local` resolving A records ✅
- Reverse zone `0.168.192.in-addr.arpa` resolving PTR records ✅
- Both directions verified with `dig` ✅
- Port 53 open in UFW ✅
