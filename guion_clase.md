# Guión de clase — Scripting en Bash

---

## 1. Introducción a los scripts de Bash

### ¿Por qué usamos Bash?

Bash es el lenguaje de scripting estándar en sistemas Linux/Unix. En el mundo profesional se utiliza para:

- **Automatización**: evitar tareas repetitivas manuales.
- **Copias de seguridad y versionado** de archivos y configuraciones.
- **Procesos periódicos**: tareas programadas con `cron` (backups, limpieza de logs...).
- **Inicialización de proyectos**: instalación de dependencias, configuración de entornos.
- **Migración de sistemas**: mover, transformar y validar datos entre servidores — caso muy común en empresas.

> Un script de Bash no es más que un fichero de texto con una secuencia de comandos que el sistema ejecuta uno tras otro.

### El shebang `#!/bin/bash`

La primera línea de **cualquier script** debe indicar qué intérprete se usará:

```bash
#!/bin/bash
```

Sin esta línea el sistema puede no saber cómo ejecutar el fichero.

### Permisos de ejecución

Un fichero de texto no se puede ejecutar directamente. Hay que darle permiso:

```bash
chmod +x nombre_script.sh   # Permiso a todos
chmod u+x nombre_script.sh  # Permiso solo al usuario propietario
```

Una vez hecho, se ejecuta con:

```bash
./nombre_script.sh
```

---

## 2. Introducción a Vim

Durante la clase trabajaremos **solo con la línea de comandos**, sin interfaz gráfica. Esto refleja la realidad de trabajar sobre servidores remotos.

### Comandos básicos de Vim

| Acción | Teclas |
|---|---|
| Abrir un fichero | `vim nombre.sh` |
| Entrar en modo edición | `i` |
| Salir del modo edición | `Esc` |
| Guardar y salir | `Esc` → `:wq` → `Enter` |
| Salir sin guardar | `Esc` → `:q!` → `Enter` |
| Crear un fichero vacío | `touch nombre.sh` |

> **Nota**: `gedit` y otros editores gráficos no estarán disponibles en un servidor sin entorno de escritorio. Por eso es esencial aprender Vim.

---

## 3. Variables y argumentos — `argumentos.sh`

### Variables

En Bash, las variables se declaran sin tipo y se usan con `$`:

```bash
NOMBRE="Javier"
echo "Hola, $NOMBRE"        # Con comillas dobles interpreta la variable
echo 'Hola, $NOMBRE'        # Con comillas simples lo toma como texto literal
```

> **Diferencia clave**: las comillas dobles `"..."` permiten expandir variables; las simples `'...'` no.

Para declarar **constantes** (variables de solo lectura):

```bash
readonly PI=3.14
```

### Argumentos del script

Al ejecutar un script podemos pasarle datos desde fuera:

```bash
./argumentos.sh hola mundo
```

Dentro del script, esos valores se reciben con variables especiales:

![Variables especiales de Bash](variables-especiales.png)

| Variable | Significado |
|---|---|
| `$0` | Nombre del script |
| `$1`, `$2`... | Primer, segundo... argumento |
| `$#` | Número total de argumentos recibidos |
| `$@` | Todos los argumentos como lista |
| `$?` | Código de retorno del último comando ejecutado |

### Código completo — `argumentos.sh`

```bash
#!/bin/bash

PRIMERO="$1"
SEGUNDO="$2"

echo "El primer argumento es '$PRIMERO' y el segundo es '$SEGUNDO'"

echo "Nombre del script: $0"
echo "Número total de argumentos: $#"
echo "Valor de todos los argumentos: $@"
```

**Ejecución de ejemplo:**

```bash
./argumentos.sh linux bash
# El primer argumento es 'linux' y el segundo es 'bash'
# Nombre del script: ./argumentos.sh
# Número total de argumentos: 2
# Valor de todos los argumentos: linux bash
```

---

## 4. Arrays — `array.sh`

Un array permite almacenar múltiples valores en una sola variable.

### Declarar un array

```bash
array=("primero" "segundo" "tercero" "cuarto")
```

### Operaciones con arrays

