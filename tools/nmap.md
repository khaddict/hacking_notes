# nmap

### What is nmap ?

Nmap, short for "Network Mapper," is a powerful open-source tool used for network exploration and security auditing. It is designed to discover hosts and services on a computer network, creating a map of the network's structure. Nmap operates by sending packets to target hosts and analyzing the responses to gather information about open ports, services, and other details.

### Install nmap

- `apt install nmap`

### Options

- `-A` : Enable OS detection, version detection, script scanning, and traceroute
- `-p` : Only scan specified ports
- `-sU` : UDP Scan
- `-sV` : Probe open ports to determine service/version info
- `-sC` : Equivalent to --script=default
- `-v` : Verbose
- `-Pn` : Treat all hosts as online -- skip host discovery

### Examples

- `nmap -A -p- test.lab`
- `nmap -p 21,22,80 -sC -sV test.lab`
- `nmap -A -v -Pn -p 20-80 test.lab`