# A3.B Resultados, errores y esperas

Lanzar una tarea es solo la mitad: casi siempre querrás **recoger su resultado** y saber si **falló**. Lo que representa «un resultado que todavía no existe» recibe distintos nombres según el lenguaje, pero la idea es la misma.

## Resultados futuros en Python

Una función `async def` devuelve una **corrutina**, y `await` espera su valor **sin bloquear** el bucle. `asyncio.gather(*corrutinas)` las ejecuta a la vez y devuelve la lista de resultados **en el mismo orden**. Si una falla, `gather` propaga la **primera** excepción. Para limitar la espera: `asyncio.wait_for(corrutina, segundos)`, que lanza `asyncio.TimeoutError`.

| Qué | En Python |
|---|---|
| Un valor que llegará | corrutina / `Future` |
| Esperar el valor | `await c` |
| Lanzar varias y esperar todas | `asyncio.gather` |
| Límite de tiempo | `asyncio.wait_for` |

## Un ejemplo: tres consultas y un error

Se consultan los precios de tres productos **a la vez** (cada consulta tarda 100 ms), se suman, y después se pide un producto que no existe para ver cómo llega el error:

```python
# Tareas que DEVUELVEN un resultado (o un error): se lanzan todas a la vez y se recogen con await.
import asyncio

PRECIOS = {"pan": 2, "leche": 1, "queso": 5}


async def precio(producto):
    await asyncio.sleep(0.1)  # simula una consulta lenta
    if producto not in PRECIOS:
        raise ValueError(f"producto desconocido: {producto}")
    return PRECIOS[producto]


async def main():
    productos = ["pan", "leche", "queso"]

    # gather lanza las tres a la vez y devuelve una lista con los resultados, en el mismo orden
    resultados = await asyncio.gather(*(precio(p) for p in productos))
    for producto, p in zip(productos, resultados):
        print(f"precio de {producto}: {p}")
    print(f"total: {sum(resultados)}")

    # un error dentro de una tarea se lanza al hacer await, y se captura con try/except como siempre
    try:
        await precio("caviar")
    except ValueError as e:
        print(f"error: {e}")


asyncio.run(main())
```

Salida:

```text
precio de pan: 2
precio de leche: 1
precio de queso: 5
total: 8
error: producto desconocido: caviar
```

El error se lanza al hacer `await` (o en `gather`), como una excepción normal: se captura con `try`/`except`.

Los resultados se muestran **en el orden en que se pidieron**, aunque las consultas hayan terminado en otro. Es lo que se espera de una lista de resultados.

!!! tip "Esperar no es lo mismo que bloquear"
    `await`, `join` o `get` **esperan** al resultado. En los modelos con `await` (Dart, Kotlin, Python) esa espera deja libre al hilo para otras tareas. En `join()`/`get()` de Java **el hilo que espera se queda parado**: es lo normal en un programa de consola, pero no en una interfaz gráfica, que se congelaría.

## Límites de tiempo y cancelación

Una tarea que tarda demasiado puede dejar tu programa colgado. Todo lenguaje ofrece una **espera con límite** (mira la tabla de arriba): si se pasa el tiempo, obtienes un error o un valor vacío y decides qué hacer, por ejemplo **reintentar** o avisar. Cancelar la tarea libera recursos, pero en muchos casos solo se **solicita** y la tarea tiene que cooperar.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Lanzar varias tareas y esperarlas **una por una al lanzarlas** (cada una espera a la anterior) | Lánzalas todas primero y luego espera: así avanzan a la vez |
| Olvidar capturar el error de una tarea | El error llega al esperar el resultado: ahí va el `try` |
| Esperar sin límite una operación de red | Pon un límite de tiempo |
| Reintentar sin pausa ni tope | Limita el número de intentos (ejercicio [A3.4](ejercicios.md)) y espera entre ellos |

## Para practicar

Los ejercicios [A3.3 y A3.4](ejercicios.md) usan un límite de tiempo y reintentos. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
