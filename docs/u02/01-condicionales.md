# 2.1 Condicionales: decidir qué hacer

Un programa no ejecuta siempre las mismas líneas: **decide** según los datos. Para decidir usa una **condición**, una expresión cuyo resultado es verdadero o falso (un valor **booleano**), y ejecuta un bloque u otro según el resultado.

## `if`, `else if` y `else`

La estructura completa tiene tres partes: lo que se hace **si** se cumple la condición, lo que se hace si **no** pero se cumple otra, y lo que se hace en **cualquier otro caso**.

```python
def calificacion(nota):
    if nota < 0 or nota > 10:
        return "Nota no válida"
    elif nota < 5:
        return "Insuficiente"
    elif nota < 6:
        return "Suficiente"
    elif nota < 7:
        return "Bien"
    elif nota < 9:
        return "Notable"
    elif nota < 10:
        return "Sobresaliente"
    else:
        return "Matrícula"


if __name__ == "__main__":
    for nota in [3, 5, 6, 8, 9, 10, 11]:
        print(f"{nota} -> {calificacion(nota)}")
```

Salida:

```text
3 -> Insuficiente
5 -> Suficiente
6 -> Bien
8 -> Notable
9 -> Sobresaliente
10 -> Matrícula
11 -> Nota no válida
```

Las condiciones se evalúan **de arriba abajo** y se ejecuta **solo la primera rama que se cumple**; el resto se salta. Por eso el orden importa.

!!! warning "El orden de las ramas"
    Si en la función anterior pusieras primero `nota < 10`, la nota 3 entraría ahí y obtendrías «Sobresaliente». Ordena las condiciones de la más restrictiva a la más general, y deja el `else` para lo que no encaja en ninguna.

!!! tip "Casos raros primero"
    Comprobar al principio lo que no es válido (aquí, una nota fuera de 0 a 10) y devolver enseguida evita anidar condiciones y deja el resto del código más claro.

## Operadores de comparación y lógicos

| Comparación | Significa |
|---|---|
| `==` · `!=` | igual · distinto |
| `<` · `<=` | menor · menor o igual |
| `>` · `>=` | mayor · mayor o igual |

| Lógico | Símbolo en Python | Ejemplo |
|---|---|---|
| Y | `and` | `edad >= 18 and carnet` |
| O | `or` | `dia == 6 or dia == 7` |
| NO | `not` | `not carnet` |

Las condiciones lógicas se evalúan de izquierda a derecha y **se detienen en cuanto el resultado está claro** (*cortocircuito*): en `a && b`, si `a` es falso ya no se mira `b`. Se aprovecha para escribir `x != 0 && 10 / x > 2` sin riesgo de dividir entre cero.

En Python `==` compara el **contenido** y `is` comprueba si son *el mismo objeto*. Para números y textos usa siempre `==`; `is` se reserva para `None` (`if x is None`).

!!! warning "`=` no es `==`"
    Un solo `=` **asigna** un valor; dos `==` **comparan**. Escribir `if x = 3:` es un `SyntaxError`, así que Python te avisa antes de ejecutar.

En Python cualquier valor se puede usar como condición: son **falsos** `False`, `None`, `0`, `0.0`, `""` (texto vacío) y las colecciones vacías (`[]`, `{}`, `()`); todo lo demás es verdadero. Por eso `if lista:` significa «si la lista no está vacía». Es cómodo, pero conviene ser explícito cuando haya dudas.

## Una condición en una sola línea

Cuando solo hay que elegir entre dos **valores**, se puede escribir la condición dentro de la expresión.

```python
def mostrar(edad, carnet):
    tipo = "mayor" if edad >= 18 else "menor"
    permiso = "con carnet" if carnet else "sin carnet"
    conducir = "puede conducir" if edad >= 18 and carnet else "no puede conducir"
    print(f"{edad} años ({tipo}), {permiso} -> {conducir}")


if __name__ == "__main__":
    mostrar(17, True)
    mostrar(18, False)
    mostrar(18, True)
```

Salida:

```text
17 años (menor), con carnet -> no puede conducir
18 años (mayor), sin carnet -> no puede conducir
18 años (mayor), con carnet -> puede conducir
```

Úsalo para valores sencillos; si la condición o las ramas se complican, es mejor un `if` normal.

## Selección múltiple

Cuando una misma variable se compara con muchos valores concretos, el `if`/`else if` se hace largo. Para eso existe la **selección múltiple**.

```python
def tipo_de_dia(dia):
    if dia in (1, 2, 3, 4, 5):
        return "laborable"
    elif dia in (6, 7):
        return "fin de semana"
    else:
        return "día no válido"


if __name__ == "__main__":
    for dia in [1, 3, 6, 7, 9]:
        print(f"{dia} -> {tipo_de_dia(dia)}")
```

Para unos pocos casos basta con `if`/`elif` y el operador `in` (`dia in (1, 2, 3, 4, 5)`): comprueba si un valor está en un grupo.

Y en su forma más moderna:

```python
def tipo_de_dia(dia):
    match dia:
        case 1 | 2 | 3 | 4 | 5:
            return "laborable"
        case 6 | 7:
            return "fin de semana"
        case _:
            return "día no válido"


if __name__ == "__main__":
    for dia in [1, 3, 6, 7, 9]:
        print(f"{dia} -> {tipo_de_dia(dia)}")
```

Desde Python 3.10 existe **`match`/`case`**, parecido al `switch` de otros lenguajes pero mucho más potente (también reconoce estructuras). `|` une varias alternativas y `_` es el caso «cualquier otro». No hace falta `break`: se ejecuta solo el primer caso que coincide.

Salida:

```text
1 -> laborable
3 -> laborable
6 -> fin de semana
7 -> fin de semana
9 -> día no válido
```

Las dos versiones producen la misma salida.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Condiciones en mal orden, y una rama nunca se ejecuta | Prueba con un valor de cada rama y los valores **límite** (aquí 4, 5, 9 y 10) |
| Confundir `=` con `==` | Léelo en voz alta: «es igual a» |
| Olvidar el caso por defecto (`else`, `default`, `_`) | Pregúntate qué pasa con un dato inesperado |
| Condiciones repetidas con `||` que podrían ser un `switch` | Si comparas la misma variable con muchos valores, usa selección múltiple |

## Para practicar

Haz los [ejercicios 2.1 de condicionales](condicionales.md). Empieza por los primeros y, cuando funcionen, prueba cada uno con un valor de cada rama y con los límites.
