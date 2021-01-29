# Level 0

## Vulnerability

Hardcoded password, discoverable with gdb

## Context

We find a binary with owner ```level01``` and SUID.
```
level00@OverRide:~$ ls -l
-rwsr-s---+ 1 level01 users 7280 Sep 10  2016 level00
```

When run it prompts user for password, then exits if given invalid password.
```
level00@OverRide:~$ ./level00
***********************************
* 	     -Level00 -		  *
***********************************
Password: Please!

Invalid Password!
level00@OverRide:~$
```

## Solution

Investigating with ```gdb``` we find no other functions apart from ```main()```. ```main()``` calls ```scanf()``` to read the password from stdin, then compares given password with value ```0x149c``` (5276 in decimal). If given password = 5276, a call to ```system()``` opens a shell. See [disassembly notes](https://github.com/anyashuka/Override/blob/main/level00/Ressources/disassembly_notes.md) for detailed gdb assembly breakdown.

Using password 5276:
```
level00@OverRide:~$ ./level00
***********************************
* 	     -Level00 -		  *
***********************************
Password:5276

Authenticated!
$ whoami
level01
$ cat /home/users/level01/.pass
uSq2ehEGT6c9S24zbshexZQBXUGrncxn5sD5QfGL
```

Exit the shell and log in to level01.
```
$ exit
level00@OverRide:~$ su level01
Password: uSq2ehEGT6c9S24zbshexZQBXUGrncxn5sD5QfGL
```

## Recreate Exploited Binary

As user ```level01```, in ```/tmp```, create and compile ```level00_source.c```
```
level01@OverRide:/tmp$ gcc level00_source.c -o level00_source
```

Edit permissions including suid, then move the binary to home directory.
```
level01@OverRide:/tmp$ chmod u+s level00_source; chmod +wx ~; mv level00_source ~
```

Exit back to user ```level00```, then launch the source.
```
level00@OverRide:~$ /home/users/level01/level00_source
***********************************
* 		 -Level00 -		  *
***********************************
Password:5276

Authenticated!
$ whoami
level01
```