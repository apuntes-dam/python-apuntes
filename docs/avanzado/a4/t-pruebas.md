# A4.A Qué es una prueba y cómo se escribe

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5. La [unidad 1](../../u01/04-pruebas.md) ya te enseñó una primera prueba; aquí aprendes a escribirlas con método.

Una **prueba automática** es un programa corto que ejecuta tu código con unos datos **y comprueba el resultado por ti**. Lo que compruebas a mano una vez, la prueba lo comprueba **todas las veces**, en segundos, cada vez que cambias algo. Si rompes algo sin querer, lo sabes **al momento** y no dentro de una semana.

## La receta: preparar, actuar, comprobar

Casi todas las pruebas siguen tres pasos (*Arrange, Act, Assert*):

| Paso | Qué haces | En el ejemplo |
|---|---|---|
| **Preparar** | Creas los objetos y los datos | Un carrito nuevo |
| **Actuar** | Llamas al código que se prueba | `agregar('tarta', 1800, 2)` |
| **Comprobar** | Dices qué esperas | El total es `3600` |

Una buena prueba **comprueba una sola cosa**, tiene un **nombre que dice lo que debe ocurrir** y **no depende de otra prueba**.

## El marco de pruebas de Python

Python trae **`unittest`** en su biblioteca estándar. Una clase que hereda de `unittest.TestCase` agrupa pruebas, y cada método cuyo nombre empieza por **`test`** es una. Las comprobaciones son `assertEqual`, `assertTrue`... Se ejecuta con **`python -m unittest -v`**. Otra opción muy usada es **pytest** (unidad 1), que hace lo mismo con `assert` simples.

## Un ejemplo: un carrito de la compra

Primero, el código que se prueba. Los precios van en **céntimos** (enteros) para evitar errores de decimales:

**`carrito.py`**

```python
"""El código que se prueba: un carrito de la compra.
Los precios van en céntimos (números enteros) para evitar errores de redondeo con decimales."""


class Carrito:
    def __init__(self):
        self._lineas = []

    def agregar(self, producto, precio, cantidad):
        if cantidad <= 0:
            raise ValueError("la cantidad debe ser mayor que cero")
        if precio < 0:
            raise ValueError("el precio no puede ser negativo")
        self._lineas.append((producto, precio, cantidad))

    @property
    def total(self):
        return sum(precio * cantidad for _, precio, cantidad in self._lineas)

    @property
    def unidades(self):
        return sum(cantidad for _, _, cantidad in self._lineas)

    @property
    def vacio(self):
        return not self._lineas

    def con_descuento(self, porcentaje):
        """El total después de aplicar un descuento de 0 a 100 por ciento."""
        if not 0 <= porcentaje <= 100:
            raise ValueError("el porcentaje debe estar entre 0 y 100")
        return self.total - self.total * porcentaje // 100
```

Y sus tres primeras pruebas:

**`test_carrito.py`**

```python
import unittest

from carrito import Carrito


# unittest viene con Python: cada método que empieza por «test» es una prueba; setUp se ejecuta antes de CADA una.
class PruebasCarrito(unittest.TestCase):
    def setUp(self):
        self.carrito = Carrito()  # un carrito nuevo antes de CADA prueba: así no dependen unas de otras

    def test_un_carrito_nuevo_esta_vacio_y_su_total_es_0(self):
        self.assertTrue(self.carrito.vacio)
        self.assertEqual(self.carrito.total, 0)

    def test_agregar_un_producto_suma_su_importe(self):
        self.carrito.agregar("tarta", 1800, 2)
        self.assertEqual(self.carrito.total, 3600)
        self.assertEqual(self.carrito.unidades, 2)

    def test_con_varios_productos_el_total_es_la_suma_de_los_importes(self):
        self.carrito.agregar("tarta", 1800, 2)
        self.carrito.agregar("galleta", 100, 12)
        self.assertEqual(self.carrito.total, 4800)


if __name__ == "__main__":
    unittest.main()
```

El método **`setUp`** se ejecuta **antes de cada prueba**. Por eso el carrito se vuelve a crear: ninguna prueba hereda lo que hizo la anterior.

Al ejecutar las pruebas:

```text
test_agregar_un_producto_suma_su_importe (test_carrito.PruebasCarrito.test_agregar_un_producto_suma_su_importe) ... ok
test_con_varios_productos_el_total_es_la_suma_de_los_importes (test_carrito.PruebasCarrito.test_con_varios_productos_el_total_es_la_suma_de_los_importes) ... ok
test_un_carrito_nuevo_esta_vacio_y_su_total_es_0 (test_carrito.PruebasCarrito.test_un_carrito_nuevo_esta_vacio_y_su_total_es_0) ... ok
----------------------------------------------------------------------
Ran 3 tests
OK
```

Cada línea acaba en `ok` si la prueba pasó, y el resumen dice `Ran 3 tests` y `OK`.

## Cuando una prueba falla

Para ver un fallo **de verdad**, rompí el código a propósito: cambié `precio × cantidad` por `precio + cantidad` en el cálculo del total. Esto es lo que se ve:

```text
FAIL: test_agregar_un_producto_suma_su_importe (...)
AssertionError: 1802 != 3600
```

Un buen mensaje de fallo dice **qué prueba falló**, **qué se esperaba** (`3600`) y **qué se obtuvo** (`1802`). Con eso suele bastar para encontrar el error sin depurar.

!!! tip "Una prueba que nunca ha fallado no demuestra nada"
    Cuando escribas una prueba, **rómpela una vez a propósito** (cambia el resultado esperado o el código) y comprueba que falla. Así sabes que de verdad está mirando algo.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Pruebas que dependen unas de otras (el orden importa) | Prepara de cero antes de cada prueba |
| Nombres como `test1`, `test2` | Describe el comportamiento: «agregar un producto suma su importe» |
| Comprobar muchas cosas distintas en una sola prueba | Una idea por prueba: si falla, sabes qué falló |
| Copiar en la prueba el mismo cálculo que hace el código | Escribe el resultado esperado **a mano** (`3600`), no `precio * cantidad` |

## Para practicar

Los ejercicios [A4.1 y A4.4](ejercicios.md) piden escribir una función con sus pruebas y una preparación previa. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
