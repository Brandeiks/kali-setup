# 🔐 Permisos Especiales en Linux — Sticky Bit

> Entendiendo el bit pegajoso y por qué `/tmp` no es el caos que parece — por [Brandeiks](https://github.com/Brandeiks)

```bash
$ chmod +t /tmp/compartido
# ¿Por qué nadie puede borrar los archivos de los demás? Esto lo explica todo.
```

---

## 📋 Contenido

| Sección | Descripción |
|---|---|
| [🍬 ¿Qué es el Sticky Bit?](#qué-es-el-sticky-bit) | Origen y propósito |
| [⚙️ ¿Cómo se activa?](#cómo-se-activa) | Notación simbólica y octal |
| [👁️ ¿Cómo se ve en `ls -l`?](#cómo-se-ve-en-ls--l) | Reconocerlo en la salida del terminal |
| [📁 El caso clásico: `/tmp`](#el-caso-clásico-tmp) | Por qué existe y qué protege |
| [🧪 Demostración práctica](#demostración-práctica) | Simulación con dos usuarios |
| [📜 Sticky Bit en archivos](#sticky-bit-en-archivos) | ¿Tiene efecto hoy en día? |
| [⚡ Referencia rápida](#referencia-rápida) | Tabla y casos de uso reales |

---

## 🍬 ¿Qué es el Sticky Bit?

El **Sticky Bit** es un permiso especial en Linux que, cuando se aplica a un **directorio**, restringe quién puede **eliminar o renombrar** los archivos dentro de él.

La regla es simple:

> 🛡️ En un directorio con Sticky Bit activo, solo el **dueño del archivo**, el **dueño del directorio** o **root** pueden eliminar o renombrar ese archivo — aunque otros usuarios tengan permiso de escritura (`w`) sobre el directorio.

Sin Sticky Bit, cualquier usuario con `w` sobre un directorio puede borrar *cualquier archivo* dentro de él, incluyendo los de otros. Eso es un problema en entornos multiusuario. ⚠️

---

## ⚙️ ¿Cómo se activa?

### 🔤 Notación simbólica

```bash
chmod +t nombre_directorio    # Añadir Sticky Bit
chmod -t nombre_directorio    # Quitar Sticky Bit
```

### 🔢 Notación octal

El Sticky Bit ocupa el **cuarto dígito** (el más a la izquierda), que normalmente se omite cuando es `0`:

```
  1   7   7   5
  │   │   │   └── others: r-x
  │   │   └────── group:  rwx
  │   └────────── owner:  rwx
  └────────────── especial: sticky bit activo
```

```bash
chmod 1775 directorio/   # rwxrwxr-x + sticky bit
chmod 1777 /tmp          # rwxrwxrwx + sticky bit  ← el clásico
```

> 💡 Los otros bits especiales son: `4` (SUID) y `2` (SGID). El Sticky es `1`.

---

## 👁️ ¿Cómo se ve en `ls -l`?

El Sticky Bit aparece como una **`t`** en la última posición de los permisos (donde normalmente estaría la `x` de *others*):

```bash
$ ls -ld /tmp
drwxrwxrwt 12 root root 4096 Mar 20 09:41 /tmp
#        ^
#        └── 't' minúscula = sticky bit activo + others tiene 'x'
```

### 🤔 ¿Y si others no tiene `x`?

```bash
drwxrwxrwT
#        ^
#        └── 'T' mayúscula = sticky bit activo, pero others NO tiene 'x'
```

| Carácter | Significado |
|---|---|
| `t` 🔡 (minúscula) | Sticky Bit activo + `x` para others |
| `T` 🔠 (mayúscula) | Sticky Bit activo, sin `x` para others |

---

## 📁 El caso clásico: `/tmp`

```bash
$ ls -ld /tmp
drwxrwxrwt 12 root root 4096 Mar 20 09:41 /tmp
```

`/tmp` tiene permisos `1777`: todos pueden leer, escribir y ejecutar — pero **nadie puede borrar archivos que no son suyos**. 🚫

Esto es crítico porque `/tmp` es un directorio **compartido por todos los procesos y usuarios** del sistema. Sin el Sticky Bit:

- 👩‍💻 El usuario `alice` crea `/tmp/sesion_alice.sock`
- 👨‍💻 El usuario `bob` (con permisos `w` en `/tmp`) podría borrarlo
- 💥 La sesión de `alice` se interrumpe — o peor, se explota

Con el Sticky Bit activo, `bob` obtiene un error: 🛑

```bash
$ rm /tmp/sesion_alice.sock
rm: cannot remove '/tmp/sesion_alice.sock': Operation not permitted
```

---

## 🧪 Demostración práctica

Simulación con dos usuarios: `kali` y `alumno`.

```bash
# 👑 Como root: crear directorio compartido con sticky bit
$ mkdir /compartido
$ chmod 1777 /compartido
$ ls -ld /compartido
drwxrwxrwt 2 root root 4096 Mar 20 10:00 /compartido
```

```bash
# 🐉 Como kali: crear un archivo dentro
$ touch /compartido/archivo_kali.txt
$ ls -l /compartido
-rw-r--r-- 1 kali kali 0 Mar 20 10:01 archivo_kali.txt
```

```bash
# 🎓 Como alumno: intentar borrar el archivo de kali
$ rm /compartido/archivo_kali.txt
rm: cannot remove '/compartido/archivo_kali.txt': Operation not permitted  🚫
```

```bash
# 🎓 Como alumno: puede crear sus propios archivos sin problema
$ touch /compartido/archivo_alumno.txt   # ✅ permitido
$ rm /compartido/archivo_alumno.txt      # ✅ permitido (es suyo)
```

🍬 El Sticky Bit protege la **propiedad individual** dentro de un espacio colectivo.

---

## 📜 Sticky Bit en archivos

Históricamente, el Sticky Bit aplicado a un **archivo ejecutable** le indicaba al kernel que mantuviera el programa en memoria (swap) después de ejecutarlo, para cargarlo más rápido la próxima vez 🏎️. De ahí el nombre *"sticky"* — el proceso se "pegaba" a la memoria.

**Hoy en día ese comportamiento está obsoleto.** 🪦 Los kernels modernos gestionan la memoria de forma más eficiente y el Sticky Bit en archivos es ignorado por la mayoría de sistemas Linux actuales.

> ⚠️ En la práctica: el Sticky Bit solo tiene efecto relevante sobre **directorios**.

---

## ⚡ Referencia rápida

| Comando | Resultado |
|---|---|
| `chmod +t dir/` | ✅ Activa el Sticky Bit (conserva permisos actuales) |
| `chmod -t dir/` | ❌ Desactiva el Sticky Bit |
| `chmod 1777 dir/` | 🔓 `rwxrwxrwx` + Sticky Bit |
| `chmod 1755 dir/` | 🔒 `rwxr-xr-x` + Sticky Bit |
| `ls -ld dir/` | 👁️ Verificar si tiene Sticky Bit (`t` o `T` al final) |
| `stat dir/` | 📊 Ver permisos completos incluyendo bits especiales |

### 🌍 Casos de uso reales

| Escenario | Por qué usar Sticky Bit |
|---|---|
| 🗂️ `/tmp` del sistema | Directorio multiusuario: cada proceso protege sus archivos temporales |
| ☁️ Carpeta de uploads compartida | Cada usuario sube sus archivos sin poder borrar los ajenos |
| 🏫 Directorio de trabajos en red | Entornos educativos o laborales con múltiples cuentas |
| 🚩 CTFs y laboratorios Linux | Común en máquinas con múltiples jugadores simultáneos |

> 💡 **Principio de mínimo privilegio aplicado**: el Sticky Bit no da más acceso — lo *restringe*. Es una capa de protección sobre un directorio que ya es permisivo.

---

## ✍️ Notas del autor

El Sticky Bit es uno de esos detalles que pasan desapercibidos hasta que alguien pregunta "¿por qué no puedo borrar este archivo si el directorio es `777`?" 🤯. La respuesta está en esa pequeña `t` al final de los permisos.

Si vienes del mundo legal como yo ⚖️, piensa en el Sticky Bit como una **cláusula de no disposición**: el espacio es compartido, todos pueden escribir — pero nadie puede eliminar lo que no le pertenece.

---

*🔐 Construyendo en la intersección entre el derecho y la seguridad digital.*
