# A3.A Tareas a la vez

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5 y la [unidad A1 de funciones como valores](../a1/index.md), porque las tareas se pasan como funciones.

Un programa **concurrente** tiene varias tareas **en marcha a la vez**. No siempre significa ejecutar a la vez de verdad: un solo procesador puede **alternar** entre tareas tan rápido que parece simultáneo. Cuando varias tareas se ejecutan **realmente** a la vez en varios núcleos, se habla de **paralelismo**.

| Tipo de tarea | Qué hace casi todo el tiempo | Ejemplos | Qué ayuda |
|---|---|---|---|
| **De espera** (*I/O-bound*) | **Esperar** a algo externo | Descargar, consultar una base de datos, leer un archivo | Alternar tareas (concurrencia) |
| **De cálculo** (*CPU-bound*) | **Calcular** sin parar | Comprimir, procesar una imagen, recorrer millones de datos | Repartir entre núcleos (paralelismo) |

## El modelo de Python

Python ofrece tres caminos. **`asyncio`**: un solo hilo y un bucle de eventos, con `async`/`await`, ideal para **muchas esperas** (red, ficheros). **`threading`**: varios hilos, útiles también cuando se espera, pero en CPython el **GIL** hace que **dos hilos no ejecuten código Python a la vez**, así que no aceleran el cálculo. **`multiprocessing`** (o `ProcessPoolExecutor`): varios procesos, cada uno con su intérprete, que sí aprovechan varios núcleos para cálculo pesado.

## Un ejemplo: tres tareas a la vez

Tres tareas simulan descargas de distinta duración (300, 100 y 200 ms). Se lanzan **todas a la vez**, se espera a que acaben y se comprueba el tiempo total:

```python
# Tres tareas que "esperan" (como una descarga) y avanzan a la vez, con asyncio (un solo hilo y un bucle de eventos).
import asyncio
import time


async def tarea(nombre, ms):
    await asyncio.sleep(ms / 1000)  # se pausa la corrutina, no el programa
    print(f"termina {nombre}")


async def main():
    inicio = time.perf_counter()
    tareas = []
    for nombre, ms in [("A", 300), ("B", 100), ("C", 200)]:
        print(f"empieza {nombre}")
        tareas.append(asyncio.create_task(tarea(nombre, ms)))  # se programa y todavía NO se espera
    await asyncio.gather(*tareas)  # ahora sí: esperar a las tres

    ms = (time.perf_counter() - inicio) * 1000
    print(f"las tres tareas juntas tardaron menos de 450 ms: {'sí' if ms < 450 else 'no'}")


asyncio.run(main())
```

Salida:

```text
empieza A
empieza B
empieza C
termina B
termina C
termina A
las tres tareas juntas tardaron menos de 450 ms: sí
```

`await asyncio.sleep` pausa la **corrutina**, no el programa. Si usaras `time.sleep` dentro de una corrutina, bloquearías todo el bucle y las tareas irían **una tras otra**.

Fíjate en dos cosas de la salida:

1. **Empiezan todas antes de que termine ninguna.** Se lanzan sin esperar.
2. **Terminan por duración, no por orden de lanzamiento**: B (100 ms), luego C (200 ms) y por último A (300 ms). El tiempo total es el de la **más lenta**, unos 300 ms, no la suma de las tres (600 ms).

!!! warning "Un orden que depende del tiempo"
    El orden en que terminan las tareas **no está garantizado por el lenguaje**: depende de cuánto tarden de verdad. En este ejemplo las pausas están muy separadas para que el resultado sea siempre el mismo. En un programa real, no escribas código que dependa de qué tarea termina antes.

## Qué usar en cada caso

| Si necesitas | En Python |
|---|---|
| Esperar red, archivos o temporizadores | `asyncio` |
| Esperas con bibliotecas que no son `async` | `threading` o `ThreadPoolExecutor` |
| Cálculo pesado | `multiprocessing` / `ProcessPoolExecutor` |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Bloquear con una espera normal (`sleep`) dentro de una tarea asíncrona | Usa la espera propia del modelo (`delay`, `asyncio.sleep`, `Future.delayed`) |
| Lanzar una tarea y no esperarla | Guarda el `Future`/`Job`/tarea y espera a que termine |
| Dar por hecho el orden en que terminan | Si importa el orden, espera a cada una por separado |
| Crear un hilo por cada tarea pequeña | Usa un grupo de hilos o corrutinas |

## Para practicar

El ejercicio [A3.1](ejercicios.md) pide lanzar tres descargas a la vez, y el [A3.2](ejercicios.md) repartir una suma entre dos tareas. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