| Operación | Sintaxis |
|---|---|
| Acceder a un elemento | `${array[0]}` |
| Todos los elementos | `${array[*]}` |
| Número de elementos | `${#array[@]}` |
| Modificar un elemento | `array[0]="nuevo"` |
| Añadir un elemento | `array+=("quinto")` |
| Eliminar un elemento | `unset array[2]` |

> Los arrays en Bash pueden mezclar tipos: cadenas, números, etc.

### Código completo — `array.sh`

```bash
#!/bin/bash

array=("primero" "segundo" "tercero" "cuarto")

echo "Primer elemento: '${array[0]}'"
echo "Todo el array: '${array[*]}'"
echo "Número total de elementos: '${#array[@]}'"

array[0]="modificado"
echo "Array tras modificar: '${array[*]}'"

array+=("quinto")
echo "Ahora tiene '${#array[@]}' elementos: '${array[*]}'"

unset array[2]
echo "Array tras eliminar el elemento 2: '${array[*]}'"

array_diverso=("primero" 2 'tercero' 4 "quinto")
echo "Array diverso: ${array_diverso[*]}"
```

---

## 5. Operaciones aritméticas — `aritmetica.sh`

### Aritmética en Bash

Para realizar cálculos numéricos es **obligatorio** envolver la expresión en `$(( ))`:

```bash
resultado=$(( 5 + 3 ))
echo $resultado   # 8
```

Operadores disponibles:

| Operador | Significado |
|---|---|
| `+` | Suma |
| `-` | Resta |
| `*` | Multiplicación |
| `/` | División entera |
| `%` | Módulo (resto) |
| `**` | Potencia |

### El comando `cut`

`cut` permite extraer columnas de una cadena usando un delimitador:

```bash
echo "hola:mundo:bash" | cut -f2 -d ":"   # mundo
```

- `-d ":"` → delimitador
- `-f2` → campo número 2

En el script lo usamos para extraer el tamaño de un archivo de la salida de `ls -l`:

```bash
tamanio=$(ls -l archivo.txt | cut -f5 -d " ")
```

### Concatenación de cadenas

```bash
a="hola"
b="mundo"
c="$a$b"
echo $c   # holamundo
```

### Código completo — `aritmetica.sh`

```bash
#!/bin/bash

fs1=$(ls -l $1 | cut -f5 -d " ")
fs2=$(ls -l $2 | cut -f5 -d " ")

echo "El tamaño del archivo $1 es: $fs1 bytes"
echo "El tamaño del archivo $2 es: $fs2 bytes"

suma=$(($fs1 + $fs2))
resta=$(($fs1 - $fs2))
multiplicacion=$(($fs1 * $fs2))
division=$(($fs1 / $fs2))

echo "La suma de los tamaños es: $suma bytes"
echo "La resta de los tamaños es: $resta bytes"
echo "La multiplicación de los tamaños es: $multiplicacion bytes"
echo "La división de los tamaños es: $division bytes"

concatenados="$1$2"
echo "Los nombres de los archivos concatenados son: $concatenados"
```

**Ejecución de ejemplo:**

```bash
./aritmetica.sh fichero1.txt fichero2.txt
```

---

## 6. Condicionales — `condicionales.sh`

### Estructura `if / elif / else`

```bash
if [ condición ]; then
    # se ejecuta si la condición es verdadera
elif [ otra_condición ]; then
    # se ejecuta si la anterior era falsa y esta es verdadera
else
    # se ejecuta si ninguna condición fue verdadera
fi
```

### Operadores de comparación

![Operadores condicionales en Bash](condicionales.png)

**Comparación numérica:**

| Operador | Significado |
|---|---|
| `-eq` | Igual a |
| `-ne` | Distinto de |
| `-lt` | Menor que |
| `-le` | Menor o igual que |
| `-gt` | Mayor que |
| `-ge` | Mayor o igual que |

**Comparación de cadenas:**

| Operador | Significado |
|---|---|
| `=` | Igual |
| `!=` | Distinto |

**Comprobación de ficheros:**

