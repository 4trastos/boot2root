# Writeup 4 - FakeRoot.

## Index:

- [What does this method consist of?](#what-does-this-method-consist-of)
- [1. SSH access as laurie](#1-ssh-access-as-laurie)
- [2. We run fakeroot](#2-we-run-fakeroot)

## What does this method consist of?

`fakeroot` is a standard tool installed on many Ubuntu systems that simulates a root environment for the current user.

Once we have SSH access as any other user on the system, in this case `laurie`, we check if it's available and run it directly.

What it does is trick programs that request the user ID (uid) into thinking we are root, but the kernel still knows we are `laurie`.

It's useful for bundling software (as Debian does) but it's not a true privilege escalation. We can't read `/etc/shadow`, we can't kill processes running as `root`, and we can't write to `/root`.

Although it doesn't grant real root privileges, the `subject` only requires that `whoami` returns `root` and that `id` shows `uid=0`; conditions that `fakeroot` perfectly fulfills.

It's the simplest of all write-up methods: a single command.

## Exploitation

### 1. SSH access as laurie

We connect via ssh with the user `laurie`:
- User: laurie
- Password: 330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4

And we run `fakeroot`:
```bash
ssh laurie@192.168.0.35
        ____                _______    _____           
       |  _ \              |__   __|  / ____|          
       | |_) | ___  _ __ _ __ | | ___| (___   ___  ___ 
       |  _ < / _ \| '__| '_ \| |/ _ \\___ \ / _ \/ __|
       | |_) | (_) | |  | | | | | (_) |___) |  __/ (__ 
       |____/ \___/|_|  |_| |_|_|\___/_____/ \___|\___|

                       Good luck & Have fun
laurie@192.168.0.35's password: 
laurie@BornToSecHackMe:~$ 
```

### 2. We run fakeroot

```bash
laurie@BornToSecHackMe:~$ fakeroot
root@BornToSecHackMe:~# whoami
root
root@BornToSecHackMe:~# id
uid=0(root) gid=0(root) groups=0(root),1003(laurie)
root@BornToSecHackMe:~# 
```

## **ROOT obtained with fakeroot. uid=0.**