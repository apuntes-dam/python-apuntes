# A2.A Tipos parametrizados

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5, sobre todo clases e interfaces. Si te haces un lío con los tipos, repasa [las unidades 4 y 5](../../unidades.md).

Una **clase genérica** o una **función genérica** se escribe una sola vez y sirve para **varios tipos**. En lugar de decir «una caja de enteros» y «una caja de textos» con dos clases, dices «una caja de `T`», donde `T` es un **tipo que se decide al usarla**.

## Por qué existe

Python no obliga a nada: una lista admite cualquier cosa y una función acepta cualquier argumento. Las anotaciones genéricas sirven para **documentar** y para que un comprobador (`mypy`, `pyright` o tu editor) avise de errores **antes** de ejecutar, aunque el intérprete las ignora.

## La sintaxis en Python

| Qué | Cómo se escribe en Python |
|---|---|
| Declarar el tipo variable | `T = TypeVar("T")` (de `typing`) |
| Clase genérica | `class Caja(Generic[T]): ...` |
| Función genérica | `def primero(lista: list[T]) -> T:` |
| Usarla | `numero: Caja[int] = Caja(42)` (anotación opcional) |
| Varios parámetros | `class Par(Generic[A, B])` |
| Sintaxis nueva (3.12+) | `class Caja[T]:` y `def primero[T](lista: list[T]) -> T:`, sin `TypeVar` |

Por convención, el parámetro de tipo se llama con una letra mayúscula: `T` (*type*), `E` (*element*), `K` y `V` (*key* y *value*), o `A`, `B` cuando hay varios.

## Un ejemplo

Una caja genérica, un par con dos tipos distintos y una función que devuelve el primer elemento de una lista de cualquier tipo:

```python
# Tipos genéricos: Caja[T] guarda un valor de cualquier tipo T.
# En Python los tipos son ANOTACIONES: las lee un comprobador como mypy o el editor, no las exige el intérprete.
from typing import Generic, TypeVar

T = TypeVar("T")
A = TypeVar("A")
B = TypeVar("B")


class Caja(Generic[T]):
    def __init__(self, valor: T):
        self.valor = valor


# Con dos parámetros de tipo: Par[A, B]
class Par(Generic[A, B]):
    def __init__(self, primero: A, segundo: B):
        self.primero = primero
        self.segundo = segundo

    def __str__(self):
        return f"({self.primero}, {self.segundo})"


# Función genérica: sirve para listas de cualquier tipo y devuelve ese mismo tipo
def primero(lista: list[T]) -> T:
    return lista[0]


texto = Caja("hola")  # un comprobador de tipos deduce Caja[str]
numero: Caja[int] = Caja(42)  # también se puede anotar a mano
print(f"caja de texto: {texto.valor}")
print(f"caja de entero: {numero.valor}")
print(f"par: {Par('Ana', 30)}")
print(f"primero de [3, 4, 5]: {primero([3, 4, 5])}")
print(f"primero de [a, b]: {primero(['a', 'b'])}")

# mypy avisaría de: n: int = texto.valor (es un str), pero el programa se ejecutaría igualmente
t: str = texto.valor
print(f"longitud: {len(t)}")
```

Salida:

```text
caja de texto: hola
caja de entero: 42
par: (Ana, 30)
primero de [3, 4, 5]: 3
primero de [a, b]: a
longitud: 4
```

Fíjate en lo que **no** hace falta: ni convertir tipos ni escribir una clase por cada tipo. La función `primero` devuelve un entero cuando le das enteros y un texto cuando le das textos, y el lenguaje lo sabe.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar `Object`/`Any`/`dynamic` «para que valga con todo» | Si lo único que cambia es el tipo, usa un parámetro de tipo |
| Escribir `T` y luego querer llamar a un método suyo | Sin restricción, de `T` no se sabe nada: mira [A2.B](t-restricciones.md) |
| Creer que un genérico existe igual al ejecutar en todos los lenguajes | No es así: mira [A2.C](t-coleccion.md) |

## Para practicar

Los ejercicios [A2.1 y A2.2](ejercicios.md) usan una función y una clase genéricas. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
