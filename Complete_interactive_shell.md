# How to upgrade a rev shell to a fully interactive TTY shell

Get shell then:

```
python -c 'import pty; pty.spawn("/bin/bash")'  
```

Ctrl+Z (Background the current process)

```
stty raw -echo
fg
reset
control-c
export TERM=xterm
```

A fully interactive shell is there for you :)
