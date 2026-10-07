# A3.C Compartir datos con seguridad

El mayor peligro de ejecutar cosas a la vez no es que falle algo de forma evidente, sino que **a veces** salga mal y **a veces** no. Son los fallos más difíciles de encontrar.

## La condición de carrera

En Python varios hilos **comparten** la memoria. `contador += 1` parece una operación, pero son varias (leer, sumar, escribir), y un hilo puede ser interrumpido a mitad: se produce una **condición de carrera**. El GIL **no lo evita**. Se arregla con un **candado**: `threading.Lock` y `with candado:` deja entrar a **un solo hilo a la vez**. En `asyncio` (un solo hilo) la tarea solo se interrumpe en un `await`, así que muchos contadores simples no necesitan candado.

El ejemplo hace que cuatro hilos sumen 1000 veces cada uno, y repite el experimento **tres veces**:

```python
# Cuatro hilos suman 1000 veces cada uno a un contador COMPARTIDO.
# «contador += 1» (leer, sumar, escribir) puede mezclarse entre hilos; el Lock deja entrar a uno solo cada vez.
# (El GIL de CPython hace que dos hilos no ejecuten código Python a la vez, pero NO evita este problema.)
import threading

contador = 0
candado = threading.Lock()


def sumar_mil():
    global contador
    for _ in range(1000):
        with candado:
            contador += 1


def ejecutar():
    global contador
    contador = 0
    hilos = [threading.Thread(target=sumar_mil) for _ in range(4)]
    for hilo in hilos:
        hilo.start()
    for hilo in hilos:
        hilo.join()  # esperar a que termine cada hilo
    return contador


resultados = [ejecutar() for _ in range(3)]
print("resultados de 3 ejecuciones:", " ".join(str(r) for r in resultados))
```

Salida:

```text
resultados de 3 ejecuciones: 4000 4000 4000
```

El resultado es **siempre 4000**. Sin protección, ejecutarlo varias veces daría **valores distintos y casi siempre menores**, porque se pierden sumas: lo peligroso es que **a veces coincidiría con 4000 por casualidad** y el fallo pasaría desapercibido.

!!! warning "Probar no demuestra que esté bien"
    Un programa concurrente que funcionó diez veces puede fallar la undécima. Razona sobre **qué datos se comparten** y **quién los modifica**, en vez de fiarte de las pruebas.

## Mejor que compartir: pasarse mensajes

Si dos tareas necesitan intercambiar datos, a menudo es más simple y seguro que **no compartan nada** y se manden mensajes por un canal o cola. Un **productor** genera datos, un **consumidor** los procesa.

Una **`asyncio.Queue`** es la cola de las corrutinas: el productor hace `put` y el consumidor `get`, que **espera** si está vacía. Un valor especial (aquí `None`) avisa de que **ya no hay más**. No hace falta ningún candado. (Para hilos, existe `queue.Queue`.)

```python
# Productor y consumidor: en lugar de compartir una variable, uno mete mensajes en una cola y el otro los saca.
# asyncio.Queue es la cola de las corrutinas: get() espera si está vacía.
import asyncio

FIN = None  # un valor especial que significa «ya no hay más»


async def productor(cola):
    for i in range(1, 4):
        await asyncio.sleep(0.05)  # tarda en tener el dato
        print(f"produce {i}")
        await cola.put(i)
    await cola.put(FIN)


async def consumidor(cola):
    suma = 0
    while True:
        n = await cola.get()  # espera hasta que haya algo
        if n is FIN:
            return suma
        print(f"consume {n}")
        suma += n


async def main():
    cola = asyncio.Queue()
    tarea = asyncio.create_task(productor(cola))
    suma = await consumidor(cola)
    await tarea
    print(f"suma: {suma}")


asyncio.run(main())
```

Salida:

```text
produce 1
consume 1
produce 2
consume 2
produce 3
consume 3
suma: 6
```

Mira cómo se intercalan `produce` y `consume`: cada dato se consume **en cuanto llega**, sin esperar a que se produzca el siguiente.

## Reglas para dormir tranquilo

| Regla | Por qué |
|---|---|
| **No compartas** si puedes evitarlo: pasa mensajes o devuelve resultados | Sin memoria compartida no hay carreras |
| Si compartes, que sea **inmutable** (unidad [A1.C](../a1/t-composicion.md)) | Lo que no cambia, no se pisa |
| Protege **todo el acceso** al dato compartido, lecturas incluidas | Un solo acceso sin candado reabre el problema |
| Mantén el candado **el menor tiempo posible** | Un candado largo convierte lo concurrente en secuencial |
| Si usas varios candados, **tómalos siempre en el mismo orden** | Si no, dos tareas pueden esperarse la una a la otra para siempre (*interbloqueo* o *deadlock*) |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Proteger la escritura pero no la lectura | Protege ambas con el mismo candado |
| Dar por válido un resultado porque «salió bien al probar» | Razona sobre qué se comparte, no solo pruebes |
| Olvidar avisar al consumidor de que ya no hay más datos | Usa un valor de fin, o cierra el canal |
| Mantener un candado mientras haces una espera larga | Calcula fuera y entra al candado solo para actualizar el dato |

## Para practicar

Los ejercicios [A3.5 y A3.6](ejercicios.md) usan un contador protegido y un productor con consumidor. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
