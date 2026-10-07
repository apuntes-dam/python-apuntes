# 4.A Clases y objetos

Hasta ahora los datos (variables) y las operaciones (funciones) iban por separado. En la **programación orientada a objetos (POO)** se agrupan en una sola pieza: el **objeto**, que guarda sus propios datos y sabe hacer sus propias operaciones.

| Concepto | Qué es | Ejemplo |
|---|---|---|
| **Clase** | La **plantilla** que describe cómo son y qué hacen los objetos de un tipo | `Libro` |
| **Objeto** (o *instancia*) | Una **cosa concreta** creada a partir de la clase | «El Quijote», «Dune» |
| **Atributo** | Un dato que guarda cada objeto (su **estado**) | `titulo`, `paginas`, `leidas` |
| **Método** | Una operación que sabe hacer el objeto (su **comportamiento**) | `leer(...)`, `progreso()` |
| **Constructor** | Código que se ejecuta al **crear** el objeto y lo deja listo | `Libro('Dune', 400)` |

Todos los libros comparten la misma clase, pero **cada objeto tiene su propio estado**: leer páginas de un libro no cambia el otro.

## Tu primera clase

```python
class Libro:
    def __init__(self, titulo, paginas):
        self.titulo = titulo
        self.paginas = paginas
        self.leidas = 0

    def leer(self, cantidad):
        self.leidas += cantidad

    def progreso(self):
        return self.leidas * 100 // self.paginas

    def __str__(self):
        return f"Libro({self.titulo}, {self.paginas} págs, leídas {self.leidas})"


if __name__ == "__main__":
    quijote = Libro("El Quijote", 1000)
    dune = Libro("Dune", 400)
    dune.leer(100)
    print(quijote)
    print(dune)
    print(f"progreso de Dune: {dune.progreso()}%")

    otro = dune  # no es una copia: es el mismo objeto
    otro.leer(50)
    print(f"tras leer 50 más: {dune}")
```

Salida:

```text
Libro(El Quijote, 1000 págs, leídas 0)
Libro(Dune, 400 págs, leídas 100)
progreso de Dune: 25%
tras leer 50 más: Libro(Dune, 400 págs, leídas 150)
```

Observa cuatro cosas:

1. La clase `Libro` se escribe **una vez** y se usa para crear tantos objetos como se quiera (`quijote`, `dune`).
2. Los métodos (`leer`, `progreso`) trabajan con los atributos **del objeto sobre el que se llaman**.
3. `toString` (o su equivalente) define **cómo se convierte el objeto en texto**: es lo que se muestra al imprimirlo.
4. Al final, `otro = dune` **no crea un libro nuevo**: ahora `otro` y `dune` son dos nombres para el **mismo objeto**, y leer con uno cambia lo que se ve con el otro. Los objetos se manejan **por referencia**, igual que las listas del [apartado 3.1](../u03/01-listas.md).

## Cómo se escribe en Python

| Necesito... | En Python |
|---|---|
| Declarar la clase | `class Libro:` (el cuerpo va indentado) |
| Atributo | `self.titulo = titulo` dentro de `__init__` |
| Constructor | El método especial `__init__(self, ...)` |
| Crear un objeto | `Libro("Dune", 400)` (sin `new`) |
| Llamar a un método | `dune.leer(100)` |
| El propio objeto | `self`: es **obligatorio** como primer parámetro de cada método y al usar los atributos |
| Texto del objeto | `def __str__(self): return "..."` |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Confundir la **clase** con el **objeto** | La clase es el molde; cada objeto es una pieza hecha con él |
| Creer que asignar un objeto a otra variable lo copia | Es la misma referencia; para una copia hay que crearla explícitamente |
| Escribir toda la lógica en `main` y usar la clase solo para guardar datos | Pon cada operación en el método de la clase a la que corresponde |
| Un atributo que nunca cambia pero no se marca como constante | Márcalo (`final`, `val`) para que el compilador lo vigile |

Olvidar `self.` delante de un atributo crea una variable local que desaparece al terminar el método, y olvidar `self` como primer parámetro provoca un `TypeError`.

## Para practicar

Las primeras clases se practican en [U4.2 · POO I](poo-1.md). La [tabla de equivalencias de POO](index.md#equivalencias-de-poo-en-python) del índice de la unidad resume la sintaxis de Python.
