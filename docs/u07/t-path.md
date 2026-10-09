# 7.E Ficheros con pathlib, bytes y bloques

En [7.B](t-archivos.md) y [7.C](t-texto.md) ya usaste `pathlib.Path` y `read_text`/`write_text`. Aquí se completa el cuadro con lo que suele pedirse en el módulo de Acceso a Datos:

* Las **excepciones concretas** de cada operación y cómo reaccionar a ellas.
* Los **ficheros binarios**: se guardan **bytes**, no letras.
* **Leer y escribir por bloques**: en vez de cargar todo el archivo de golpe, se trabaja con un trozo cada vez.

Todos los ejemplos de esta página se han **ejecutado** con Python en una carpeta vacía, en Windows (por eso las rutas llevan `\`).

## Path: operaciones y errores

`Path("datos") / "nota.txt"` une rutas con el operador `/`. Un `Path` es solo una ruta: que exista no significa que el archivo exista.

| Quiero… | Se escribe |
|---|---|
| Nombre, carpeta padre y extensión | `ruta.name`, `ruta.parent`, `ruta.suffix` |
| Ruta absoluta | `ruta.resolve()` |
| Saber si existe, si es archivo o carpeta | `ruta.exists()`, `ruta.is_file()`, `ruta.is_dir()` |
| Tamaño en bytes | `ruta.stat().st_size` |
| Crear una carpeta | `ruta.mkdir()`; con las que falten: `ruta.mkdir(parents=True)` |
| Crear un archivo vacío | `ruta.touch(exist_ok=False)` |
| Copiar | `shutil.copy(origen, destino)` |
| Mover o cambiar de nombre | `ruta.rename(destino)` (o `replace`) |
| Borrar un archivo | `ruta.unlink()`; si no existe y no importa: `ruta.unlink(missing_ok=True)` |
| Borrar una carpeta vacía | `ruta.rmdir()` (con todo lo que contiene: `shutil.rmtree(ruta)`) |

Un recorrido completo, con `try`, `except` y un `finally` que limpia lo que se haya creado:

```python
from pathlib import Path
import shutil

carpeta = Path("datos")
nota = carpeta / "nota.txt"
try:
    carpeta.mkdir(parents=True, exist_ok=True)
    nota.touch(exist_ok=False)
    nota.write_text("hola", encoding="utf-8")
    print("Ruta absoluta:", nota.resolve())
    print("Nombre:", nota.name, "| Padre:", nota.parent)
    print("Es archivo:", nota.is_file(), "| Es carpeta:", nota.is_dir())
    print("Tamaño:", nota.stat().st_size, "bytes")
    copia = Path(shutil.copy(nota, carpeta / "copia.txt"))
    movido = copia.rename(carpeta / "final.txt")
    print("Existe copia.txt:", copia.exists(), "| Existe final.txt:", movido.exists())
except OSError as e:
    print("Error:", type(e).__name__, "-", e)
finally:
    (carpeta / "final.txt").unlink(missing_ok=True)
    nota.unlink(missing_ok=True)
    if carpeta.exists():
        carpeta.rmdir()
    print("Limpieza hecha: existe la carpeta =", carpeta.exists())
```

**Salida:**

```text
Ruta absoluta: C:\Users\ana\proyecto\datos\nota.txt
Nombre: nota.txt | Padre: datos
Es archivo: True | Es carpeta: False
Tamaño: 4 bytes
Existe copia.txt: False | Existe final.txt: True
Limpieza hecha: existe la carpeta = False
```

El `finally` se ejecuta **siempre**, haya habido error o no. Por eso es el sitio para borrar lo que el programa creó.

### Qué pasa cuando algo falla

Cada fallo es una excepción distinta, y todas son subclases de **`OSError`**. Mira cuáles saltan y, sobre todo, **cuáles no**:

```python
from pathlib import Path
import shutil

def intentar(texto, accion):
    try:
        print(f"{texto} -> {accion()}")
    except OSError as e:
        print(f"{texto} -> {type(e).__name__}")

carpeta = Path("datos")
nota = carpeta / "nota.txt"
intentar("crear carpeta", lambda: carpeta.mkdir())
intentar("crear la carpeta otra vez", lambda: carpeta.mkdir())
intentar("lo mismo con exist_ok=True", lambda: carpeta.mkdir(exist_ok=True))
intentar("crear archivo", lambda: nota.touch(exist_ok=False))
intentar("crear el archivo otra vez", lambda: nota.touch(exist_ok=False))
intentar("lo mismo sin exist_ok=False", lambda: nota.touch())
intentar("tamaño de uno que no existe", lambda: (carpeta / "no.txt").stat().st_size)
intentar("copiar", lambda: shutil.copy(nota, carpeta / "copia.txt"))
intentar("copiar sobre uno que existe", lambda: shutil.copy(nota, carpeta / "copia.txt"))
intentar("copiar uno que no existe", lambda: shutil.copy(carpeta / "no.txt", carpeta / "x.txt"))
intentar("borrar uno que no existe", lambda: (carpeta / "no.txt").unlink())
intentar("unlink con missing_ok=True", lambda: (carpeta / "no.txt").unlink(missing_ok=True))
intentar("borrar una carpeta con cosas dentro", lambda: carpeta.rmdir())
intentar("mkdir sin la carpeta padre", lambda: (carpeta / "a" / "b").mkdir())
intentar("mkdir con parents=True", lambda: (carpeta / "a" / "b").mkdir(parents=True))
intentar("rename sobre uno que existe", lambda: nota.rename(carpeta / "copia.txt"))
intentar("replace sobre uno que existe", lambda: nota.replace(carpeta / "copia.txt"))
```

**Salida:**

```text
crear carpeta -> None
crear la carpeta otra vez -> FileExistsError
lo mismo con exist_ok=True -> None
crear archivo -> None
crear el archivo otra vez -> FileExistsError
lo mismo sin exist_ok=False -> None
tamaño de uno que no existe -> FileNotFoundError
copiar -> datos\copia.txt
copiar sobre uno que existe -> datos\copia.txt
copiar uno que no existe -> FileNotFoundError
borrar uno que no existe -> FileNotFoundError
unlink con missing_ok=True -> None
borrar una carpeta con cosas dentro -> OSError
mkdir sin la carpeta padre -> FileNotFoundError
mkdir con parents=True -> None
rename sobre uno que existe -> FileExistsError
replace sobre uno que existe -> datos\copia.txt
```

| Excepción | Cuándo se produce |
|---|---|
| `FileExistsError` | Crear algo donde ya hay un archivo o carpeta con ese nombre (`mkdir()`, `touch(exist_ok=False)`, `rename` en Windows) |
| `FileNotFoundError` | La ruta no existe: tamaño, copia, borrado... o `mkdir()` cuando falta la carpeta padre |
| `OSError` | Casos sin clase propia, como borrar una carpeta que todavía tiene cosas dentro |
| `PermissionError` | No tienes permiso sobre el archivo o la carpeta |

!!! warning "Lo que NO da error"
    `touch()` sin `exist_ok=False` no falla si el archivo ya existe, `shutil.copy` **sobrescribe** el destino sin avisar y `replace` también. Si no quieres perder datos, comprueba antes con `exists()`.

El texto del mensaje (`str(e)`) lo escribe el sistema operativo y **depende del idioma de Windows**, por eso aquí solo se muestra el nombre de la excepción. Para saber qué ruta falló, usa `e.filename`.

## Ficheros binarios

En un fichero binario cada elemento es un **byte**, un número entre 0 y 255. Se abre con el modo **`"wb"`** (escribir) o **`"rb"`** (leer), siempre dentro de `with`, que cierra el archivo al terminar aunque haya un error.

* `salida.write(bytes([numero]))` escribe **un** byte.
* `entrada.read(1)` lee un byte y lo devuelve como un objeto `bytes`; `dato[0]` es el número. Cuando no queda nada, devuelve **`b""`** (vacío), que cuenta como falso en un `while`.

```python
from pathlib import Path

fichero = Path("datos.bin")

with fichero.open("wb") as salida:
    for numero in [72, 111, 108, 97]:
        salida.write(bytes([numero]))
print("Tamaño:", fichero.stat().st_size, "bytes")

with fichero.open("rb") as entrada:
    dato = entrada.read(1)
    while dato:
        print("Leído", dato[0], "=", chr(dato[0]))
        dato = entrada.read(1)
```

**Salida:**

```text
Tamaño: 4 bytes
Leído 72 = H
Leído 111 = o
Leído 108 = l
Leído 97 = a
```

Para archivos pequeños hay atajos: `ruta.write_bytes(datos)` y `ruta.read_bytes()`.

!!! note "En Python los bytes son de 0 a 255"
    Un byte de Python nunca sale negativo. Y si intentas crear uno fuera de rango, Python avisa en vez de recortarlo en silencio:

```python
from pathlib import Path

fichero = Path("datos.bin")
fichero.write_bytes(bytes([200]))
print("read_bytes():", fichero.read_bytes())
print("Primer valor:", fichero.read_bytes()[0])

try:
    bytes([300])
except ValueError as e:
    print("ValueError:", e)
```

**Salida:**

```text
read_bytes(): b'\xc8'
Primer valor: 200
ValueError: bytes must be in range(0, 256)
```

!!! warning "Bytes y texto no se mezclan"
    En modo `"wb"` hay que escribir `bytes`, no `str`. Para pasar de texto a bytes se usa `"hola".encode("utf-8")`.

```python
from pathlib import Path

fichero = Path("datos.bin")
try:
    with fichero.open("wb") as salida:
        salida.write("hola")
except TypeError as e:
    print("TypeError:", e)
```

**Salida:**

```text
TypeError: a bytes-like object is required, not 'str'
```

### Leer y escribir por bloques

Leer byte a byte es lento con archivos grandes. Lo habitual es leer un **bloque** de golpe: `entrada.read(10)` devuelve **hasta** 10 bytes (menos en el último bloque) y `b""` cuando se acaba el archivo.

```python
from pathlib import Path

origen = Path("origen.bin")
origen.write_bytes(bytes(range(65, 90)))        # 25 bytes de prueba

copia = Path("copia.bin")
with origen.open("rb") as entrada, copia.open("wb") as salida:
    bloque = entrada.read(10)
    while bloque:
        print("Bloque de", len(bloque), "bytes")
        salida.write(bloque)
        bloque = entrada.read(10)
print("Origen:", origen.stat().st_size, "bytes | Copia:", copia.stat().st_size, "bytes")
```

**Salida:**

```text
Bloque de 10 bytes
Bloque de 10 bytes
Bloque de 5 bytes
Origen: 25 bytes | Copia: 25 bytes
```

El último bloque trae solo 5 bytes. Con `read(10)` no hay problema, porque el objeto devuelto ya tiene el tamaño justo. El error aparece si reutilizas un **buffer** con `readinto`, que rellena un `bytearray` y devuelve cuántos bytes ha puesto:

```python
from pathlib import Path

origen = Path("origen.bin")
origen.write_bytes(bytes(range(65, 90)))

mala = Path("mala.bin")
with origen.open("rb") as entrada, mala.open("wb") as salida:
    buffer = bytearray(10)
    leidos = entrada.readinto(buffer)
    while leidos:
        salida.write(buffer)            # ¡escribe los 10 bytes aunque se hayan leído menos!
        leidos = entrada.readinto(buffer)
print("Origen:", origen.stat().st_size, "bytes | Copia mal hecha:", mala.stat().st_size, "bytes")
```

**Salida:**

```text
Origen: 25 bytes | Copia mal hecha: 30 bytes
```

!!! warning "Escribe solo lo que has leído"
    `salida.write(buffer)` escribe **todo** el buffer. En el último bloque conserva los bytes del bloque anterior, así que la copia sale más grande y estropeada. Con `readinto`, escribe `buffer[:leidos]`.

## Ficheros de texto: caracteres, bytes y bloques

Un archivo de texto también son bytes. Para leerlo como letras hay que decir cómo se codifican, y por eso **caracteres y bytes no coinciden**: en UTF-8 las letras sin tilde ocupan 1 byte, pero la `ñ`, las vocales con tilde o el `€` ocupan 2 o 3.

```python
from pathlib import Path

texto = "Año 2026: 5 € ñandú\nsegunda línea\n"
fichero = Path("texto.txt")

with fichero.open("w", encoding="utf-8") as f:
    f.write(texto)
print("Caracteres:", len(texto))
print("Bytes en el disco:", fichero.stat().st_size)

with fichero.open("w", encoding="utf-8", newline="\n") as f:
    f.write(texto)
print("Bytes con newline=\"\\n\":", fichero.stat().st_size)
```

**Salida:**

```text
Caracteres: 34
Bytes en el disco: 42
Bytes con newline="\n": 40
```

Salen 34 caracteres, pero 40 bytes con `newline="\n"`, y **42 con la configuración por defecto**: en Windows, Python convierte cada `\n` en `\r\n` al escribir (y lo deshace al leer). Si necesitas el contenido exacto, usa `newline="\n"`.

!!! warning "Indica siempre `encoding=\"utf-8\"`"
    Si no lo indicas, Python usa la codificación del sistema, que en Windows no es UTF-8:

```python
from pathlib import Path
import locale

fichero = Path("texto.txt")
with fichero.open("w") as f:                        # sin encoding
    f.write("ñ")
print("Codificación por defecto:", locale.getpreferredencoding(False))
print("Bytes escritos:", fichero.read_bytes())

with fichero.open("w", encoding="utf-8") as f:      # con encoding
    f.write("ñ")
print("Con utf-8:", fichero.read_bytes())
```

**Salida:**

```text
Codificación por defecto: cp1252
Bytes escritos: b'\xf1'
Con utf-8: b'\xc3\xb1'
```

Con `read(n)` en modo texto se leen **n caracteres** (no bytes), así que nunca se parte una `ñ` por la mitad:

```python
from pathlib import Path

fichero = Path("texto.txt")
fichero.write_text("Año 2026: 5 € ñandú\nsegunda línea\n", encoding="utf-8")

with fichero.open(encoding="utf-8") as lector:
    n = 1
    bloque = lector.read(8)
    while bloque:
        print(f"Bloque {n} ({len(bloque)} caracteres): [{bloque!r}]")
        n += 1
        bloque = lector.read(8)
```

**Salida:**

```text
Bloque 1 (8 caracteres): ['Año 2026']
Bloque 2 (8 caracteres): [': 5 € ña']
Bloque 3 (8 caracteres): ['ndú\nsegu']
Bloque 4 (8 caracteres): ['nda líne']
Bloque 5 (2 caracteres): ['a\n']
```

Para escribir por bloques se cortan trozos del texto con `texto[inicio:fin]`:

```python
from pathlib import Path

letras = "ABCDEFGHIJKLMNOPQRSTU"                    # 21 caracteres
destino = Path("letras.txt")
escrituras = 0
with destino.open("w", encoding="utf-8") as escritor:
    for inicio in range(0, len(letras), 8):
        escritor.write(letras[inicio:inicio + 8])  # el último trozo es más corto
        escrituras += 1
print("Escrituras:", escrituras)
print("Contenido:", destino.read_text(encoding="utf-8"))
```

**Salida:**

```text
Escrituras: 3
Contenido: ABCDEFGHIJKLMNOPQRSTU
```

!!! note "Y si solo quieres las líneas"
    Para recorrer un archivo línea a línea no hace falta leer por bloques: `for linea in f:` o `ruta.read_text(encoding="utf-8").splitlines()` (ver [7.C](t-texto.md)). Los bloques se usan cuando el archivo es grande o cuando te piden leer un número fijo de caracteres.

## Errores frecuentes

* **Olvidar `with`**: el archivo se queda abierto y puede no guardarse todo lo escrito.
* **No indicar `encoding="utf-8"`**: la `ñ` y las tildes se guardan distinto según el equipo.
* **Escribir `str` en modo `"wb"`** (o esperar `str` al leer en `"rb"`): da `TypeError`.
* **Escribir el buffer entero** en vez de `buffer[:leidos]`.
* **Dar por hecho que `copy` o `replace` avisan** cuando el destino existe: sobrescriben.
* **Esperar el mismo tamaño en Windows y Linux**: el texto con `\n` ocupa más en Windows si no indicas `newline="\n"`.

## Para practicar

Los ejercicios [7.16 a 7.21](path.md) usan todo lo de esta página.
