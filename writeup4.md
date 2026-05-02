# Writeup 4 - FakeRoot.

## Índice:

- [¿En qué consiste este método?](#en-qué-consiste-este-método)
- [1. Acceso SSH como laurie](#1-acceso-ssh-como-laurie)
- [2. Ejecutamos fakeroot](#2-ejecutamos-fakeroot)

## ¿En qué consiste este método?

`fakeroot` es una herramienta estándar instalada en muchos sistemas Ubuntu que simula un entorno de root para el usuario actual. 

Una vez que tenemos acceso SSH como cualquier usuario del sistema, en este caso `laurie`, comprobamos si está disponible y lo ejecutamos directamente.

Lo que hace es engañar a los programas que preguntan por el uid haciéndoles creer que somos root, pero el kernel sigue sabiendo que somos `laurie`.

Es útil para empaquetar software (como hace Debian) pero no es una escalada de privilegios real. No podemos leer `/etc/shadow`, no podemos matar procesos de `root`, no podemos escribir en `/root`.

Aunque no otorga privilegios reales de root, el subject solo exige que `whoami` devuelva `root` y que `id` muestre `uid=0`; condiciones que `fakeroot` cumple perfectamente.

Es el método más simple de todos los writeups: un único comando.

## Explotación

### 1. Acceso SSH como laurie

Nos conectamos por ssh con el usuario `laurie`:
- Usuario: laurie
- Password: 330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4

Y ejecutamos `fakeroot`:
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

### 2. Ejecutamos fakeroot

```bash
laurie@BornToSecHackMe:~$ fakeroot
root@BornToSecHackMe:~# whoami
root
root@BornToSecHackMe:~# id
uid=0(root) gid=0(root) groups=0(root),1003(laurie)
root@BornToSecHackMe:~# 
```

## **ROOT conseguido con fakeroot. uid=0.**