# A2.B Restricciones y variación

## Restringir el tipo

Si una función genérica necesita **hacer algo** con sus valores, como compararlos o sumarlos, tiene que saber que existe esa operación. Se lo cuentas con una **restricción**.

En Python se restringe con **`bound=`** (un tipo máximo) o con una lista cerrada de tipos: `TypeVar("Numero", int, float)`. Para «cualquier tipo que sepa compararse» se declara un **`Protocol`** con el método `__lt__`. Es lo que lee `mypy`; al ejecutar, nada de esto se comprueba.

```python
# Restricciones: se anotan con "bound" (un tipo máximo) o con una lista cerrada de tipos.
# Igual que en cualquier anotación de Python, las comprueba mypy o el editor, no el intérprete.
from typing import Callable, Protocol, TypeVar


class Comparable(Protocol):
    """Cualquier tipo que sepa decir si es menor que otro (int, float, str...)."""

    def __lt__(self, otro, /) -> bool: ...


C = TypeVar("C", bound=Comparable)
Numero = TypeVar("Numero", int, float)  # solo int o float
T = TypeVar("T")


def maximo(lista: list[C]) -> C:
    mayor = lista[0]
    for x in lista:
        if mayor < x:
            mayor = x
    return mayor


def sumar(lista: list[Numero]) -> float:
    return float(sum(lista))


# Sin restricción: acepta cualquier T y una función que lo examina
def contar_si(lista: list[T], cumple: Callable[[T], bool]) -> int:
    return sum(1 for x in lista if cumple(x))


print(f"maximo de [3, 9, 4]: {maximo([3, 9, 4])}")
print(f"maximo de [pera, manzana, uva]: {maximo(['pera', 'manzana', 'uva'])}")
print(f"suma de [1, 2, 3]: {sumar([1, 2, 3])}")
print(f"suma de [0.5, 0.25]: {sumar([0.5, 0.25])}")
print(f"pares en [1..6]: {contar_si([1, 2, 3, 4, 5, 6], lambda n: n % 2 == 0)}")
# maximo([object()]) lo marcaría mypy: object no define __lt__
```

Salida:

```text
maximo de [3, 9, 4]: 9
maximo de [pera, manzana, uva]: uva
suma de [1, 2, 3]: 6.0
suma de [0.5, 0.25]: 0.75
pares en [1..6]: 3
```

| Función | Qué demuestra |
|---|---|
| `maximo` | Funciona con enteros **y** con textos, pero solo con tipos que se pueden comparar |
| `sumar` | Una restricción a **números**: acepta enteros y decimales |
| `contarSi` | **Sin** restricción: no necesita saber nada de `T`, porque delega en la función que recibe |

## ¿Una lista de enteros es una lista de números?

Parece que sí, pero depende de si la lista se puede **modificar**. Cada lenguaje lo resuelve distinto:

Para el comprobador de tipos, `list` es **invariante** (una `list[int]` no vale donde se espera `list[float]`, porque podrías añadir un `float`), mientras que `Sequence` y `tuple` solo se leen y son **covariantes**. Al ejecutar no hay diferencia: es otra comprobación que hace `mypy`, no el intérprete.

!!! tip "Cómo decidir en Python"
    Pregúntate si tu función **solo lee** de la colección, solo **escribe** o hace las dos cosas. Si solo lee, acepta el tipo más amplio que puedas; si hace las dos, el tipo tiene que ser exacto.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Restringir demasiado (por ejemplo, pedir `ArrayList` cuando vale cualquier lista) | Pide la interfaz más general que necesites |
| Llamar a un método que la restricción no garantiza | Añade la restricción que lo garantice, o recibe una función como parámetro |
| Querer escribir en una colección que se ha recibido como «solo lectura» | Recibe una colección modificable del tipo exacto |

## Para practicar

Los ejercicios [A2.3 y A2.5](ejercicios.md) usan una restricción y una función como criterio. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