| Operador | Significado |
|---|---|
| `-f` | Es un fichero regular |
| `-d` | Es un directorio |
| `-L` | Es un enlace simbólico |
| `-e` | Existe |

### Uso práctico: validar número de argumentos

```bash
if [ $# -ne 2 ]; then
    echo "Error: se necesitan exactamente 2 argumentos."
    exit 1
fi
```

> Este patrón es muy habitual al inicio de cualquier script.

### La sentencia `case`

Útil cuando hay múltiples valores posibles para una misma variable:

```bash
case "$variable" in
    "valor1")
        echo "Es valor1"
        ;;
    "valor2")
        echo "Es valor2"
        ;;
    *)
        echo "Otro valor"
        ;;
esac
```

### Código completo — `condicionales.sh`

```bash
#!/bin/bash

if [ $(whoami) = 'root' ]; then
    echo "Tú eres root"
else
    echo "Tú no eres root"
fi

AGE=$1

if [ $AGE -lt 13 ]; then
    echo "Eres un niño."
elif [ $AGE -lt 20 ]; then
    echo "Eres un adolescente."
elif [ $AGE -lt 65 ]; then
    echo "Eres un adulto."
else
    echo "Eres un adulto mayor."
fi

if [ $# -ne 2 ]; then
    echo "Error: Número inválido de argumentos"
    exit 1
fi

file=$2

if [ -f $file ]; then
    echo "$file es un archivo regular."
elif [ -L $file ]; then
    echo "$file es un soft link."
elif [ -d $file ]; then
    echo "$file es un directorio."
else
    echo "$file no existe"
fi
```

---

## 7. Bucles — `bucles.sh`

### Bucle `for` clásico

```bash
for ((i = 0; i < 3; i++)); do
    echo "Iteración $i"
done
```

### Bucle `for` sobre una lista o array

```bash
array=("si" "no" "tal vez")
for i in "${array[@]}"; do
    echo $i
done
```

También se puede iterar sobre ficheros de un directorio:

```bash
for i in ./*; do
    echo $i
done
```

### Bucle `while`

Se ejecuta **mientras** la condición sea verdadera:

```bash
num=1
while [ $num -le 3 ]; do
    echo "Número: $num"
    num=$(($num + 1))
done
```

### Bucle `until`

Se ejecuta **hasta que** la condición sea verdadera (contrario a `while`):

```bash
num=1
until [ $num -gt 3 ]; do
    echo "Número: $num"
    num=$(($num + 1))
done
```

> **En pizarra**: diferencia entre `while` y `until`. `while` continúa mientras la condición es cierta; `until` continúa mientras es falsa.

### `break` y `continue`

- `break` → sale del bucle inmediatamente.
- `continue` → salta a la siguiente iteración.

```bash
# break: sale al llegar a 3
for ((i=1; i<=10; i++)); do
    echo $i
    if [ $i -eq 3 ]; then
        break
    fi
done

# continue: solo imprime números impares
for ((i=0; i<=10; i++)); do
    if [ $(($i % 2)) -ne 1 ]; then
        continue
    fi
    echo $i
done
```

### Código completo — `bucles.sh`

```bash
#!/bin/bash

for ((i = 0 ; i < 2 ; i++)); do
    echo "Hola a todos"
done

echo "------------------"

for i in ./*; do
    formateado=$(echo $i | cut -f2 -d "/")
    echo $formateado
done

echo "------------------"

num=1
while [ $num -le 3 ]; do
    num=$(($num+1))
    echo "Hola a todas"
done

echo "------------------"

num2=1
until [ $num2 -gt 3 ]; do
    num2=$(($num2+1))
    echo "Hola a todas 2"
done

echo "------------------"

array=("si" "no" "tal vez")
for i in "${array[@]}"; do
    echo $i
done

echo "------------------"

for ((i=1;i<=10;i++)); do
    echo $i
    if [ $i -eq 3 ]; then
        break
    fi
done

echo "------------------"

for ((i=0;i<=10;i++)); do
    if [ $(($i % 2)) -ne 1 ]; then
        continue
    fi
    echo $i
done
```

---

## 8. Funciones — `funciones.sh`

