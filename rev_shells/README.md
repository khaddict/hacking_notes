# Reverse shells

https://www.revshells.com/

## Pretty shell

https://hacktricks.wiki/en/generic-hacking/reverse-shells/full-ttys.html  

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# CTRL+Z
stty raw -echo; fg; ls; export SHELL=/bin/bash
export TERM=xterm; stty rows 38 columns 116
reset
```
