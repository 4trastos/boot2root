# Boot2Root

## Descripción

Proyecto de pentesting de 42Network. El objetivo es obtener acceso `root` (uid=0) en un servidor vulnerable usando al menos **2 métodos diferentes**, documentando cada uno con un writeup completo paso a paso.

---

## Estructura del repositorio

```bash
Boot2Root/
├── writeup1        # Método 1 para obtener root
├── writeup2        # Método 2 para obtener root
├── scripts/        # Scripts opcionales usados durante la explotación
└── bonus/          # Métodos adicionales (bonus)
```

---

## Requisitos

- Máquina virtual de 64 bits con la ISO del proyecto `BornToSecHackMe`.
- La IP del servidor **no es visible** en el prompt — hay que descubrirla.
- Se debe explotar el **servidor**, no el archivo ISO directamente.
- Está **prohibido** explotar el arranque del servidor (GRUB u otros).
- Está **prohibido** el bruteforce de usuarios.

---

## Parte obligatoria

Obtener acceso `root` real (uid=0 con shell interactiva) mediante **2 métodos
diferentes**. Cada método debe estar documentado en su propio writeup con todos
los pasos descritos.

> **Nota:** Ser root en una base de datos u otro servicio equivalente **no cuenta**
> como solución válida a menos que sea un paso intermedio claramente documentado.

---

## Parte bonus

Cada método adicional y funcional para obtener root suma 1 o 2 puntos extra
sobre 5. El servidor tiene múltiples vectores de ataque — merece la pena
explorarlo en profundidad.

> El bonus solo se evalúa si la parte obligatoria está **perfecta**.

---

## Advertencias

- El repositorio **no debe contener ningún binario**.
- Los archivos del servidor no deben incluirse en el repo bajo ninguna
  circunstancia — deben descargarse durante la evaluación si es necesario.
- El uso de herramientas externas requiere un entorno configurado (VM, Docker,
  Vagrant).
- Todo el contenido del repositorio debe poder ser explicado y justificado
  durante la evaluación.

---

## Nota sobre la bin bomb

> Si la contraseña encontrada es `123456`, la contraseña a usar es `123546`.
