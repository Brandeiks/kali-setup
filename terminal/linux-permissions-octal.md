# 🔐 Permisos en Linux — Notación Octal

> Entendiendo `chmod` desde cero — por [Brandeiks](https://github.com/Brandeiks)

```bash
$ chmod 564 pepe
# ¿Qué acaba de pasar? Esto lo explica todo.
```

---

## 📋 Contenido

| Sección | Descripción |
|---|---|
| [¿Qué son los permisos?](#qué-son-los-permisos) | Conceptos base: usuario, grupo, otros |
| [La notación simbólica](#la-notación-simbólica-rwx) | Leer `rwxr-xr--` sin morir en el intento |
| [La notación octal](#la-notación-octal) | El truco del 4-2-1 |
| [Ejemplos prácticos](#ejemplos-prácticos) | Casos reales con `chmod` |
| [Referencia rápida](#referencia-rápida) | Tabla de valores más usados |

---

## ¿Qué son los permisos?

En Linux, cada archivo y directorio tiene **tres tipos de permisos** asignados a **tres identidades**:

```
drwxrwxr-- 2 kali Alumnos 4096 Mar 19 17:12 pepe
 ─┬──┬──┬─     ─┬─   ─┬─
  │  │  │       │     └── Otros (everyone else)
  │  │  │       └──────── Grupo
  │  │  └──────────────── Otros
  │  └─────────────────── Grupo
  └────────────────────── Dueño (owner)
```

Las tres identidades:

| Identidad | Descripción |
|---|---|
| **Owner** | El usuario dueño del archivo |
| **Group** | El grupo al que pertenece el archivo |
| **Others** | Cualquier otro usuario del sistema |

---

## La notación simbólica (rwx)

Cada identidad tiene hasta tres permisos posibles:

| Símbolo | Permiso | Sobre archivos | Sobre directorios |
|---|---|---|---|
| `r` | read | Leer contenido | Listar archivos (`ls`) |
| `w` | write | Modificar/eliminar | Crear/eliminar dentro |
| `x` | execute | Ejecutar | Entrar al directorio (`cd`) |
| `-` | sin permiso | — | — |

Ejemplo real de la práctica:

```bash
# Antes del chmod:
drwxrwxrwx  2 kali Alumnos  4096 Mar 19 17:12 pepe
# └──┘└──┘└──┘
# owner group others → todos tienen rwx

# Después de: chmod 564 pepe
dr-xrw-r--  2 kali Alumnos  4096 Mar 19 17:12 pepe
# └─┘└──┘└─┘
# owner=r-x  group=rw-  others=r--
```

---

## La notación octal

En lugar de escribir `rwxr-xr--`, Linux permite representar los permisos con **un solo dígito por identidad**, usando el sistema **base 8**.

### El truco del 4-2-1

Cada permiso tiene un valor fijo:

```
r = 4
w = 2
x = 1
- = 0
```

Para calcular el dígito de cada identidad, **suma los valores de los permisos activos**:

```
r w x        r w -        - - x
4+2+1 = 7    4+2+0 = 6    0+0+1 = 1
```

### Visualización con binario (la forma elegante)

Cada posición `rwx` es en realidad un **bit**:

```
r w x  →  binario  →  octal
─────────────────────────────
1 1 1  →    7      →  rwx  (todo)
1 1 0  →    6      →  rw-
1 0 1  →    5      →  r-x
1 0 0  →    4      →  r--
0 1 1  →    3      →  -wx
0 1 0  →    2      →  -w-
0 0 1  →    1      →  --x
0 0 0  →    0      →  ---  (nada)
```

### Desglose de `chmod 564`

```
  5        6        4
r-x       rw-      r--
4+0+1    4+2+0    4+0+0
owner    group    others
```

Verificación con `ls -l`:

```bash
$ chmod 564 pepe
$ ls -l
dr-xrw-r-- 2 kali Alumnos 4096 Mar 19 17:12 pepe
```

✅ Coincide exactamente.

---

## Ejemplos prácticos

```bash
# Script ejecutable solo por el dueño
chmod 700 script.sh      # rwx --- ---

# Archivo de configuración privado
chmod 600 config.env     # rw- --- ---

# Archivo público de solo lectura
chmod 644 index.html     # rw- r-- r--

# Directorio compartido con el grupo
chmod 775 proyecto/      # rwx rwx r-x

# Sin permisos para nadie (útil para pruebas)
chmod 000 secreto.txt    # --- --- ---
```

---

## Referencia rápida

| Octal | Simbólico | Uso común |
|---|---|---|
| `777` | `rwxrwxrwx` | ⚠️ Nunca en producción |
| `755` | `rwxr-xr-x` | Directorios públicos, binarios |
| `700` | `rwx------` | Scripts privados |
| `664` | `rw-rw-r--` | Archivos de proyecto en equipo |
| `644` | `rw-r--r--` | Archivos web, configs públicas |
| `600` | `rw-------` | Llaves SSH, `.env`, credenciales |
| `400` | `r--------` | Archivos de solo lectura críticos |

> 💡 **Regla de oro en seguridad**: asigna el **mínimo permiso necesario**. Principio de mínimo privilegio (least privilege).

---

## Notas del autor

Este write-up surge de una práctica con `chmod 564` en un directorio de prueba. Lo que parecía un número arbitrario tiene una lógica binaria elegante detrás.

Si vienes del mundo legal como yo, piensa en los permisos como **niveles de acceso en un contrato**: defines exactamente quién puede leer, modificar o ejecutar — y nadie más.

---

*Construyendo en la intersección entre el derecho y la seguridad digital.*
