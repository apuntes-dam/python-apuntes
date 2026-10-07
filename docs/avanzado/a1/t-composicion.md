# A1.C Componer, esperar e inmutabilidad

Tres ideas que completan la programación funcional: **componer** funciones pequeñas en otras mayores, **calcular solo lo necesario** (evaluación perezosa) y **no modificar los datos** (inmutabilidad).

## Componer funciones

Si tienes funciones pequeñas que hacen una cosa cada una, puedes unirlas: la salida de una es la entrada de la siguiente. Eso es una **tubería**.

Python no trae un operador de composición, pero `functools.reduce` junta una lista de funciones en una sola y `functools.partial` fija argumentos de una función (**aplicación parcial**). Un **cierre** que devuelve otro cierre (`potencia(base)(exp)`) es lo que otros lenguajes llaman *currying*.

```python
# Funciones de orden superior: componer funciones, encadenarlas, fijar argumentos y devolver funciones.
import re
from functools import partial, reduce


def doble(n):
    return n * 2


def sumar3(n):
    return n + 3


# componer(f, g)(x) = f(g(x))
def componer(f, g):
    return lambda x: f(g(x))


# una "tubería": reduce junta todas las funciones de la lista en una sola
def tuberia(pasos):
    return reduce(lambda acc, paso: (lambda x: paso(acc(x))), pasos)


# función que devuelve otra función (currying): potencia(base)(exponente)
def potencia(base):
    return lambda exp: base ** exp


print(f"componer(doble, sumar3)(4) = {componer(doble, sumar3)(4)}")

slug = tuberia([str.strip, str.lower, lambda s: re.sub(r"\s+", "-", s)])
print(f'slug: "  Hola Mundo Cruel  " -> "{slug("  Hola Mundo Cruel  ")}"')


def sumar(a, b):
    return a + b


# aplicación parcial: partial fija el primer argumento
sumar5 = partial(sumar, 5)
print(f"sumar5(10) = {sumar5(10)}")
print(f"potencia(2)(10) = {potencia(2)(10)}")
```

Salida:

```text
componer(doble, sumar3)(4) = 14
slug: "  Hola Mundo Cruel  " -> "hola-mundo-cruel"
sumar5(10) = 15
potencia(2)(10) = 1024
```

La función `slug` convierte un título en una dirección web: quita los espacios de los extremos, pasa a minúsculas y cambia los espacios por guiones. Cada paso es una función de una línea, fácil de probar por separado.

## Calcular solo lo necesario

Una colección **ansiosa** calcula todos sus elementos al crearse. Una **perezosa** calcula cada elemento cuando alguien lo pide, y por eso puede ser **infinita**. En Python:

Una función con **`yield`** es un **generador**: entrega un valor y queda pausada hasta que piden el siguiente, así que puede describir una secuencia **infinita**. `filter` y `map` devuelven iteradores perezosos, e `itertools.islice` toma solo los primeros valores.

```python
# Evaluación perezosa: los valores se calculan solo cuando alguien los pide.
from itertools import islice


# una función con yield es un generador: entrega un valor y se queda "en pausa" hasta el siguiente
def naturales():
    n = 1
    while True:  # una secuencia infinita: nunca termina por sí sola
        yield n
        n += 1


def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b


def es_par(n):
    print(f"revisando {n}")
    return n % 2 == 0


# filter y map devuelven iteradores perezosos: todavía no se ha calculado nada
pares = map(lambda n: n * n, filter(es_par, naturales()))
print("(se define la secuencia: todavía no se ha calculado nada)")

# islice(…, 3) pide valores uno a uno hasta tener tres; aquí se hace el trabajo
print("primeros 3 pares al cuadrado:", list(islice(pares, 3)))
print("fibonacci:", list(islice(fibonacci(), 8)))
```

Salida:

```text
(se define la secuencia: todavía no se ha calculado nada)
revisando 1
revisando 2
revisando 3
revisando 4
revisando 5
revisando 6
primeros 3 pares al cuadrado: [4, 16, 36]
fibonacci: [0, 1, 1, 2, 3, 5, 8, 13]
```

Mira el orden de la salida. Primero aparece el aviso de que la secuencia está definida pero sin calcular; después, `revisando 1` a `revisando 6`: solo se revisaron los números necesarios para encontrar **tres pares**, y ni uno más. Eso es la evaluación perezosa.

## No modificar los datos

Una función es **pura** si su resultado depende solo de sus argumentos y no cambia nada fuera de ella. Las puras son más fáciles de entender y de probar, y también de usar **a la vez** en varios hilos (lo verás en A3). La forma de conseguirlo con datos es la **inmutabilidad**: en vez de cambiar un objeto, se crea otro con el cambio.

`@dataclass(frozen=True)` hace que asignar a un campo lance `FrozenInstanceError`, y **`dataclasses.replace(obj, campo=valor)`** crea una copia cambiando solo ese campo. Las **tuplas** no se pueden modificar: `lista + (4,)` crea otra tupla.

```python
# Inmutabilidad: en vez de modificar un objeto, se crea una copia con el cambio.
from dataclasses import dataclass, replace


# frozen=True: asignar a un campo lanza FrozenInstanceError
@dataclass(frozen=True)
class Jugador:
    nombre: str
    puntos: int


def describir(j):
    return f"{j.nombre} tiene {j.puntos} puntos"


# función pura: el resultado depende solo de los argumentos y no toca nada de fuera
def suma_pura(a, b):
    return a + b


# función impura: lee y modifica una variable externa; el mismo argumento da resultados distintos
total = 0


def sumar_al_total(x):
    global total
    total += x
    return total


original = Jugador("Ana", 10)
actualizado = replace(original, puntos=15)  # copia con el cambio
print("original:", describir(original))
print("actualizado:", describir(actualizado))
print("original sigue igual:", describir(original))

lista = (1, 2, 3)  # una tupla no se puede modificar
nueva = lista + (4,)
print("lista original:", list(lista))
print("lista nueva:", list(nueva))

print("pura:", suma_pura(2, 3), "y", suma_pura(2, 3))
print("impura:", sumar_al_total(3), "y", sumar_al_total(3))
```

Salida:

```text
original: Ana tiene 10 puntos
actualizado: Ana tiene 15 puntos
original sigue igual: Ana tiene 10 puntos
lista original: [1, 2, 3]
lista nueva: [1, 2, 3, 4]
pura: 5 y 5
impura: 3 y 6
```

Fíjate en las dos últimas líneas: la función **pura** da `5` las dos veces; la **impura** da `3` y luego `6`, porque guarda estado fuera de la función y el mismo argumento produce resultados distintos.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Esperar que una secuencia perezosa «ya esté calculada» | Recuerda que solo se calcula al pedirla; si hay efectos secundarios (como imprimir), salen más tarde |
| Recorrer una secuencia infinita sin límite | Corta siempre con `take`, `limit` o `islice` antes de convertir a lista |
| Modificar una lista que otra parte del programa también usa | Devuelve una copia nueva con el cambio |
| Mezclar funciones puras con efectos secundarios | Separa el cálculo (puro) de lo que imprime o guarda |

## Para practicar

Los ejercicios [A1.5 y A1.6](ejercicios.md) usan una tubería y una secuencia perezosa. Para ver estas ideas en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
