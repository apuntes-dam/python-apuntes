# U4 · Programación orientada a objetos

Pasar del código suelto a **clases y objetos**: atributos, métodos, constructores, validación, encapsulamiento y colecciones de objetos. Termina con un proyecto personal o en grupo.

## Antes de empezar: qué debes dominar

* Clase, objeto, atributo y método.
* Constructores (principal y alternativos) y valores por defecto.
* Encapsulamiento: visibilidad y propiedades de solo lectura.
* Enumerados, sobrecarga y `toString`/igualdad.
* Colecciones de objetos y validación con excepciones.

## Ejercicios de la unidad

| Bloque | Ejercicios |
|---|---|
| [U4.1 · Repaso de las unidades 1 a 3](repaso.md) | 1 |
| [U4.2 · POO I (ejercicios 1 al 5)](poo-1.md) | 5 |
| [U4.3 · POO II (ejercicios 6 al 10)](poo-2.md) | 5 |
| [U4.4 · Robots (parte 1)](robots-1.md) | 2 |
| [U4.5 · Robots (parte 2 y reto)](robots-2.md) | 1 |
| [U4.6 · Prueba: Cafetera y Taza](prueba.md) | 2 |
| [U4.7 · Juego del ahorcado (grupos)](proyecto.md) | 1 |
| [U4.8 · Cambio de rol: explícamelo tú (grupos)](cambio-de-rol.md) | 2 |

## Equivalencias de POO en Python

| Idea | En Python |
|---|---|
| Constructor | `def __init__(self, ...)` (solo uno; varias formas con valores por defecto o `@classmethod`) |
| Propiedad calculada de solo lectura | `@property def imc(self): ...` |
| Privado | por convención, nombre que empieza por `_` |
| Validar al crear | `raise ValueError(...)` en `__init__` |
| Herencia | `class B(A):`, con `super().__init__(...)` |
| Clase abstracta | `from abc import ABC, abstractmethod` |
| Enumerado | `from enum import Enum` |
| Datos con igualdad por valor | `@dataclass` (módulo `dataclasses`) |
| Parámetro por defecto | `def agregar(self, cantidad=200)` |
| Representación en texto | `__str__` |
| Igualdad | `__eq__` |

## Antes de pasar a los ejercicios

Cuando hayas leído y practicado la teoría, marca la casilla para **desbloquear** los ejercicios de esta unidad:

<div class="ej-check" data-unit="u04"></div>
