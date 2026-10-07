# 6.A Diseñar una jerarquía de clases

En la [unidad 5](../u05/index.md) viste **cómo se escribe** la herencia, las clases abstractas y las interfaces. Aquí se trata de **decidir bien**: qué clases hacen falta, qué va en la base y qué en cada hija, y cómo se organizan los constructores para que el diseño aguante cambios.

## Un ejemplo completo: un hotel

```python
from abc import ABC, abstractmethod


class Habitacion(ABC):
    def __init__(self, numero):
        self.numero = numero
        self.libre = True

    @abstractmethod
    def tipo(self):
        ...

    @abstractmethod
    def precio(self):
        ...

    def describir(self):
        ocupada = "" if self.libre else " (ocupada)"
        return f"{self.numero} {self.tipo()}: {self.precio()} €/noche{ocupada}"


class Individual(Habitacion):
    def tipo(self):
        return "Individual"

    def precio(self):
        return 50


class Doble(Habitacion):
    def __init__(self, numero, desayuno=False):
        super().__init__(numero)
        self.desayuno = desayuno

    def tipo(self):
        return "Doble con desayuno" if self.desayuno else "Doble"

    def precio(self):
        return 90 if self.desayuno else 80


class Suite(Habitacion):
    def __init__(self, numero, extras=0):
        super().__init__(numero)
        self.extras = extras

    def tipo(self):
        return "Suite"

    def precio(self):
        return 150 + 20 * self.extras


class Hotel:
    def __init__(self):
        self._habitaciones = []

    @property
    def todas(self):
        return tuple(self._habitaciones)

    def agregar(self, habitacion):
        self._habitaciones.append(habitacion)

    def reservar(self, numero):
        encontrada = next((h for h in self._habitaciones if h.numero == numero), None)
        if encontrada is None:
            raise ValueError(f"la habitación {numero} no existe")
        if not encontrada.libre:
            raise RuntimeError(f"la habitación {numero} ya está ocupada")
        encontrada.libre = False

    def libres(self):
        return [h for h in self._habitaciones if h.libre]

    def precio_medio(self):
        suma = 0
        for h in self._habitaciones:
            suma += h.precio()
        return suma // len(self._habitaciones)


if __name__ == "__main__":
    hotel = Hotel()
    hotel.agregar(Individual(101))
    hotel.agregar(Doble(201, desayuno=True))
    hotel.agregar(Suite(301, extras=2))
    for h in hotel.todas:
        print(h.describir())

    hotel.reservar(201)
    print(f"tras reservar la 201: {hotel.todas[1].describir()}")
    try:
        hotel.reservar(201)
    except RuntimeError as e:
        print(f"Error: {e}")
    try:
        hotel.reservar(999)
    except ValueError as e:
        print(f"Error: {e}")
    print("libres: " + ", ".join(str(h.numero) for h in hotel.libres()))
    print(f"precio medio: {hotel.precio_medio()} €")
```

Salida:

```text
101 Individual: 50 €/noche
201 Doble con desayuno: 90 €/noche
301 Suite: 190 €/noche
tras reservar la 201: 201 Doble con desayuno: 90 €/noche (ocupada)
Error: la habitación 201 ya está ocupada
Error: la habitación 999 no existe
libres: 101, 301
precio medio: 110 €
```

Cómo se ha decidido el diseño:

* **La base lleva lo que es común a todas.** `Habitacion` tiene el `numero` y si está `libre`, que valen para cualquier habitación, y un método `describir()` ya escrito.
* **Cada hija define solo lo que cambia.** El `tipo()` y el `precio()` son **abstractos**: cada habitación sabe el suyo. `Doble` y `Suite` añaden sus propios datos (`desayuno`, `extras`).
* **`describir()` usa los métodos abstractos.** Es el patrón del [método plantilla](../u05/t-abstractas.md): la base fija el esquema, las hijas rellenan los huecos.
* **El hotel solo conoce `Habitacion`.** Si se añade un tipo nuevo, `Hotel` no cambia: es polimorfismo y composición («un hotel **tiene** habitaciones»).
* **Las reglas, dentro de su sitio.** Reservar comprueba que la habitación exista y esté libre, y lanza una excepción **antes** de modificar nada. Además, `todas` devuelve una **tupla**, que no se puede modificar, en vez de la lista interna.
* **Los constructores reparten el trabajo.** La base inicializa el número; cada hija pasa ese dato hacia arriba y se queda con el suyo.

## Modificadores que ayudan a diseñar

| Necesito... | En Python |
|---|---|
| Clase que no se puede instanciar | heredar de `ABC` con `@abstractmethod` |
| Clase de la que no se puede heredar | no hay mecanismo (solo convención) |
| Solo contrato | `ABC` sin atributos, o `typing.Protocol` |
| Jerarquía cerrada | no hay; se emula con un tipo unión |
| Visible solo «por convención» | nombre con guion bajo `_numero` |
| Valor que no cambia tras crear el objeto | `@dataclass(frozen=True)` o una propiedad sin `setter` |

## Preguntas antes de crear una jerarquía

| Pregunta | Si la respuesta es... |
|---|---|
| ¿La relación es «es un»? | No → composición o interfaz, no herencia |
| ¿Hay código o datos comunes? | Sí → clase base (abstracta si no tiene sentido por sí sola) |
| ¿Solo hay que fijar un contrato? | Sí → interfaz |
| ¿Lo que cambia es un cálculo o una regla? | Valorar pasar una **función** (ver [4.E](../u04/t-modelar.md)) |
| ¿Las variantes son un conjunto cerrado con datos distintos? | Jerarquía cerrada (ver [5.D](../u05/t-sealed.md)) |
| ¿Se podrá usar cualquier hija donde se pida la base sin sorpresas? | Si no → hay que rediseñar (ver [6.C](t-solid-2.md), principio L) |

## Buenas prácticas

* **Jerarquías cortas y anchas** (una base y varias hijas) mejor que largas y estrechas.
* **La base, estable.** Cuanto más se cambia, más hijas se rompen. Si una base cambia a menudo, probablemente tiene demasiadas responsabilidades.
* **Cierra lo que no necesites abrir** (`final`, `sealed`, el comportamiento por defecto de Kotlin): permitir heredar es una decisión, no un valor por defecto.
* **No expongas tus colecciones internas**: ofrece métodos o una vista de solo lectura.
* **Prefiere la composición** cuando dudes entre heredar o tener un atributo.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Una clase base con atributos que a algunas hijas no les sirven | Mover esos atributos a las hijas que los usan |
| Hijas que sobrescriben un método solo para «no hacer nada» o lanzar un error | Señal de que la jerarquía está mal (ver 6.C) |
| Repetir el mismo código en varias hijas | Subirlo a la base |
| Una base con métodos que solo usa una hija | Bajarlos a esa hija |

## Para practicar

Haz los ejercicios de [U6.1 · Jerarquía de clases](jerarquia.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
