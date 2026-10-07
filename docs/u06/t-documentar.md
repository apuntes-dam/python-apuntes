# 6.D Comentarios y documentación

Hay dos formas distintas de explicar el código, y conviene no mezclarlas:

* **Comentarios** (`//`, `#`): notas **para quien lea el código**, que explican cosas puntuales del interior.
* **Documentación** (comentarios especiales): descripción **de las clases y funciones públicas para quien las vaya a usar**, sin necesidad de leer su código. Una herramienta la convierte en páginas web.

## Un buen comentario explica el porqué

El código ya dice **qué** hace; un comentario solo aporta valor si cuenta lo que el código **no** puede decir: **por qué** se hizo así.

| Comentario | Valoración |
|---|---|
| `i = i + 1  // suma 1 a i` | **Sobra**: repite el código |
| `// bucle for` | **Sobra**: se ve a simple vista |
| `// la API devuelve las fechas en UTC: hay que convertir antes de comparar` | **Útil**: explica una razón que no se ve |
| `// TODO: cambiar cuando exista el servicio de pagos` | **Útil**, si se revisa y se acaba borrando |
| `// precio = precio * 1.1` (código desactivado) | **Sobra**: bórralo; el control de versiones ya guarda el historial |

!!! tip "Antes de comentar, mejora el nombre"
    Si necesitas un comentario para explicar qué es `p`, quizá lo que falta es llamarla `precioConIva`. Un código con buenos nombres y funciones cortas se explica casi solo, y los comentarios se reservan para el **porqué**.

## Documentar clases y funciones

Una buena documentación de una clase o función pública responde a cuatro preguntas: **qué hace**, **qué recibe**, **qué devuelve** y **qué errores puede dar**; y, si ayuda, un **ejemplo de uso**.

```python
class Conversor:
    """Convierte temperaturas entre grados Celsius y Fahrenheit.

    Ejemplo:
        >>> Conversor().celsius_a_fahrenheit(100)
        212
    """

    def celsius_a_fahrenheit(self, celsius):
        """Devuelve la temperatura en grados Fahrenheit equivalente a ``celsius``.

        Args:
            celsius: temperatura en grados Celsius (un entero).

        Returns:
            La temperatura en grados Fahrenheit (un entero).

        Raises:
            ValueError: si ``celsius`` está por debajo del cero absoluto (-273 °C).
        """
        if celsius < -273:
            raise ValueError("por debajo del cero absoluto")
        return celsius * 9 // 5 + 32


if __name__ == "__main__":
    conversor = Conversor()
    for c in [100, 0, -40]:
        print(f"{c} °C = {conversor.celsius_a_fahrenheit(c)} °F")
    try:
        conversor.celsius_a_fahrenheit(-300)
    except ValueError as e:
        print(f"Error: {e}")
```

Salida:

```text
100 °C = 212 °F
0 °C = 32 °F
-40 °C = -40 °F
Error: por debajo del cero absoluto
```

## Cómo se hace en Python

| Necesito... | En Python |
|---|---|
| Comentario de documentación | una **cadena `"""..."""`** (*docstring*) como **primera línea** de la clase o función |
| Estilo habitual | secciones `Args:`, `Returns:` y `Raises:` (estilo Google) |
| Ver la documentación | `help(Conversor)` en el intérprete; la lee también el editor |
| Probar los ejemplos de la documentación | `python -m doctest -v modulo.py` ejecuta las líneas que empiezan por `>>>` |
| Generar la documentación en HTML | `python -m pydoc -w modulo` (crea `modulo.html`); para sitios completos, Sphinx o mkdocstrings |

He comprobado que `python -m doctest` ejecuta el ejemplo de la documentación y pasa, y que `pydoc -w` genera el HTML.

## Qué documentar

* **Sí:** las clases y funciones **públicas**: lo que usarán otras personas (o tú mismo dentro de un mes).
* **Sí:** las excepciones que pueden lanzar y las **condiciones** que deben cumplir los datos de entrada.
* **No:** lo evidente (`/// Devuelve el nombre.` sobre `getNombre()`). Una documentación vacía es ruido.
* **Mantenla al día:** una documentación que dice algo distinto de lo que hace el código es peor que no tenerla.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Comentar cada línea | Solo el porqué de lo que no sea obvio |
| Documentar con frases que repiten el nombre del método | Decir algo que el nombre no diga: unidades, límites, errores |
| Dejar código antiguo comentado | Borrarlo (Git guarda el historial) |
| Cambiar el código y olvidar actualizar el comentario | Repasa los comentarios cuando modifiques lo que describen |

## Para practicar

Haz los ejercicios de [U6.3 · Comentarios y documentación](documentacion.md): documentar un código, generar la documentación y limpiar comentarios que sobran. [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) también incluye cómo se escriben los comentarios en cada lenguaje.
