# Apuntes: Shell scripting en Linux

Ejercicios de scripting en Bash: comprobar puertos en escucha y procesar ficheros de texto.

---

## 1. Conceptos básicos de un script Bash

### Estructura mínima

```bash
#!/bin/bash
echo "Hola mundo"
```

- `#!/bin/bash` (*shebang*): indica qué intérprete ejecuta el script.
- Para ejecutarlo hay que darle permisos: `chmod +x script.sh` y lanzarlo con `./script.sh`.

### Parámetros del script

| Variable | Significado |
|---|---|
| `$0` | Nombre del script |
| `$1`, `$2`... | Primer parámetro, segundo... |
| `$#` | Número de parámetros recibidos |
| `$@` | Todos los parámetros |
| `$?` | Código de salida del último comando (0 = correcto) |

### Códigos de salida

- `exit 0` → todo ha ido bien.
- `exit 1` (u otro distinto de 0) → ha habido un error.

Es importante terminar con `exit 1` en los errores para que otros scripts o herramientas sepan que algo falló.

### Condicionales

```bash
if [ "$a" -lt 10 ]; then
    echo "menor"
elif [ "$a" -eq 10 ]; then
    echo "igual"
else
    echo "mayor"
fi
```

| Operador | Significado |
|---|---|
| `-eq` / `-ne` | igual / distinto (números) |
| `-lt` / `-le` | menor / menor o igual |
| `-gt` / `-ge` | mayor / mayor o igual |
| `-f fichero` | el fichero existe y es normal |
| `-d carpeta` | la carpeta existe |

> Las comillas en `"$var"` evitan errores cuando la variable está vacía o tiene espacios.

---

## 2. Ejercicio 1: comprobar si un puerto TCP está en escucha

### Enunciado

Script que reciba un número de puerto TCP como parámetro (entre 1 y 1024) y muestre si está en estado `LISTENING`. Si el parámetro no es válido, mostrará un error y terminará.

### Código

```bash
#!/bin/bash

# 1. Comprobar que se ha pasado exactamente un parámetro
if [ $# -ne 1 ]; then
    echo "Error: número de parámetros incorrecto."
    echo "Uso: $0 <puerto entre 1 y 1024>"
    exit 1
fi

puerto=$1

# 2. Comprobar que es un número entero entre 1 y 1024
if ! [[ "$puerto" =~ ^[0-9]+$ ]]; then
    echo "Error: '$puerto' no es un número válido."
    exit 1
fi

puerto=$((10#$puerto))   # normaliza (por ejemplo 080 -> 80)

if [ "$puerto" -lt 1 ] || [ "$puerto" -gt 1024 ]; then
    echo "Error: el puerto debe estar entre 1 y 1024."
    exit 1
fi

# 3. Comprobar si hay algún socket TCP en LISTEN con ese puerto
if ss -ltnH | awk '{print $4}' | grep -Eq "[:.]${puerto}$"; then
    echo "El puerto TCP $puerto está en estado LISTENING."
else
    echo "El puerto TCP $puerto NO está en estado LISTENING."
fi
```

### Explicación paso a paso

**Validación en tres niveles** (de más simple a más específico):

1. **Número de parámetros:** `$#` debe ser 1.
2. **Que sea un número:** la expresión regular `^[0-9]+$` solo acepta dígitos. Se usa `=~` dentro de `[[ ]]`.
3. **Que esté en rango:** entre 1 y 1024 (los llamados *puertos bien conocidos*).

**`$((10#$puerto))`:** en Bash, un número que empieza por 0 se interpreta como octal (`08` daría error). `10#` fuerza base decimal.

**El comando `ss`:**

```bash
ss -ltnH
```

| Opción | Significado |
|---|---|
| `-l` | solo sockets en escucha (*listening*) |
| `-t` | solo TCP |
| `-n` | mostrar números, sin resolver nombres |
| `-H` | sin línea de cabecera |

Ejemplo de salida:

```
LISTEN 0 128 0.0.0.0:22   0.0.0.0:*
LISTEN 0 511 [::]:80      [::]:*
```

**La tubería (pipe) `|`:**

- `awk '{print $4}'` extrae la columna 4 (dirección local, `0.0.0.0:22`).
- `grep -Eq "[:.]22$"` busca que la dirección **termine** en `:22`.
  - `-E` → expresiones regulares extendidas.
  - `-q` → modo silencioso: no imprime, solo devuelve éxito o fallo.
  - El `$` final evita que el puerto 8 coincida con el 80 o el 8080.

El `if` evalúa directamente el código de salida de `grep`: 0 si encontró algo, distinto de 0 si no.

### Pruebas

