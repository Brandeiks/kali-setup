# 🗂️ Configuración de íconos en la terminal con `lsd` + Nerd Fonts

> **Entorno:** Kali Linux 2025.4 | **Shell:** zsh | **Herramienta:** lsd (LSDeluxe)

---

## ¿Qué es `lsd`?

`lsd` (LSDeluxe) es un reemplazo moderno del comando `ls` escrito en Rust. Su principal característica es mostrar **íconos visuales** junto a cada archivo y directorio, lo que facilita la identificación rápida del tipo de contenido en la terminal.

**Antes:**
```
Desktop  Documents  Downloads  Music  Pictures
```

**Después:**
```
 Desktop    Documents    Downloads    Music    Pictures
```

---

## Requisitos previos

- Kali Linux (o cualquier distro Debian-based)
- Terminal con soporte para fuentes personalizadas (XFCE Terminal, Kitty, Alacritty, etc.)
- Conexión a internet

---

## Paso 1 — Instalar `lsd`

```bash
sudo apt update && sudo apt install lsd -y
```

Verificar la instalación:

```bash
which lsd
# Salida esperada: /usr/bin/lsd

lsd --version
# Salida esperada: lsd 1.x.x
```

---

## Paso 2 — Instalar una Nerd Font

Los íconos de `lsd` requieren una **Nerd Font** instalada en el sistema. Sin ella, los íconos se mostrarán como caracteres extraños (`?` o `□`).

En este caso se usa **MesloLGS NF**, una de las más populares y compatibles con zsh + Powerlevel10k.

```bash
# Crear directorio de fuentes del usuario
mkdir -p ~/.local/share/fonts
cd ~/.local/share/fonts

# Descargar Meslo Nerd Font
wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.2.1/Meslo.zip

# Descomprimir
unzip Meslo.zip

# Actualizar caché de fuentes del sistema
fc-cache -fv
```

Verificar que las fuentes se instalaron:

```bash
ls ~/.local/share/fonts/ | grep -i meslo
```

---

## Paso 3 — Configurar la fuente en la terminal

Para que los íconos se rendericen correctamente, la terminal debe usar la Nerd Font instalada.

**En XFCE Terminal:**

1. Abrir la terminal
2. Ir a **Edit → Preferences**
3. Pestaña **Appearance**
4. En el campo **Font**, seleccionar `MesloLGS NF Regular` (o cualquier variante con "Nerd Font")
5. Aplicar y cerrar

> ⚠️ **Importante:** Si no cambias la fuente de la terminal, los íconos NO se mostrarán correctamente aunque `lsd` esté instalado.

---

## Paso 4 — Configurar aliases en `.zshrc`

Para que `ls` use automáticamente `lsd`, se agregan aliases al archivo de configuración de zsh.

```bash
nano ~/.zshrc
```

Agregar al final del archivo:

```bash
# lsd aliases — reemplazo moderno de ls con íconos
alias ls='lsd'
alias ll='lsd -l'
alias la='lsd -la'
alias lt='lsd --tree'
```

Guardar con `Ctrl+O` → Enter → `Ctrl+X` para salir de nano.

Aplicar los cambios:

```bash
source ~/.zshrc
```

---

## Verificación final

```bash
ls        # Listado básico con íconos
ll        # Listado largo con permisos e íconos
la        # Listado con archivos ocultos
lt        # Vista de árbol de directorios
```

Si todo está bien, la salida de `ls` debería mostrar íconos de carpeta 📁 junto a cada directorio y íconos específicos por tipo de archivo.

---

## Referencia de comandos `lsd`

| Comando | Descripción |
|---|---|
| `lsd` | Listado básico con íconos |
| `lsd -l` | Listado largo con permisos, tamaño y fecha |
| `lsd -la` | Listado largo incluyendo archivos ocultos |
| `lsd --tree` | Vista de árbol de directorios |
| `lsd --tree --depth 2` | Árbol con profundidad limitada |
| `lsd -S` | Ordenar por tamaño |
| `lsd -t` | Ordenar por fecha de modificación |
| `lsd --group-dirs first` | Mostrar carpetas primero |

---

## Troubleshooting

### Los íconos aparecen como `?` o `□`
→ La fuente de la terminal no es una Nerd Font. Revisa el **Paso 3**.

### `lsd: command not found` después de configurar el alias
→ `lsd` no está instalado. Ejecuta `sudo apt install lsd -y`.

### `zsh: bad pattern` al pegar comandos
→ Ocurre cuando se pegan backticks (`` ` ``) formateados desde una fuente externa. Escribe los comandos manualmente o usa un editor de texto plano para copiarlos.

### El alias `ls` no funciona tras reiniciar la terminal
→ Verifica que los aliases estén correctamente guardados en `~/.zshrc`:
```bash
grep "alias ls" ~/.zshrc
```

---

## Recursos

- [lsd en GitHub](https://github.com/lsd-rs/lsd)
- [Nerd Fonts](https://www.nerdfonts.com/)
- [Repositorio de Nerd Fonts en GitHub](https://github.com/ryanoasis/nerd-fonts)

---

*Write-up realizado en Kali Linux 2025.4 como parte de configuración de entorno para ciberseguridad.*
