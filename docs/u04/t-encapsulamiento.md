# 4.B Encapsulamiento

**Encapsular** es **ocultar el estado interno** de un objeto y dejar solo unas operaciones controladas para usarlo. Así el objeto se encarga de que sus datos siempre tengan sentido: un precio nunca es negativo, un stock nunca baja de cero.

Sin encapsulamiento, cualquiera podría escribir `producto.precio = -5` y el error aparecería mucho más tarde, lejos de donde se produjo. Con él, el fallo salta **justo al intentar el cambio incorrecto**.

## Visibilidad

Python **no tiene privacidad real**: por convención un nombre que empieza por **guion bajo** (`_precio`) significa «no lo toques desde fuera». Con dos guiones (`__precio`) el nombre se transforma para dificultar el acceso, pero tampoco es una barrera. Se confía en que el programador respete la convención.

## Acceso controlado y validación

```python
class Producto:
    def __init__(self, nombre, precio, stock):
        self._comprobar_precio(precio)
        self.nombre = nombre
        self._precio = precio
        self._stock = stock

    @property
    def precio(self):
        return self._precio

    @precio.setter
    def precio(self, nuevo):
        self._comprobar_precio(nuevo)
        self._precio = nuevo

    @property
    def stock(self):
        return self._stock

    @property
    def valor_stock(self):
        return self._precio * self._stock

    def vender(self, cantidad):
        if cantidad > self._stock:
            raise RuntimeError("no hay stock suficiente")
        self._stock -= cantidad

    @staticmethod
    def _comprobar_precio(precio):
        if precio < 0:
            raise ValueError("el precio no puede ser negativo")


if __name__ == "__main__":
    p = Producto("Cuaderno", 3, 10)
    print(f"{p.nombre}: {p.precio} € x {p.stock} = {p.valor_stock} €")
    p.vender(4)
    print(f"tras vender 4, quedan {p.stock}")
    try:
        p.vender(20)
    except RuntimeError as e:
        print(f"Error: {e}")
    try:
        p.precio = -1
    except ValueError as e:
        print(f"Error: {e}")
    p.precio = 7
    print(f"nuevo valor del stock: {p.valor_stock} €")
    try:
        Producto("Roto", -2, 1)
    except ValueError as e:
        print(f"Error: {e}")
```

Salida:

```text
Cuaderno: 3 € x 10 = 30 €
tras vender 4, quedan 6
Error: no hay stock suficiente
Error: el precio no puede ser negativo
nuevo valor del stock: 42 €
Error: el precio no puede ser negativo
```

Qué hace cada pieza:

* **Atributos protegidos** (`precio`, `stock`): no se pueden cambiar desde fuera sin pasar por el código de la clase.
* **Validar al crear y al modificar**: el constructor y el cambio de precio comprueban el valor y **lanzan una excepción** si no es válido (ver [2.3 Excepciones](../u02/02-excepciones.md)).
* **Solo lectura**: el stock se puede consultar, pero **solo** `vender` lo modifica.
* **Propiedad calculada**: `valorStock` no se guarda; se calcula cada vez a partir de `precio` y `stock`, así nunca queda desactualizada.

Se usa el decorador **`@property`** para el *getter* y **`@nombre.setter`** para el *setter*: desde fuera, `p.precio` y `p.precio = 7` parecen un atributo normal pero ejecutan código. Sin `setter`, la propiedad es de solo lectura. Una propiedad calculada, como `valor_stock`, es un `@property` sin atributo detrás.

!!! tip "Valida antes de modificar"
    Fíjate en `vender`: primero comprueba que hay stock y **solo después** resta. Si la comprobación falla, el objeto queda como estaba. Modificar primero y validar después deja objetos a medias.

## Igualdad: identidad frente a valor

Dos objetos pueden ser **el mismo** (la misma referencia) o ser **iguales** (tener el mismo contenido). Por defecto, `==` solo reconoce lo primero; para que dos puntos con las mismas coordenadas sean iguales hay que decírselo.

```python
from dataclasses import dataclass


class PuntoSimple:
    def __init__(self, x, y):
        self.x = x
        self.y = y


@dataclass(frozen=True)
class Punto:
    x: int
    y: int


if __name__ == "__main__":
    print(f"sin igualdad por valor: {'sí' if PuntoSimple(1, 2) == PuntoSimple(1, 2) else 'no'}")
    print(f"con igualdad por valor: {'sí' if Punto(1, 2) == Punto(1, 2) else 'no'}")
    conjunto = {Punto(1, 2), Punto(1, 2), Punto(3, 4)}
    print(f"puntos distintos en el conjunto: {len(conjunto)}")
```

Salida:

```text
sin igualdad por valor: no
con igualdad por valor: sí
puntos distintos en el conjunto: 2
```

Un **`@dataclass`** genera `__init__`, `__eq__` y `__repr__` a partir de los campos anotados; con `frozen=True` además es inmutable y se puede meter en un conjunto o usar como clave. Sin él, `==` compara referencias.

Por la misma razón, en un **conjunto** o como **clave de un mapa**, la igualdad por valor decide si dos objetos son «el mismo elemento» (ver [3.3 Conjuntos](../u03/03-conjuntos.md)).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Atributos públicos y modificables desde cualquier sitio | Hazlos privados y ofrece métodos para lo que haga falta |
| Un *setter* que acepta cualquier valor | Valida y lanza una excepción si no es correcto |
| Ofrecer un *setter* para todo «por si acaso» | Solo lo que de verdad deba poder cambiar: lo demás, de solo lectura |
| Validar en el constructor pero no en el *setter* (o al revés) | El mismo control en **todas** las puertas de entrada |
| Sobrescribir `equals` sin `hashCode` (o al revés) | Siempre juntos y con los mismos campos |

## Para practicar

Los ejercicios de validación y atributos de solo lectura están en [U4.2 · POO I](poo-1.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