### Definir y llamar a una función

```bash
mi_funcion () {
    echo "Hola desde la función"
}

mi_funcion   # llamada a la función
```

### Valor de retorno y `$?`

Las funciones en Bash devuelven un **código numérico** entre 0 y 255 con `return`:

- `0` → éxito (convención)
- Cualquier otro valor → error o estado específico

```bash
mi_funcion () {
    return 2
}

mi_funcion
echo "La función devolvió: $?"   # 2
```

> No se deben usar valores negativos en `return`; Bash los convierte y el comportamiento no es el esperado.

### Pasar argumentos a una función

Dentro de una función, `$1`, `$2`... hacen referencia a sus propios argumentos, **no** a los del script:

```bash
fun () {
    echo "$1 es el primer argumento de fun()"
    echo "$2 es el segundo argumento de fun()"
}

fun "hola" "mundo"
```

> Si necesitas usar los argumentos del script dentro de una función, guárdalos en variables antes de llamarla.

### Variables locales con `local`

Sin `local`, todas las variables de una función son **globales**:

```bash
var="global"

mi_funcion () {
    local var_local="solo aquí"
    echo "Dentro: $var, $var_local"
}

mi_funcion
echo "Fuera: $var"
echo "Fuera: $var_local"   # Vacío, var_local no existe aquí
```

### Código completo — `funciones.sh`

```bash
#!/bin/bash

hola () {
    echo "Hola Mundo"
    return 2
}

hola

echo "----------------------"

error () {
    return 0
}

error
echo "El estado de retorno de la función error es: $?"
hola
echo "El estado de retorno de la función hola es: $?"

echo "----------------------"

fun () {
    echo "$1 es el primer argumento de fun()"
    echo "$2 es el segundo argumento de fun()"
}

echo "$1 es el primer argumento del script."
echo "$2 es el segundo argumento del script."

fun Yes 7

echo "----------------------"

var="Variable global"

fen () {
    local local_var="Variable local"
    echo "Dentro de fen(), var es: $var"
    echo "Dentro de fen(), local_var es: $local_var"
}

fen

echo "Fuera de fen(), var es: $var"
echo "Fuera de fen(), local_var es: $local_var"
```

---

## Extra — Ejecutar un script desde cualquier directorio

Para ejecutar el script sin necesidad de estar en su carpeta, se añade su ruta al `PATH`:

```bash
export PATH="$PATH:/ruta/a/tu/carpeta/scripts"
```

Para que sea permanente, añade esa línea al final de `~/.bashrc` y recarga:

```bash
source ~/.bashrc
```

A partir de entonces, cualquier script en esa carpeta se podrá ejecutar solo con su nombre, sin `./`.

---

---

# Ejercicio final — Script de migración de archivos

> Este ejercicio pone en práctica **todo lo visto durante la clase**: argumentos, arrays, aritmética, condicionales, bucles y funciones.

## Enunciado

Crea un script llamado `migracion.sh` que:

1. Reciba **dos argumentos**: directorio origen y directorio destino.
2. Compruebe que se han pasado exactamente 2 argumentos.
3. Compruebe que ambos directorios existen.
4. Muestre el espacio disponible en el destino **antes** de migrar.
5. Mueva todos los archivos del origen al destino mostrando el nombre y tamaño de cada uno.
6. Lleve la cuenta del total de archivos movidos y bytes totales.
7. Muestre el espacio disponible en el destino **después** de migrar.
8. Al final, imprima por consola un informe con el resumen de la operación.

## Pautas de desarrollo

Sigue estos pasos en orden:

### Paso 1 — Shebang, argumentos y variables iniciales

```bash
#!/bin/bash

# Comprobar número de argumentos
if [ $# -ne 2 ]; then
    echo "Error: Se necesitan exactamente 2 argumentos."
    echo "Uso: $0 <directorio_origen> <directorio_destino>"
    exit 1
fi

ORIGEN="$1"
DESTINO="$2"
```

### Paso 2 — Comprobar que los directorios existen

Usa el operador `-d` visto en los condicionales:

