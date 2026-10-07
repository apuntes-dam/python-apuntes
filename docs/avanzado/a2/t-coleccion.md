# A2.C Tu colección y qué pasa al ejecutar

## Una colección genérica propia

Las listas, mapas y conjuntos que ya usas son clases genéricas. Aquí escribimos una a mano: una **pila**, donde el último elemento en entrar es el primero en salir (como una pila de platos). La misma clase sirve para letras y para enteros:

```python
# Una colección genérica propia: una pila (el último en entrar es el primero en salir).
from typing import Generic, TypeVar

T = TypeVar("T")


class Pila(Generic[T]):
    def __init__(self):
        self._elementos: list[T] = []

    def apilar(self, x: T) -> None:
        self._elementos.append(x)

    def desapilar(self) -> T:
        if not self._elementos:
            raise IndexError("la pila está vacía")
        return self._elementos.pop()

    @property
    def tope(self) -> T:
        return self._elementos[-1]

    @property
    def tamano(self) -> int:
        return len(self._elementos)

    @property
    def vacia(self) -> bool:
        return not self._elementos


letras: Pila[str] = Pila()
for l in ["a", "b", "c"]:
    letras.apilar(l)
print(f"tamaño: {letras.tamano}")
print(f"tope: {letras.tope}")
print(f"sacar: {letras.desapilar()}")
print(f"sacar: {letras.desapilar()}")
print(f"quedan: {letras.tamano}")
print(f"vacía: {'sí' if letras.vacia else 'no'}")
print(f"sacar: {letras.desapilar()}")
try:
    letras.desapilar()
except IndexError as e:
    print(f"error: {e}")

# la misma clase, con otro tipo
numeros: Pila[int] = Pila()
for n in [1, 2, 3]:
    numeros.apilar(n)
salida = []
while not numeros.vacia:
    salida.append(numeros.desapilar())
print("pila de enteros al vaciarla:", " ".join(str(n) for n in salida))
```

Salida:

```text
tamaño: 3
tope: c
sacar: c
sacar: b
quedan: 1
vacía: no
sacar: a
error: la pila está vacía
pila de enteros al vaciarla: 3 2 1
```

Qué se ve en la salida:

1. Se apilan `a`, `b`, `c`; el **tope** es la última (`c`) y se sacan en orden contrario: `c`, `b`, `a`.
2. Intentar sacar de una pila **vacía** lanza una excepción con un mensaje claro, que el programa captura. No se devuelve un valor inventado.
3. Con una `Pila<Int>`, los números `1, 2, 3` salen como `3 2 1`: es la misma clase con otro tipo.

!!! tip "Una pila para algo útil"
    Una pila sirve para **deshacer** acciones, comprobar si los paréntesis de una expresión están equilibrados o recorrer estructuras sin recursión. El ejercicio [A2.4](ejercicios.md) la usa para invertir una lista.

## Qué pasa con los tipos al ejecutar

Los genéricos se comprueban **al compilar o al analizar el código**. Lo que ocurre después, al ejecutar, **depende del lenguaje**:

En Python los tipos genéricos **no existen al ejecutar**: son anotaciones. Una lista es una `list` aunque la hayas anotado como `list[int]`, y nada impide meterle un texto. Quien vigila los tipos es el comprobador (`mypy`, el editor), no el intérprete.

```python
# En Python los tipos genéricos NO existen al ejecutar: son solo anotaciones para el editor y mypy.
enteros: list[int] = [1, 2]
textos: list[str] = ["a"]

print("type(enteros):", type(enteros))
print("type(textos):", type(textos))
print("¿misma clase en ejecución?", "sí" if type(enteros) is type(textos) else "no")

# nada impide mezclar tipos: Python no lo comprueba al ejecutar
enteros.append("tres")
print("lista de enteros ahora:", enteros)
```

Salida:

```text
type(enteros): <class 'list'>
type(textos): <class 'list'>
¿misma clase en ejecución? sí
lista de enteros ahora: [1, 2, 'tres']
```

Si necesitas validar tipos al ejecutar, hay que comprobarlos tú (`isinstance(x, int)`) o usar una biblioteca de validación.

!!! warning "Por eso no se mezclan en la misma lista sin querer"
    En los lenguajes que comprueban al compilar, intentar meter un texto en una lista de enteros ni siquiera compila. En Python, el programa se ejecuta igualmente y el fallo aparece más tarde, cuando otra parte del código espera un número.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Devolver un valor inventado al sacar de una colección vacía | Lanza una excepción con un mensaje claro o devuelve un valor opcional |
| Esperar poder preguntar `¿es una lista de enteros?` al ejecutar en todos los lenguajes | Depende del lenguaje; si lo necesitas, guarda el tipo tú mismo |
| Escribir una colección propia cuando ya existe una en la biblioteca | Usa las del lenguaje salvo que quieras aprender o necesites un comportamiento distinto |

## Para practicar

Los ejercicios [A2.4 y A2.6](ejercicios.md) usan una pila y una caché genéricas. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
