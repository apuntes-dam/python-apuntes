# 4.C Constructores y enumerados

## Más de una forma de crear un objeto

A veces un objeto se puede crear de varias maneras: con unos valores por defecto, con datos propios o a partir de una «receta» ya preparada. Cada lenguaje lo resuelve a su manera.

Python solo tiene **un** `__init__`, así que se usan **parámetros con valor por defecto** (`nombre="Margarita"`), que se pueden pasar por nombre. Para constructores alternativos con otro significado se usan **métodos de clase** con `@classmethod` (`Pizza.cuatro_quesos(...)`), que reciben la clase como `cls` y devuelven un objeto.

```python
from enum import Enum


class Tamano(Enum):
    PEQUENA = ("Pequeña", 6)
    MEDIANA = ("Mediana", 8)
    GRANDE = ("Grande", 10)

    def __init__(self, etiqueta, precio_base):
        self.etiqueta = etiqueta
        self.precio_base = precio_base


class Pizza:
    def __init__(self, tamano, nombre="Margarita", extras=0):
        self.tamano = tamano
        self.nombre = nombre
        self.extras = extras

    @classmethod
    def cuatro_quesos(cls, tamano):
        return cls(tamano, "Cuatro quesos", 3)

    def anadir_extra(self, cantidad=1):
        self.extras += cantidad

    def precio(self):
        return self.tamano.precio_base + self.extras

    def __str__(self):
        return f"Pizza({self.nombre}, {self.tamano.etiqueta}, {self.extras} extras, {self.precio()} €)"


if __name__ == "__main__":
    a = Pizza(Tamano.MEDIANA)
    b = Pizza.cuatro_quesos(Tamano.GRANDE)
    c = Pizza(Tamano.PEQUENA, "Barbacoa")
    print(a)
    print(b)
    print(c)
    a.anadir_extra()
    a.anadir_extra(2)
    print(a)
    print(f"total del pedido: {a.precio() + b.precio() + c.precio()} €")
    print("tamaños: " + ", ".join(t.etiqueta for t in Tamano))
```

Salida:

```text
Pizza(Margarita, Mediana, 0 extras, 8 €)
Pizza(Cuatro quesos, Grande, 3 extras, 13 €)
Pizza(Barbacoa, Pequeña, 0 extras, 6 €)
Pizza(Margarita, Mediana, 3 extras, 11 €)
total del pedido: 30 €
tamaños: Pequeña, Mediana, Grande
```

En este ejemplo hay tres formas de obtener una pizza: **solo con el tamaño** (el nombre por defecto es «Margarita»), **con nombre propio** (`Barbacoa`) y la receta ya preparada con un **constructor alternativo** (`cuatroQuesos`, que además trae tres extras). Después, `anadirExtra` se llama **con y sin argumento**.

Para un método que se puede llamar con o sin argumento, como `anadir_extra()` y `anadir_extra(2)`, se usa **un solo método** con un parámetro con valor por defecto (`cantidad=1`).

!!! tip "Un solo sitio para las reglas"
    Procura que **todos los constructores acaben pasando por el mismo código** (un constructor principal al que los demás llaman): así cualquier regla que añadas más adelante (por ejemplo, un máximo de extras) se escribe **una sola vez**.

## Enumerados

Un **enumerado** es un tipo con un **conjunto fijo y cerrado de valores**: los días de la semana, los estados de un pedido, los tamaños de una pizza. Es mejor que usar números o textos sueltos porque el compilador impide valores inventados (`Tamano.gigante` no existe) y el código se lee solo.

Un `Enum` en Python puede tener **datos asociados**: se dan como tupla y se recogen en `__init__`. Se recorre con `for t in Tamano`. Por convención los miembros van en MAYÚSCULAS.

En el ejemplo, cada tamaño lleva **datos asociados** (la etiqueta que se muestra y el precio base), de modo que el resto del programa no necesita un `if` por cada tamaño: pregunta el dato al propio valor.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Repetir las comprobaciones en cada constructor | Un constructor principal y los demás que lo llaman |
| Dos constructores casi iguales con distinto orden de parámetros | Usa valores por defecto o un método de fábrica con nombre claro |
| Usar números o textos para representar categorías (`tipo = 1`) | Un enumerado |
| Cadenas de `if`/`else` según el valor de un enumerado | Pon el dato o el comportamiento **dentro** del enumerado |

## Para practicar

Los constructores, los enumerados y la sobrecarga se practican en [U4.6 · Prueba](prueba.md). Compara cómo se escribe en otro lenguaje con [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
