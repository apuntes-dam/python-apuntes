# A4.C Aislar lo que no controlas

Una prueba tiene que dar **siempre el mismo resultado**. Pero hay código que depende de cosas que **no controlas**: la hora, el azar, una base de datos, una conexión a internet, un archivo. Si tu código llama a esas cosas directamente, sus pruebas **pasarán o fallarán según el día o la hora**.

## La solución: pedir las dependencias desde fuera

En vez de que la tienda pregunte la hora al sistema, **recibe un reloj** al crearse. Es la **inyección de dependencias** (la «D» de [SOLID](../../u06/t-solid-2.md)). En el programa real se le da el reloj verdadero; en las pruebas, uno **falso** que devuelve la hora que haga falta.

El reloj es simplemente una **función** que devuelve la hora, así que uno falso es `lambda: 15`: «siempre las 15». Python no necesita interfaces para esto.

**`tienda.py`**

```python
"""Código que depende de la HORA. Si llamara a datetime.now() directamente, sus pruebas dependerían de cuándo se ejecuten.
Solución: recibir un reloj desde fuera (inyección de dependencias) y, en las pruebas, darle uno falso.
Aquí el reloj es simplemente una función que devuelve la hora."""
from datetime import datetime


def reloj_real():
    return datetime.now().hour


class Tienda:
    def __init__(self, reloj=reloj_real):
        self._reloj = reloj

    @property
    def saludo(self):
        h = self._reloj()
        if h < 12:
            return "buenos días"
        if h < 20:
            return "buenas tardes"
        return "buenas noches"

    @property
    def abierta(self):
        return 9 <= self._reloj() < 21
```

**`test_tienda.py`**

```python
import unittest

from tienda import Tienda


class PruebasTienda(unittest.TestCase):
    # un reloj falso es una función que devuelve siempre la hora que la prueba quiera: lambda: 8 es «siempre las 8»
    def test_por_la_manana_saluda_con_buenos_dias_y_la_tienda_esta_cerrada_a_las_8(self):
        tienda = Tienda(lambda: 8)
        self.assertEqual(tienda.saludo, "buenos días")
        self.assertFalse(tienda.abierta)

    def test_por_la_tarde_saluda_con_buenas_tardes_y_esta_abierta(self):
        tienda = Tienda(lambda: 15)
        self.assertEqual(tienda.saludo, "buenas tardes")
        self.assertTrue(tienda.abierta)

    def test_por_la_noche_saluda_con_buenas_noches_y_esta_cerrada(self):
        tienda = Tienda(lambda: 22)
        self.assertEqual(tienda.saludo, "buenas noches")
        self.assertFalse(tienda.abierta)

    def test_abre_a_las_9_y_cierra_a_las_21_casos_limite(self):
        self.assertFalse(Tienda(lambda: 8).abierta)
        self.assertTrue(Tienda(lambda: 9).abierta)
        self.assertTrue(Tienda(lambda: 20).abierta)
        self.assertFalse(Tienda(lambda: 21).abierta)


if __name__ == "__main__":
    unittest.main()
```

Al ejecutar las pruebas:

```text
test_abre_a_las_9_y_cierra_a_las_21_casos_limite (test_tienda.PruebasTienda.test_abre_a_las_9_y_cierra_a_las_21_casos_limite) ... ok
test_por_la_manana_saluda_con_buenos_dias_y_la_tienda_esta_cerrada_a_las_8 (test_tienda.PruebasTienda.test_por_la_manana_saluda_con_buenos_dias_y_la_tienda_esta_cerrada_a_las_8) ... ok
test_por_la_noche_saluda_con_buenas_noches_y_esta_cerrada (test_tienda.PruebasTienda.test_por_la_noche_saluda_con_buenas_noches_y_esta_cerrada) ... ok
test_por_la_tarde_saluda_con_buenas_tardes_y_esta_abierta (test_tienda.PruebasTienda.test_por_la_tarde_saluda_con_buenas_tardes_y_esta_abierta) ... ok
----------------------------------------------------------------------
Ran 4 tests
OK
```

Las pruebas dan el mismo resultado **a las 3 de la mañana y a las 3 de la tarde**, porque la hora la decide cada prueba. Sin el reloj falso habría que esperar al día siguiente para probar «buenas noches».

## Dobles de prueba

Un objeto que sustituye a otro en las pruebas se llama **doble de prueba**. Hay varios tipos:

| Tipo | Qué hace | Ejemplo |
|---|---|---|
| **Falso** (*fake*) | Una versión simple pero que funciona | Una base de datos en memoria; el reloj de arriba |
| **Sustituto** (*stub*) | Devuelve respuestas fijas | Un servicio que siempre contesta «aprobado» |
| **Espía / simulado** (*mock*) | Además, **registra cómo lo llamaron** | Comprobar que se envió exactamente un correo |

Empieza siempre por el más simple (un falso hecho a mano). Las bibliotecas de simulación solo hacen falta cuando hay muchas dependencias.

## Qué probar y qué no

| Prueba | Qué comprueba | Cuántas |
|---|---|---|
| **Unitarias** | Una función o clase **sola** (las de esta unidad) | Muchas: son rápidas y baratas |
| **De integración** | Varias piezas juntas (por ejemplo, tu código con una base de datos real) | Menos |
| **De extremo a extremo** | El programa entero, como lo usaría una persona | Pocas: son lentas y frágiles |

!!! tip "Escribir la prueba primero"
    En el **desarrollo guiado por pruebas** (*TDD*) el ciclo es: 1) escribes una prueba que **falla**, 2) escribes el código **mínimo** para que pase, 3) **mejoras** el código sin que las pruebas dejen de pasar. Obliga a pensar primero qué debe hacer el código y deja una red de seguridad desde el principio.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Código que llama a la hora, al azar o a la red directamente | Recíbelo como parámetro o dependencia |
| Pruebas que fallan «a veces» | Busca lo que no controlas: hora, azar, orden, red |
| Falsos tan complicados que hay que probarlos a ellos | Mantenlos mínimos |
| Probar detalles internos en lugar del comportamiento | Prueba lo que el código **hace**, no cómo lo hace |

## Para practicar

El ejercicio [A4.6](ejercicios.md) pide probar un saludo con un reloj falso. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
