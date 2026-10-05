# 1.2 Práctica con el lenguaje

## Estructura de un programa Python

```python
# 1. Importaciones
import math

# 2. Funciones
def cuadrado(n):
    return n * n

# 3. Programa principal
if __name__ == "__main__":
    print(cuadrado(5))     # 25
    print(math.sqrt(16))   # 4.0
```

Un programa es un archivo `.py` que se ejecuta de arriba abajo. **No hay llaves ni `;`: el bloque lo marca la indentación** (4 espacios por convención).

## Variables

```python
nombre = "Ana"      # str
edad = 20           # int
altura = 1.68       # float
activo = True       # bool
apodo = None        # ausencia de valor
```

No se declara el tipo: lo tiene el **valor**, y la misma variable puede cambiar de tipo (no es recomendable). Se puede consultar con `type(variable)`.

## Constantes

Python no tiene constantes reales. Por **convención**, un nombre en MAYÚSCULAS indica que no debe cambiar:

```python
IVA = 0.21
```

## Literales

`42` (int), `3.14` (float), `"hola"` o `'hola'` (str), `True` (bool), `[1, 2]` (list), `(1, 2)` (tuple), `{1, 2}` (set), `{"a": 1}` (dict), `None`.

## Operadores

```python
print(7 + 2)     # 9
print(7 / 2)     # 3.5  (siempre float)
print(7 // 2)    # 3    (división entera)
print(7 % 2)     # 1
print(2 ** 3)    # 8    (potencia)
print(3 > 2 and 2 > 1)   # True
x = 5
x += 2           # 7
print(x)
```

Grupos: aritméticos (`+ - * / // % **`), relacionales (`== != < > <= >=`), lógicos (`and or not`), asignación (`= += -=`), pertenencia (`in`) e identidad (`is`).

!!! tip "No existen `++` ni `--`"
    Se escribe `x += 1`.

## Entrada y salida

```python
nombre = input("¿Cómo te llamas? ")
print(f"Hola, {nombre}")              # f-string
print("Tienes", 20, "años")           # varios argumentos separados por espacio
```

## Comentarios

```python
# Comentario de una línea

"""
Cadena de varias líneas: se usa como
documentación (docstring) de funciones y módulos.
"""
```