```bash
./comprobar_puerto.sh 22       # suele estar en escucha si hay SSH
./comprobar_puerto.sh 5000     # Error: fuera de rango
./comprobar_puerto.sh abc      # Error: no es número
./comprobar_puerto.sh          # Error: faltan parámetros
```

### Alternativas y notas

- `netstat -ltn` hace lo mismo, pero está obsoleto (paquete `net-tools`).
- `ss -ltnp` (con `-p`) muestra también el proceso que usa el puerto; requiere `sudo` para ver procesos de otros usuarios.
- Relevancia en ciberseguridad: comprobar qué puertos están abiertos es parte del **bastionado** (cuanto menos puertos abiertos, menor superficie de ataque) y de la **fase de reconocimiento** en hacking ético.

---

## 3. Ejercicio 2: sumar tiempos por usuario y ordenar

### Enunciado

Dado un fichero `datos.txt` con usuario y tiempo de uso (`HH:MM:SS`), mostrar un listado ordenado de forma ascendente por tiempo total. Si un usuario se repite, debe aparecer una sola línea con los tiempos sumados.

Fichero de ejemplo:

```
Pepe 02:30:44
Marcos 23:56:33
Pepe 10:33:01
Marta 05:47:44
Pepe 12:22:33
José 11:55:00
```

### Código

```bash
#!/bin/bash

fichero=${1:-datos.txt}

if [ ! -f "$fichero" ]; then
    echo "Error: no existe el fichero '$fichero'."
    exit 1
fi

awk '
{
    split($2, t, ":")
    segundos[$1] += t[1]*3600 + t[2]*60 + t[3]
}
END {
    for (usuario in segundos) {
        s = segundos[usuario]
        printf "%d %s %02d:%02d:%02d\n", s, usuario, s/3600, (s%3600)/60, s%60
    }
}' "$fichero" | sort -n | cut -d' ' -f2-
```

### Explicación paso a paso

**`fichero=${1:-datos.txt}`:** usa el primer parámetro; si no se pasa, toma `datos.txt` por defecto.

**`awk`:** procesa el fichero línea a línea, dividiéndola en campos (`$1` = usuario, `$2` = tiempo).

1. `split($2, t, ":")` divide `02:30:44` en `t[1]=02`, `t[2]=30`, `t[3]=44`.
2. Convierte a segundos: `horas*3600 + minutos*60 + segundos`.
3. `segundos[$1] += ...` usa un **array asociativo** indexado por el nombre. Si el usuario ya existe, **acumula**; si no, lo crea. Así se resuelven los repetidos.
4. El bloque `END { ... }` se ejecuta al terminar de leer el fichero. Recorre el array e imprime:
   - segundos totales (para poder ordenar),
   - nombre,
   - tiempo formateado `HH:MM:SS`.

**¿Por qué convertir a segundos?** Porque ordenar directamente `HH:MM:SS` como texto falla cuando las horas pasan de 99 y no permite sumar. Con segundos, el orden numérico es trivial.

**`sort -n`:** ordena numéricamente por la primera columna (los segundos). Para descendente: `sort -nr`.

**`cut -d' ' -f2-`:** elimina la primera columna (los segundos), que solo se usaba para ordenar. `-d' '` indica el separador y `-f2-` significa "desde el campo 2 hasta el final".

### Resultado esperado

```
Marta 05:47:44
José 11:55:00
Marcos 23:56:33
Pepe 25:26:18
```

Pepe suma `02:30:44 + 10:33:01 + 12:22:33 = 25:26:18`. Las horas superan 24 porque es tiempo **acumulado**, no una hora del día.

### Ideas de mejora

- Validar que cada línea tenga el formato correcto.
- Aceptar un parámetro `-d` para elegir orden descendente.
- Guardar la salida en un fichero: `./tiempos.sh datos.txt > resultado.txt`.

---

## 4. Resumen de comandos usados

| Comando | Para qué sirve |
|---|---|
| `ss` | Mostrar sockets y puertos |
| `awk` | Procesar texto por columnas, hacer cálculos |
| `grep` | Buscar patrones en texto |
| `sort` | Ordenar líneas |
| `cut` | Extraer columnas o campos |
| `chmod +x` | Dar permiso de ejecución |
| `\|` (pipe) | Pasar la salida de un comando a la entrada del siguiente |
| `>` | Redirigir la salida a un fichero |

---

## 5. Cómo ejecutarlo desde Windows

Estos scripts son para Linux. Opciones:

- **WSL** (Windows Subsystem for Linux): `wsl --install` en PowerShell como administrador. Es la opción más cómoda.
- **Máquina virtual** (VirtualBox/VMware) con Ubuntu o Kali.
- **Git Bash**: sirve para el ejercicio 2, pero `ss` no existe, así que el ejercicio 1 no funcionará.