# A3 · Ejercicios de concurrencia y asincronía

<div class="ej-gate" data-unit="a3" data-nombre="A3 · Concurrencia y asincronía"></div>

Practica tareas simultáneas, resultados y errores, límites de tiempo, reintentos y datos compartidos. Cada ejercicio indica la **salida esperada**. Las esperas son de decenas de milisegundos, así que los resultados son siempre los mismos.

## Ejercicio A3.1

**Tres descargas a la vez.** Simula tres descargas con una pausa cada una: `uno` (200 ms), `dos` (50 ms) y `tres` (100 ms). Cada descarga imprime `termina <nombre>` al acabar.

Lánzalas **todas a la vez**, espera a que terminen y comprueba que el conjunto tardó **menos de 350 ms** (si fueran una tras otra tardarían unos 350 ms o más). Imprime `todas a la vez: sí` o `todas a la vez: no`.

**Salida esperada:**

```text
termina dos
termina tres
termina uno
todas a la vez: sí
```

!!! note "En Python"
    Usa `asyncio.sleep` y `asyncio.gather(...)`. Con `time.sleep` no habría concurrencia: bloquearía todo el programa.

<details class="sol" data-key="av/a3/A3.1">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import asyncio
import time
async def descarga(nombre, ms):
    await asyncio.sleep(ms / 1000)
    print(f"termina {nombre}")
# ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.2

**Sumar en paralelo.** Reparte la suma de los números del `1` al `100` entre **dos tareas** que trabajan a la vez: una suma del `1` al `50` y la otra del `51` al `100`. Espera las dos y muestra la suma total.

**Salida esperada:**

```text
5050
```

!!! note "En Python"
    `concurrent.futures.ThreadPoolExecutor` y `submit(...)`; `future.result()` recoge el valor. Para cálculo pesado de verdad, por el GIL, haría falta `ProcessPoolExecutor`.

<details class="sol" data-key="av/a3/A3.2">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>from concurrent.futures import ThreadPoolExecutor
def sumar(desde, hasta):
    return sum(range(desde, hasta + 1))
with ThreadPoolExecutor(max_workers=2) as grupo:
    primera = grupo.submit(sumar, 1, 50)
# ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.3

**Con límite de tiempo.** Escribe una función asíncrona `consulta(ms)` que espere `ms` milisegundos y devuelva `42`. Después escribe `intentar(ms, limiteMs)`, que espera el resultado de la consulta **como máximo** `limiteMs` milisegundos: si llega a tiempo imprime `resultado: 42`, y si no, imprime `tiempo agotado`.

Pruébala con `intentar(300, 100)` y con `intentar(50, 200)`.

**Salida esperada:**

```text
tiempo agotado
resultado: 42
```

!!! note "En Python"
    `asyncio.wait_for(corrutina, segundos)` lanza `asyncio.TimeoutError` si se pasa el tiempo (el límite va en **segundos**).

<details class="sol" data-key="av/a3/A3.3">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import asyncio
async def consulta(ms):
    await asyncio.sleep(ms / 1000)
    return 42
async def intentar(ms, limite_ms):
# ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.4

**Reintentos.** Escribe una operación asíncrona que **falle las dos primeras veces** que se llama y funcione la tercera (devolviendo el texto `listo`). Escribe `conReintentos(maximo)`, que llama a la operación y, si falla, la repite hasta `maximo` veces en total.

En cada intento imprime `intento N falló` o `intento N correcto`; al final, `resultado: listo`. Prueba con un máximo de 3.

**Salida esperada:**

```text
intento 1 falló
intento 2 falló
intento 3 correcto
resultado: listo
```

<details class="sol" data-key="av/a3/A3.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import asyncio
llamadas = 0
async def operacion():
    global llamadas
    await asyncio.sleep(0.01)
# ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.5

**Un contador compartido, bien protegido.** Lanza **5 tareas** que suman 200 veces cada una a un mismo contador, y muestra el valor final (siempre `1000`).

Hazlo de forma que **no se pierda ninguna suma** aunque las tareas se ejecuten a la vez.

**Salida esperada:**

```text
1000
```

!!! note "En Python"
    Un `threading.Lock` con `with candado:` alrededor de la suma.

<details class="sol" data-key="av/a3/A3.5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import threading
contador = 0
candado = threading.Lock()
def sumar_doscientos():
    global contador
# ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.6

**Productor y consumidor.** Un **productor** envía los números del `1` al `5` y un **consumidor** los va recibiendo y sumando. Se comunican por un **canal** (cola, *stream*...), no por una variable compartida. Cuando el productor termina, avisa de que no hay más.

Muestra solo la suma final.

**Salida esperada:**

```text
suma: 15
```

!!! note "En Python"
    Una `asyncio.Queue` entre dos corrutinas; `None` puede indicar «fin».

<details class="sol" data-key="av/a3/A3.6">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import asyncio
FIN = None
async def productor(cola):
    for i in range(1, 6):
        await cola.put(i)
# ... (resto de la solución bloqueado)</code></pre></div>
</details>
