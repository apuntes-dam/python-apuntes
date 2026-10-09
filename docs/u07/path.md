# U7.5 · Ficheros con pathlib, bytes y bloques

<div class="ej-gate" data-unit="u07" data-nombre="U7 · Entrada/salida y GUI"></div>

Ejercicios de [7.E](t-path.md): `Path`, ficheros binarios y lectura y escritura por bloques. Todos controlan los errores con `try`/`except` y cierran los archivos con `with`.

## Ejercicio 7.16

**Gestor de archivos con `Path`.** Haz un menú: 1 crear carpeta, 2 crear archivo vacío, 3 ver información (ruta absoluta, nombre, carpeta padre, si es archivo o carpeta y tamaño), 4 copiar, 5 mover o renombrar, 6 borrar y 7 salir. Cada opción pide los nombres que necesite. Controla los errores con `try`/`except` y muestra un mensaje distinto para `FileExistsError` y `FileNotFoundError` (usa `e.filename`) y uno general para el resto de `OSError`. Recuerda que `shutil.copy` y `replace` sobrescriben: antes de copiar o mover, comprueba si el destino ya existe.

<details class="sol" data-key="kx/u7-5/7.16">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import shutil
from pathlib import Path
def pedir(texto):
    return input(texto).strip()
opcion = ""
# ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.17

**Fichero binario de caracteres.** Menú: 1 crear, 2 mostrar y 3 salir. Al crear, pide el nombre del fichero (si existe, pregunta si se crea de nuevo) y después datos hasta que se escriba `fin`; de cada dato se guarda en el fichero, como **un byte**, el código de su primer carácter (`ord`). Si el código es mayor que 255 (como el de `€`), avisa de que no cabe en un byte y no lo guardes. Al mostrar, lee el fichero byte a byte y escribe, por cada uno, su número y su carácter.

<details class="sol" data-key="kx/u7-5/7.17">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>from pathlib import Path
def pedir(texto):
    return input(texto).strip()
opcion = ""
while opcion != "3":
# ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.18

**Copiar un binario por bloques.** Pide el nombre de un fichero origen y uno destino y cópialo leyendo y escribiendo **bloques de 4096 bytes** (sin `shutil` ni `read_bytes`). Muestra cuántos bloques se han copiado y cuántos bytes en total, y comprueba que origen y destino pesan lo mismo.

<details class="sol" data-key="kx/u7-5/7.18">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>from pathlib import Path
origen = Path(input("Origen: ").strip())
destino = Path(input("Destino: ").strip())
try:
    bloques = 0
# ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.19

**Escribir un texto por bloques.** Pide un texto y un tamaño de bloque. Escríbelo en un fichero **por trozos de ese tamaño**, cortando el texto con `texto[inicio:fin]`. Muestra cuántas escrituras han hecho falta y el contenido final del fichero.

<details class="sol" data-key="kx/u7-5/7.19">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>from pathlib import Path
texto = input("Texto: ")
try:
    tamano = int(input("Tamaño del bloque: ").strip())
    if tamano &lt; 1:
# ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.20

**Contar leyendo por bloques.** Pide el nombre de un fichero de texto y léelo en **bloques de 16 caracteres** con `read(16)`. Muestra cuántos bloques se han leído, cuántas vocales y cuántas líneas tiene. Cuida la última línea, que puede no acabar en salto de línea.

<details class="sol" data-key="kx/u7-5/7.20">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>from pathlib import Path
ruta = Path(input("Fichero: ").strip())
try:
    bloques = 0
    vocales = 0
# ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.21

**Caracteres y bytes.** Pide un texto, guárdalo en un fichero con `encoding="utf-8"` y muestra cuántos **caracteres** tiene y cuántos **bytes** ocupa en el disco. Pruébalo con `hola` y con `Año 5 €` y explica por qué en el segundo caso los números no coinciden.

<details class="sol" data-key="kx/u7-5/7.21">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>from pathlib import Path
texto = input("Texto: ")
ruta = Path("texto.txt")
ruta.write_text(texto, encoding="utf-8")
print("Caracteres:", len(texto))
# ... (resto de la solución bloqueado)</code></pre></div>
</details>
