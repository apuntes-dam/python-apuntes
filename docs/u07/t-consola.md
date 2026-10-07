# 7.A Consola: entrada y salida estándar

Cuando un programa se ejecuta en una terminal, el sistema le da **tres canales** de texto, llamados **flujos estándar**:

| Flujo | Nombre | Para qué sirve | Por defecto |
|---|---|---|---|
| `stdin` | Entrada estándar | Lo que el programa **recibe** | El teclado |
| `stdout` | Salida estándar | Los **resultados** normales | La pantalla |
| `stderr` | Salida de error | Los **avisos y errores** | La pantalla |

Los dos de salida van a la pantalla, pero son **canales distintos**, y esa diferencia permite separar los resultados de los mensajes de error.

## Mostrar datos con formato

```python
import sys

equipos = [("Águilas", 45, 12), ("Lobos", 41, 5), ("Tigres", 38, -2)]

print(f"{'Equipo':<12}{'Pts':>5}{'Dif':>6}")
print("-" * 23)
total = 0
for nombre, puntos, diferencia in equipos:
    print(f"{nombre:<12}{puntos:>5}{diferencia:>+6}")
    total += puntos
print(f"Media de puntos: {total / len(equipos):.2f}")

print("Aviso: datos de ejemplo, no oficiales", file=sys.stderr)
print("Fin del informe")
```

Salida:

```text
Equipo        Pts   Dif
-----------------------
Águilas        45   +12
Lobos          41    +5
Tigres         38    -2
Media de puntos: 41.33
Fin del informe
```

Qué conviene aprender de este ejemplo:

* **Columnas alineadas.** Se reserva un **ancho fijo** para cada columna: el texto a la izquierda y los números a la derecha. Sin eso, las columnas quedan descuadradas.
* **Decimales fijos.** La media sale con dos decimales aunque el cálculo dé muchos más.
* **El aviso va por otro canal.** Verás que la línea «Aviso: datos de ejemplo, no oficiales» **no** aparece en el recuadro de arriba: se escribe en `stderr`, no en `stdout`. En una terminal sí se vería.

| Necesito... | En Python |
|---|---|
| Alinear a la izquierda | `f"{nombre:<12}"` |
| Alinear a la derecha | `f"{puntos:>5}"` |
| Decimales fijos | `f"{media:.2f}"` |
| Repetir un texto | `"-" * 23` |
| Mostrar sin salto de línea | `print("texto", end="")` |
| Con signo | `f"{d:+}"` |

Los *f-strings* siempre usan el punto como separador decimal, sea cual sea la configuración del equipo.

## Salida estándar y salida de error

Separarlas sirve para que un programa pueda **guardar los resultados en un archivo sin mezclarlos con los errores**. En una terminal, el operador `>` redirige la salida normal y `2>` la de error:

```bash
programa > resultados.txt 2> errores.txt
```

Con el ejemplo anterior, `resultados.txt` contendría la tabla y la media y `errores.txt` solo el aviso:

```bash
python cons1.py > resultados.txt 2> errores.txt
```

Para **dar entrada desde un archivo** en lugar del teclado se usa `<` (`programa < entrada.txt`), y para encadenar programas, la **tubería** `|`.

## Leer datos

Se lee con **`input()`**, que lanza **`EOFError`** cuando ya no queda entrada (por eso el ejemplo la captura). Para leer todo, `sys.stdin`. La salida de error se escribe con `print(..., file=sys.stderr)`.

```python
print("Escribe líneas (una vacía para terminar):")
lineas = 0
palabras = 0
while True:
    try:
        linea = input()
    except EOFError:
        break
    if linea == "":
        break
    lineas += 1
    palabras += len(linea.split())
print(f"Líneas: {lineas}")
print(f"Palabras: {palabras}")
```

Este programa lee líneas **hasta encontrar una vacía** (o hasta que se acabe la entrada). Si se le escribe `hola mundo`, `esto es una prueba` y una línea vacía, muestra:

```text
Escribe líneas (una vacía para terminar):
Líneas: 2
Palabras: 6
```

Una entrada del usuario es **siempre texto**: si necesitas un número hay que convertirlo y **prever que falle** (ver el patrón de «pedir hasta que sea válido» en [2.3 Excepciones](../u02/02-excepciones.md)).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Columnas descuadradas | Fijar un ancho para cada columna y alinear números a la derecha |
| Mezclar resultados y errores en la misma salida | Escribir los avisos en `stderr` |
| Dar por hecho que siempre hay una línea más que leer | Comprobar el fin de la entrada (`null`, `hasNextLine`, `EOFError`) |
| Usar el número leído sin convertirlo ni validarlo | Convertir dentro de un `try` y volver a pedir |
| Decimales con coma en unas máquinas y con punto en otras | Fijar la configuración regional al formatear (en Java y Kotlin) |

## Para practicar

Haz los ejercicios de [U7.1 · Consola](consola.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
