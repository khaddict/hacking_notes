# Python pretty shell

First of all, check if python3 is installed :

- `python3 --version`

Then :

- `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- `CTRL + Z`
- `stty raw -echo; fg; ls`
- `export SHELL=/bin/bash`
- `export TERM=xterm`
- `stty rows 38 columns 116; reset`

# Bash pretty shell

- `script /dev/null -qc /bin/bash`
- `CTRL+Z`
- `stty raw -echo; fg; ls`
- `export SHELL=/bin/bash`
- `export TERM=xterm`
- `stty rows 38 columns 116; reset`