# A1.B Colecciones sin bucles

Casi todo lo que haces con una lista con un `for` entra en unas pocas operaciones: **quedarse con** unos elementos, **transformarlos**, **reducirlos** a un valor, **ordenarlos**, **agruparlos** y **preguntar** si alguno o todos cumplen algo. Si las pasas como funciones, el programa dice **qué** quieres, no **cómo** recorrerlo.

## Las operaciones en Python

| Qué quieres | Cómo se hace |
|---|---|
| Quedarse con los que cumplen | `[x for x in lista if ...]` o `filter(f, lista)` |
| Transformar cada elemento | `[f(x) for x in lista]` o `map(f, lista)` |
| Reducir a un solo valor | `sum(...)`, `max(...)`, o `functools.reduce(f, lista)` |
| Ordenar con un criterio | `sorted(lista, key=...)` (lista nueva; `lista.sort()` modifica) |
| Agrupar o contar por clave | `collections.Counter`, `defaultdict`, `itertools.groupby` (pide la lista ya ordenada) |
| ¿Alguno / todos? | `any(...)` / `all(...)` |

## Un ejemplo completo

Una pastelería guarda sus pedidos (producto, cantidad y precio por unidad). Con las operaciones anteriores se responde a todo sin un solo bucle explícito:

```python
# Pedidos de una pastelería: filtrar, transformar, reducir, ordenar y agrupar sin escribir bucles a mano.
from collections import Counter
from dataclasses import dataclass


@dataclass(frozen=True)
class Pedido:
    producto: str
    cantidad: int
    precio: int

    @property
    def importe(self):
        return self.cantidad * self.precio


pedidos = [
    Pedido("tarta", 2, 18),
    Pedido("galleta", 12, 1),
    Pedido("tarta", 1, 18),
    Pedido("pan", 3, 2),
    Pedido("galleta", 7, 1),
]

# filter + map: en Python lo habitual es una comprensión de listas
grandes = [p.producto for p in pedidos if p.cantidad >= 3]
print("grandes:", ", ".join(grandes))

# map: de pedidos a importes (también existe map(función, colección))
importes = [p.importe for p in pedidos]
print("importes:", importes)

# sum / reduce: de muchos valores a uno
print("total:", sum(importes))

# sorted con un criterio (key); devuelve una lista nueva
ordenados = sorted(pedidos, key=lambda p: p.importe, reverse=True)
print("mayor a menor:", ", ".join(f"{p.producto} {p.importe}" for p in ordenados))

# agrupar: Counter suma por clave (no hay un groupBy listo para esto)
unidades = Counter()
for p in pedidos:
    unidades[p.producto] += p.cantidad
print("unidades:", ", ".join(f"{k}={unidades[k]}" for k in sorted(unidades)))

# any / all: preguntas sobre toda la colección
print("¿alguno vale más de 30?", "sí" if any(p.importe > 30 for p in pedidos) else "no")
print("¿todos tienen cantidad > 0?", "sí" if all(p.cantidad > 0 for p in pedidos) else "no")
```

Salida:

```text
grandes: galleta, pan, galleta
importes: [36, 12, 18, 6, 7]
total: 79
mayor a menor: tarta 36, tarta 18, galleta 12, galleta 7, pan 6
unidades: galleta=19, pan=3, tarta=3
¿alguno vale más de 30? sí
¿todos tienen cantidad > 0? sí
```

Cada línea de la salida es una de las operaciones de la tabla. Compruébalas a mano: los importes son `2·18`, `12·1`, `1·18`, `3·2` y `7·1`; el total es `79`; y hay `19` galletas porque `12 + 7`.

## Lo que conviene saber

Una comprensión crea una lista nueva y es lo más habitual y legible en Python; `map` y `filter` devuelven un iterador **perezoso**, que hay que convertir con `list(...)` para ver su contenido. `sorted` devuelve una lista nueva, mientras que `lista.sort()` cambia la original.

!!! warning "Un orden estable"
    Si dos elementos empatan en el criterio de orden, no des por hecho en qué orden saldrán. Los datos del ejemplo no tienen empates a propósito. Si te hacen falta, añade un segundo criterio.

!!! tip "¿Bucle o función?"
    No es mejor siempre una cosa que otra. Un bucle sigue siendo lo más claro cuando el cuerpo hace varias cosas distintas o hay que parar a mitad. Usa las operaciones de colección cuando el trabajo es **una transformación de datos** y verás que el código se lee casi como la descripción del problema.

## Para practicar

Los ejercicios [A1.3 y A1.4](ejercicios.md) usan estas operaciones. Para ver la misma tabla en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
