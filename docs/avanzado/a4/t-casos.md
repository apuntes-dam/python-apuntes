# A4.B Errores y casos límite

El código casi nunca falla con los datos «normales»: falla en los **bordes** (el primero, el último, el cero, el vacío) y con los **datos inválidos**. Esas son las pruebas que más valen.

## Probar que algo falla

También hay que comprobar que el código **rechaza** lo incorrecto, con el error y el mensaje adecuados.

`with self.assertRaises(ValueError) as contexto:` comprueba que el bloque **lanza** esa excepción; después, `str(contexto.exception)` es su mensaje.

## Casos límite

Un **caso límite** es un valor en la frontera entre dos comportamientos. Si un descuento va de `0` a `100`, prueba `0` y `100` (los extremos válidos) y `-1` y `101` (justo fuera). Los errores de «uno de más o de menos» (`<` en vez de `<=`) viven ahí.

| Qué pruebas | Valores típicos |
|---|---|
| Números con un rango | El mínimo, el máximo, justo debajo y justo encima |
| Colecciones y texto | **Vacío**, un solo elemento, muchos |
| Búsquedas | El primero, el último, uno que **no está** |
| Divisiones y porcentajes | Cero, uno, el límite |

## Varios casos con una tabla

Cuando la comprobación es la misma y solo cambian los datos, se escribe **una prueba** que recorre una **tabla de casos** (entrada y resultado esperado):

Un diccionario y un bucle dentro de **una** prueba; **`self.subTest(...)`** hace que, si un caso falla, se vea cuál y se sigan probando los demás.

## El ejemplo

Las pruebas del carrito: dos para los errores, una con la tabla de descuentos y otra para el porcentaje fuera de rango.

**`test_carrito_casos.py`**

```python
import unittest

from carrito import Carrito


class PruebasCarritoCasos(unittest.TestCase):
    def setUp(self):
        self.carrito = Carrito()

    def test_una_cantidad_cero_o_negativa_lanza_un_error_con_un_mensaje_claro(self):
        # assertRaises como «with» comprueba que se lanza la excepción y deja mirar su mensaje
        with self.assertRaises(ValueError) as contexto:
            self.carrito.agregar("tarta", 1800, 0)
        self.assertEqual(str(contexto.exception), "la cantidad debe ser mayor que cero")
        with self.assertRaises(ValueError):
            self.carrito.agregar("tarta", 1800, -1)

    def test_un_precio_negativo_lanza_un_error(self):
        with self.assertRaises(ValueError):
            self.carrito.agregar("tarta", -5, 1)

    def test_descuentos_de_0_10_50_y_100_por_ciento_casos_limite_incluidos(self):
        self.carrito.agregar("tarta", 1800, 2)
        self.carrito.agregar("galleta", 100, 12)
        casos = {0: 4800, 10: 4320, 50: 2400, 100: 0}  # porcentaje -> total esperado
        for porcentaje, esperado in casos.items():
            # subTest hace que, si un caso falla, se vea cuál y se sigan probando los demás
            with self.subTest(porcentaje=porcentaje):
                self.assertEqual(self.carrito.con_descuento(porcentaje), esperado)

    def test_un_porcentaje_fuera_de_0_a_100_lanza_un_error(self):
        with self.assertRaises(ValueError):
            self.carrito.con_descuento(101)
        with self.assertRaises(ValueError):
            self.carrito.con_descuento(-1)


if __name__ == "__main__":
    unittest.main()
```

Al ejecutar las pruebas:

```text
test_descuentos_de_0_10_50_y_100_por_ciento_casos_limite_incluidos (test_carrito_casos.PruebasCarritoCasos.test_descuentos_de_0_10_50_y_100_por_ciento_casos_limite_incluidos) ... ok
test_un_porcentaje_fuera_de_0_a_100_lanza_un_error (test_carrito_casos.PruebasCarritoCasos.test_un_porcentaje_fuera_de_0_a_100_lanza_un_error) ... ok
test_un_precio_negativo_lanza_un_error (test_carrito_casos.PruebasCarritoCasos.test_un_precio_negativo_lanza_un_error) ... ok
test_una_cantidad_cero_o_negativa_lanza_un_error_con_un_mensaje_claro (test_carrito_casos.PruebasCarritoCasos.test_una_cantidad_cero_o_negativa_lanza_un_error_con_un_mensaje_claro) ... ok
----------------------------------------------------------------------
Ran 4 tests
OK
```

Los descuentos esperados se escribieron **a mano** (`4800 → 4320` con un `10 %`): si copiaras la fórmula del código en la prueba, repetirías también sus errores.

!!! warning "Cobertura no es calidad"
    Un programa de pruebas puede **ejecutar** todas las líneas del código y aun así no comprobar casi nada. Mide cuántos **casos importantes** tienes cubiertos (bordes, errores, datos raros), no solo cuántas líneas pasan por las pruebas.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Probar solo el caso feliz | Añade siempre al menos un borde y un error |
| Comprobar que «lanza algún error» sin mirar cuál | Comprueba el **tipo** y, si importa, el **mensaje** |
| Un bucle de casos donde el primer fallo oculta a los demás | Usa el mecanismo del marco para indicar el caso (`reason`, `subTest`...) |
| Escribir las pruebas copiando la implementación | Calcula el resultado esperado a mano o con otra vía |

## Para practicar

Los ejercicios [A4.2, A4.3 y A4.5](ejercicios.md) piden pruebas de casos normales, de errores y de límites con una tabla. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
