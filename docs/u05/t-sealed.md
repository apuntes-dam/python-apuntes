# 5.D Jerarquías cerradas y clases de datos

## El problema: un conjunto de variantes conocido y cerrado

A veces los distintos tipos **no se van a ampliar**: un pago es en efectivo, con tarjeta o por Bizum, y no hay una cuarta posibilidad que inventar desde fuera. Con una jerarquía normal, quien use el tipo base nunca está seguro de haber cubierto todos los casos.

Una **jerarquía cerrada** (*sealed*) le dice al compilador **exactamente cuáles son los subtipos posibles**. Así, cuando se hace algo distinto según el subtipo, el compilador **comprueba que no falte ninguno**: si más adelante se añade un caso nuevo, **avisa en todos los sitios** donde falta tratarlo.

Python **no tiene `sealed`**. Se emula con **clases de datos** (`@dataclass`) y un **tipo unión** (`Pago = Efectivo | Tarjeta | Bizum`), y se consulta con **`match`**, que reconoce cada clase y extrae sus campos. Python **no comprueba** en ejecución que se hayan cubierto todos los casos; esa comprobación la hacen verificadores de tipos como `mypy` o `pyright`.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Efectivo:
    importe: int


@dataclass(frozen=True)
class Tarjeta:
    importe: int
    ultimos4: str


@dataclass(frozen=True)
class Bizum:
    importe: int
    telefono: str


Pago = Efectivo | Tarjeta | Bizum


def describir(pago: Pago) -> str:
    match pago:
        case Efectivo(importe=importe):
            return f"Efectivo de {importe} €"
        case Tarjeta(importe=importe, ultimos4=ultimos4):
            return f"Tarjeta ****{ultimos4} de {importe} €"
        case Bizum(importe=importe, telefono=telefono):
            return f"Bizum al {telefono} de {importe} €"


def comision(pago: Pago) -> int:
    match pago:
        case Efectivo():
            return 0
        case Tarjeta(importe=importe):
            return importe * 2 // 100
        case Bizum():
            return 1


if __name__ == "__main__":
    pagos = [Efectivo(50), Tarjeta(100, "1234"), Bizum(30, "600123456")]
    total = 0
    for pago in pagos:
        c = comision(pago)
        total += c
        print(f"{describir(pago)} -> comisión {c} €")
    print(f"total de comisiones: {total} €")
```

Salida:

```text
Efectivo de 50 € -> comisión 0 €
Tarjeta ****1234 de 100 € -> comisión 2 €
Bizum al 600123456 de 30 € -> comisión 1 €
total de comisiones: 3 €
```

Observa:

* Cada variante lleva **datos distintos** (`ultimos4` solo existe en la tarjeta, `telefono` solo en Bizum). Por eso aquí no vale un enumerado.
* Las variantes son **clases de datos**, que solo guardan información y comparan por valor (ver [4.B](../u04/t-encapsulamiento.md)).
* Las funciones `describir` y `comision` **no tienen caso por defecto**: no hace falta, porque están cubiertos todos los posibles.
* El comportamiento **está fuera** de las clases (en las funciones), no dentro de ellas. Se elige así cuando las variantes son solo datos y las operaciones se van añadiendo.

## ¿Enumerado, jerarquía cerrada, abstracta o interfaz?

| Situación | Usa |
|---|---|
| Valores fijos y sencillos, a lo sumo con datos iguales para todos (días, tamaños, estados) | **Enumerado** |
| Variantes cerradas, **cada una con datos distintos** (pagos, resultados, formas de un mensaje) | **Jerarquía cerrada** |
| Familia abierta con **código común** | Clase abstracta |
| Capacidad que clases distintas pueden ofrecer | Interfaz |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Poner un caso `default`/`else` que esconde los casos nuevos | No lo pongas: así el compilador te avisa si falta uno |
| Usar una jerarquía cerrada cuando otras personas deben poder añadir variantes | Interfaz o clase abstracta (jerarquía abierta) |
| Meter la lógica de cada caso en una cadena de `if` y `instanceof` | Un `switch`/`when`/`match` con patrones |
| Usar un enumerado cuando cada variante necesita datos propios | Jerarquía cerrada |

## Para practicar

El ejercicio de la biblioteca de [U5.1](herencia.md) usa clases de datos y una jerarquía cerrada para los usuarios. La comparación entre lenguajes de todos estos tipos de clase está en [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
