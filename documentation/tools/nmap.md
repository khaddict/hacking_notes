# Nmap

## Introduction

Compact reference of common `nmap` commands and workflows used during recon and basic enumeration.

## Basic scan

List all open TCP ports:

```bash
nmap -p- <ip>
```

- `-p-` scans all 65535 TCP ports
- By default, Nmap scans TCP ports

To scan UDP ports:

```bash
nmap -sU <ip>
```

UDP is:
- much slower
- often filtered
- but important (DNS, SNMP, DHCP, NTP, etc.)

## Service and version detection

After finding open ports:

```bash
nmap -sC -sV <ip>
```

`-sC` runs the default set of NSE scripts (useful for banners, basic enum, SSL info, etc.)

`-sV` attempts to detect the service and version (and sometimes framework/OS information).

Example output:

```text
22/tcp open  ssh     OpenSSH 8.2p1
80/tcp open  http    Apache httpd 2.4.41
```

## OS detection

```bash
nmap -O <ip>
```

Attempts to guess the operating system (Linux, Windows, BSD, etc.).

## Scan timing

```bash
nmap -T4 <ip>
```

Timing templates:
- `-T0` very slow/stealthy
- `-T3` normal
- `-T4` faster (commonly used)
- `-T5` very aggressive

Commonly used: `-T4`.

## Disable host discovery (no ping)

Some hosts block ICMP. Use:

```bash
nmap -Pn <ip>
```

Forces Nmap to treat the host as "up". Useful when ping is filtered.

## Save output

Normal text:

```bash
nmap -oN scan.txt <ip>
```

XML:

```bash
nmap -oX scan.xml <ip>
```

All formats:

```bash
nmap -oA scan <ip>
```

Generates `scan.nmap`, `scan.xml`, and `scan.gnmap`. Useful for parsing.

## Scan specific ports

```bash
nmap -p 80,443,8080 <ip>
nmap -p 1-1000 <ip>  # range
```

## TCP + UDP combined

```bash
nmap -sS -sU <ip>
```

Combines TCP SYN and UDP scans.

## TCP scan types

SYN scan (most used):

```bash
nmap -sS <ip>
```

- fast
- stealthy (half-open)

TCP connect scan (no root required):

```bash
nmap -sT <ip>
```

- full TCP connection
- more detectable

## NSE scripts

List scripts:

```bash
ls /usr/share/nmap/scripts/
```

Run a specific script:

```bash
nmap --script http-title <ip>
```

Useful scripts: `http-title`, `smb-enum-shares`, `ftp-anon`, `vuln`, `ssl-cert`.

## Vulnerability scanning (NSE)

```bash
nmap --script vuln <ip>
```

May reveal CVEs or weak configurations. Always verify findings manually.

## Web enumeration examples

Get site title:

```bash
nmap --script http-title -p80,443 <ip>
```

SSL enumeration:

```bash
nmap --script ssl-cert,ssl-enum-ciphers -p443 <ip>
```

## SMB enumeration

```bash
nmap --script smb-enum-shares,smb-enum-users -p445 <ip>
```

Useful for Windows/AD targets.

## Network-wide scan

```bash
nmap 192.168.1.0/24
```

Host discovery only:

```bash
nmap -sn 192.168.1.0/24
```

## Typical workflow

1. Host discovery:

```bash
nmap -sn 192.168.1.0/24
```

2. Full port scan:

```bash
nmap -p- -T4 <ip>
```

3. Detailed scan on found ports:

```bash
nmap -sC -sV -O -p<ports> <ip>
```

Example:

```bash
nmap -sC -sV -O -p22,80,445 <ip>
```

## Important tips

- Scan all ports (`-p-`). Top-1000 scans can miss non-standard services.
- Don't forget UDP. Important services run on UDP (DNS, SNMP, NTP).
- Nmap can be noisy. Tune timing and scope to avoid detection in real engagements.

## Handy commands

```bash
nmap -p- -T4 <ip>                 # full TCP port scan
nmap -sC -sV -O <ip>              # detailed scan
nmap -sU <ip>                     # UDP scan
nmap -oA scan <ip>                # save all formats
nmap --script vuln <ip>           # NSE vuln scripts
```