```bash
if [ ! -d "$ORIGEN" ]; then
    echo "Error: El directorio origen '$ORIGEN' no existe."
    exit 1
fi

if [ ! -d "$DESTINO" ]; then
    echo "Error: El directorio destino '$DESTINO' no existe."
    exit 1
fi
```

### Paso 3 — Mostrar el espacio disponible

Para obtener el espacio libre en un directorio puedes usar el siguiente comando. No lo hemos visto en clase, pero puedes usarlo directamente:

```bash
df -h $DESTINO | tail -1 | awk '{print $4}'
```

- `df -h` → muestra el espacio en disco en formato legible.
- `tail -1` → coge solo la última línea (los datos, no la cabecera).
- `awk '{print $4}'` → extrae la cuarta columna, que es el espacio disponible.

Úsalo para mostrar el espacio antes y después de la migración:

```bash
echo "Espacio disponible antes: $(df -h $DESTINO | tail -1 | awk '{print $4}')"
```

### Paso 4 — Declarar el array y el contador de bytes

```bash
archivos_migrados=()
total_bytes=0
```

### Paso 5 — Bucle de migración

Recorre todos los archivos del origen con un `for`, mueve cada uno con `mv` y usa `$?` para saber si salió bien:

```bash
for archivo in "$ORIGEN"/*; do
    if [ -f "$archivo" ]; then
        nombre=$(basename "$archivo")
        tamanio=$(ls -l "$archivo" | cut -f5 -d " ")
        mv "$archivo" "$DESTINO/"
        if [ $? -eq 0 ]; then
            archivos_migrados+=("$nombre ($tamanio bytes)")
            total_bytes=$(($total_bytes + $tamanio))
            echo "[OK] $nombre -> $tamanio bytes"
        else
            echo "[ERROR] No se pudo mover $nombre"
        fi
    fi
done
```

> **Pistas**:
> - `basename` extrae solo el nombre del fichero de una ruta completa.
> - `cut -f5 -d " "` sobre `ls -l` extrae el tamaño — lo vimos en `aritmetica.sh`.
> - `$(( ))` para acumular el total de bytes — aritmética básica.

### Paso 6 — Mostrar el resumen final por consola

```bash
echo "-------------------------------------------"
echo "Archivos migrados : ${#archivos_migrados[@]}"
echo "Total bytes       : $total_bytes bytes"
echo "Espacio disponible después: $(df -h $DESTINO | tail -1 | awk '{print $4}')"
echo ""
echo "Detalle de archivos migrados:"
for f in "${archivos_migrados[@]}"; do
    echo "  - $f"
done
```

---

## Ejercicio extra — Envío del informe por email

Una vez el script base funcione correctamente, amplíalo con las siguientes funcionalidades:

### Extra 1 — Variable de fecha y email

Añade estas variables al inicio del script, justo después del shebang:

```bash
EMAIL="tuemail@dominio.com"
FECHA=$(date "+%Y-%m-%d %H:%M:%S")
```

- `date "+%Y-%m-%d %H:%M:%S"` genera una cadena con la fecha y hora actuales.

### Extra 2 — Construir el cuerpo del informe en una variable

```bash
INFORME="Informe de migración - $FECHA\n\nOrigen: $ORIGEN\nDestino: $DESTINO\n\nArchivos migrados:\n"
for f in "${archivos_migrados[@]}"; do
    INFORME+="  - $f\n"
done
INFORME+="\nTotal: ${#archivos_migrados[@]} archivos, $total_bytes bytes"
```

### Extra 3 — Enviar el email

```bash
echo -e "$INFORME" | mail -s "Migración $FECHA" "$EMAIL" 2>/dev/null
if [ $? -eq 0 ]; then
    echo "Email enviado a $EMAIL"
else
    echo "Advertencia: no se pudo enviar el email."
    echo "Instala mailutils con: sudo apt install mailutils"
fi
```

> `mail` es un comando del sistema para enviar correos desde la terminal. Requiere tener instalado el paquete `mailutils`.

---

## Referencias

- [It's FOSS en Español — Guías de Linux y Bash](https://itsfoss.com/es)
