# Nmap

## Scan de base

Lister tous les ports TCP ouverts :

```bash
nmap -p- <ip>
```

- `-p-` → scan les 65535 ports TCP
- Par défaut, Nmap scan en TCP

Pour scanner les ports UDP :

```bash
nmap -sU <ip>
```

UDP est :
- beaucoup plus lent
- souvent filtré
- mais très important (DNS, SNMP, DHCP, NTP, etc.)

---

# Détection de services et versions

Après avoir trouvé les ports ouverts :

```bash
nmap -sC -sV <ip>
```

### `-sC`
Lance les scripts NSE “par défaut”.

Permet notamment :
- récupérer bannières
- détecter mauvaises configs
- enum SMB/HTTP/FTP
- informations SSL
- etc.

### `-sV`
Détecte :
- service exact
- version
- parfois OS/framework

Exemple :
```text
22/tcp open  ssh     OpenSSH 8.2p1
80/tcp open  http    Apache httpd 2.4.41
```

---

# Détection du système d’exploitation

```bash
nmap -O <ip>
```

Essaye d’identifier :
- Linux
- Windows
- BSD
- version probable

---

# Rapidité du scan

## Scan rapide

```bash
nmap -T4 <ip>
```

Timing templates :
- `-T0` → très lent/furtif
- `-T3` → normal
- `-T4` → rapide (souvent utilisé)
- `-T5` → très agressif

Le plus courant :
```bash
-T4
```

---

# Désactiver le ping

Certaines machines bloquent l’ICMP.

```bash
nmap -Pn <ip>
```

Force Nmap à considérer l’hôte comme “up”.

Très utile quand :
- le ping est filtré
- la machine semble down mais répond sur certains ports

---

# Sauvegarder les résultats

## Format normal

```bash
nmap -oN scan.txt <ip>
```

## XML

```bash
nmap -oX scan.xml <ip>
```

## Tous les formats

```bash
nmap -oA scan <ip>
```

Génère :
- `scan.nmap`
- `scan.xml`
- `scan.gnmap`

Très pratique pour parser ensuite.

---

# Scan d’un port spécifique

```bash
nmap -p 80,443,8080 <ip>
```

Ou plage :

```bash
nmap -p 1-1000 <ip>
```

---

# Scan UDP + TCP

```bash
nmap -sS -sU <ip>
```

Combine :
- TCP SYN scan
- UDP scan

---

# Types de scans TCP

## SYN Scan (le plus utilisé)

```bash
nmap -sS <ip>
```

- rapide
- discret
- “half-open scan”

---

## TCP Connect Scan

```bash
nmap -sT <ip>
```

- connexion TCP complète
- plus détectable
- utilisé sans privilèges root

---

# Scripts NSE (Nmap Scripting Engine)

Lister les scripts disponibles :

```bash
ls /usr/share/nmap/scripts/
```

Lancer un script spécifique :

```bash
nmap --script http-title <ip>
```

Exemples utiles :
- `http-title`
- `smb-enum-shares`
- `ftp-anon`
- `vuln`
- `ssl-cert`

---

# Recherche de vulnérabilités

```bash
nmap --script vuln <ip>
```

Permet parfois de détecter :
- CVE connues
- configs faibles
- services vulnérables

À confirmer manuellement ensuite.

---

# Enumération web

## Récupérer le titre d’un site

```bash
nmap --script http-title -p80,443 <ip>
```

## Enum SSL

```bash
nmap --script ssl-cert,ssl-enum-ciphers -p443 <ip>
```

---

# Enum SMB

```bash
nmap --script smb-enum-shares,smb-enum-users -p445 <ip>
```

Très utile en environnement Windows/AD.

---

# Scan réseau entier

```bash
nmap 192.168.1.0/24
```

Découverte d’hôtes actifs.

---

# Découverte d’hôtes uniquement

```bash
nmap -sn 192.168.1.0/24
```

- pas de scan de ports
- uniquement découverte d’hôtes

---

# Bon workflow classique

## 1. Découverte d’hôtes

```bash
nmap -sn 192.168.1.0/24
```

---

## 2. Full port scan

```bash
nmap -p- -T4 <ip>
```

---

## 3. Scan détaillé des ports trouvés

```bash
nmap -sC -sV -O -p<ports> <ip>
```

Exemple :
```bash
nmap -sC -sV -O -p22,80,445 <ip>
```

---

# Conseils importants

## Toujours scanner tous les ports

Le top 1000 ports peut rater :
- services custom
- backdoors
- ports non standards

Donc :
```bash
-p-
```
est souvent préférable en pentest.

---

## UDP est souvent oublié

Des services critiques tournent en UDP :
- DNS (53)
- SNMP (161)
- NTP (123)

---

## Nmap peut être bruyant

Les IDS/EDR détectent facilement :
- scans agressifs
- NSE
- scans massifs

En environnement réel :
- attention au scope
- attention au timing

---

# Commandes très utiles à retenir

## Scan complet rapide

```bash
nmap -p- -T4 <ip>
```

## Scan détaillé

```bash
nmap -sC -sV -O <ip>
```

## UDP

```bash
nmap -sU <ip>
```

## Tous les formats de sortie

```bash
nmap -oA scan <ip>
```

## Vuln scan NSE

```bash
nmap --script vuln <ip>
```
