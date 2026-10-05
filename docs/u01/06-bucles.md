# 1.6 Bucles: `for` y `while`

Un bucle repite un bloque de código. En Python el `for` recorre **colecciones o rangos**, no cuenta con un contador como en Java.

## Versión 1: `for` donde se ve de dónde sale cada cosa

```python
for i in range(1, 6):
    print(f"Número: {i}")
```

| Parte | Qué hace |
|---|---|
| `i` | La variable que toma un valor distinto en cada vuelta |
| `in` | "dentro de" |
| `range(1, 6)` | **De dónde sale**: 1, 2, 3, 4, 5 (el final no se incluye) |
| bloque indentado | Lo que se repite |

## Versión 2: recorrer una colección

```python
numeros = [1, 2, 3, 4, 5]

for numero in numeros:
    if numero % 2 == 0:
        print(f"{numero} es par")
    else:
        print(f"{numero} es impar")
```

Python no tiene `forEach` como método de las listas: el propio `for` ya recorre los elementos.

## Variantes

```python
# Descendente
for i in range(5, 0, -1):
    print(i)

# Con paso de 2
for i in range(1, 11, 2):
    print(i)

# Sin incluir el último valor (range ya lo hace)
for i in range(1, 5):
    print(i)

# Recorrer una lista
nombres = ["Ana", "Luis", "Eva"]
for nombre in nombres:
    print(nombre)

# Recorrer con índice
for indice, nombre in enumerate(nombres):
    print(f"{indice}: {nombre}")
```

## Cortar un bucle con una variable de control

Una forma sencilla de parar un bucle sin `break`: una variable booleana (una *bandera*) que el bucle consulta en su condición. Cuando pasa lo que buscas, la pones en `False`.

```python
activo = True
i = 1

while activo:
    print(i)
    if i == 5:
        activo = False   # se cumple la condición: el bucle se detiene
    i += 1
```

Python no tiene `do-while`; se imita con una bandera que empieza en `True` (el cuerpo se ejecuta al menos una vez):

```python
activo = True
intentos = 0

while activo:
    intentos += 1
    print(f"Intento {intentos}")
    if intentos == 3:
        activo = False
```

??? note "Opcional: `break` y `continue`"
    Existen, pero no son imprescindibles: casi siempre se puede escribir lo mismo con una variable de control.

    ```python
    for i in range(1, 11):
        if i == 3:
            continue   # salta esta vuelta
        if i == 6:
            break      # sale del bucle
        print(i)       # 1, 2, 4, 5
    ```

## `while`

```python
cuenta = 3
while cuenta > 0:     # comprueba antes
    print(cuenta)
    cuenta -= 1
```
