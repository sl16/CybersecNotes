spawn a python pseudo-terminal
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
or
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

to enable tab autocomplete
```
// press CTRL + Z (bring process to background)
stty raw -echo
fg
```
	

