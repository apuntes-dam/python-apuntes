# 9.C Buenas prácticas: pool, seguridad, transacciones y DAO

## Pool de conexiones

Abrir una conexión con un servidor de bases de datos es **lento y costoso** (red, autenticación). Un programa que abre y cierra una conexión por cada operación desperdicia mucho tiempo. Un **pool de conexiones** mantiene varias conexiones **ya abiertas** y las **presta** a quien las necesita:

```text
programa ──pide conexión──▶  [ pool: 🔌 🔌 🔌 🔌 ]  ──▶  servidor de base de datos
         ◀──la devuelve───
```

En Python los pools los aportan las librerías de cada motor o **SQLAlchemy**, que gestiona un pool de conexiones por defecto. Con SQLite, que es un archivo local, basta con una conexión por hilo o compartida.

## Seguridad: la inyección SQL

La **inyección SQL** ocurre cuando se construye una consulta **pegando texto del usuario** dentro del SQL. Si el usuario escribe SQL en lugar de un dato normal, **cambia el significado de la consulta**: puede ver datos que no debe, saltarse un inicio de sesión o borrar tablas. Es una de las vulnerabilidades más graves y frecuentes de las aplicaciones reales.

```python
import sqlite3


def preparar(con):
    con.execute("CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, stock INTEGER NOT NULL)")
    con.execute("""INSERT INTO libros (titulo, stock) VALUES
        ('Don Quijote', 3), ('Novelas ejemplares', 2), ('Cien años de soledad', 5), ('El coronel no tiene quien le escriba', 1)""")
    con.commit()


def buscar_inseguro(con, titulo):
    sql = "SELECT titulo FROM libros WHERE titulo = '" + titulo + "'"  # ¡NUNCA así!
    return con.execute(sql).fetchall()


def buscar_seguro(con, titulo):
    return con.execute("SELECT titulo FROM libros WHERE titulo = ?", (titulo,)).fetchall()


def main():
    con = sqlite3.connect(":memory:")
    preparar(con)
    maliciosa = "x' OR '1'='1"
    print(f"[inseguro] Don Quijote -> {len(buscar_inseguro(con, 'Don Quijote'))} resultado(s)")
    print(f"[inseguro] {maliciosa} -> {len(buscar_inseguro(con, maliciosa))} resultado(s)")
    print(f"[seguro] Don Quijote -> {len(buscar_seguro(con, 'Don Quijote'))} resultado(s)")
    print(f"[seguro] {maliciosa} -> {len(buscar_seguro(con, maliciosa))} resultado(s)")
    con.close()


if __name__ == "__main__":
    main()
```

Salida:

```text
[inseguro] Don Quijote -> 1 resultado(s)
[inseguro] x' OR '1'='1 -> 4 resultado(s)
[seguro] Don Quijote -> 1 resultado(s)
[seguro] x' OR '1'='1 -> 0 resultado(s)
```

La cadena `x' OR '1'='1` hace que la consulta insegura quede como:

```sql
SELECT titulo FROM libros WHERE titulo = 'x' OR '1'='1'
```

Como `'1'='1'` siempre es cierto, la consulta **devuelve todos los libros**. Con una **consulta parametrizada**, esa misma cadena se trata como un **título que no existe**, porque el valor viaja **aparte del SQL** y nunca se interpreta como código.

`con.execute('... WHERE titulo = ?', (titulo,))`: los valores van en una **tupla aparte** (¡con la coma, si solo hay uno!).

!!! danger "Regla de oro"
    **Nunca** construyas SQL concatenando o interpolando datos que no controles (todo lo que escribe el usuario, lo que llega de un formulario o de la red). Usa **siempre** parámetros. Un parámetro sirve para **valores**; los nombres de tablas o columnas no se pueden parametrizar: si deben variar, elígelos de una lista fija de opciones permitidas.

## Transacciones

Una **transacción** agrupa varias operaciones para que se ejecuten **como una sola**: o se hacen **todas** o no se hace **ninguna**. Es imprescindible cuando un cambio requiere varios pasos. Prestar un libro, por ejemplo, exige **registrar el préstamo** y **descontar el stock**: si lo primero se hace y lo segundo falla, la base de datos quedaría incoherente.

```python
import sqlite3


class PrestamoFallido(Exception):
    pass


def preparar(con):
    con.execute("CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, stock INTEGER NOT NULL)")
    con.execute("CREATE TABLE prestamos (id INTEGER PRIMARY KEY AUTOINCREMENT, id_libro INTEGER NOT NULL, socio TEXT NOT NULL)")
    con.execute("""INSERT INTO libros (titulo, stock) VALUES
        ('Don Quijote', 3), ('Novelas ejemplares', 2), ('Cien años de soledad', 5), ('El coronel no tiene quien le escriba', 1)""")
    con.commit()


def prestar(con, id_libro, socio, fallar_a_mitad=False):
    """Registra el préstamo y descuenta el stock como UNA sola operación."""
    try:
        stock = con.execute("SELECT stock FROM libros WHERE id = ?", (id_libro,)).fetchone()[0]
        if stock < 1:
            raise PrestamoFallido("no hay stock")
        con.execute("INSERT INTO prestamos (id_libro, socio) VALUES (?, ?)", (id_libro, socio))
        if fallar_a_mitad:
            raise PrestamoFallido("error simulado a mitad de la operación")
        con.execute("UPDATE libros SET stock = stock - 1 WHERE id = ?", (id_libro,))
        con.commit()  # todo ha ido bien: se confirma
        return "correcto"
    except PrestamoFallido as e:
        con.rollback()  # algo ha fallado: se deshace TODO
        return str(e)


def entero(con, sql, parametros=()):
    return con.execute(sql, parametros).fetchone()[0]


def main():
    con = sqlite3.connect(":memory:")
    preparar(con)
    print(f"préstamo 1: {prestar(con, 4, 'Ana')}")
    print(f"préstamo 2: {prestar(con, 4, 'Luis')}")
    print(f"préstamo 3: {prestar(con, 3, 'Eva', fallar_a_mitad=True)}")
    print(f"préstamos registrados: {entero(con, 'SELECT COUNT(*) FROM prestamos')}")
    print(f"stock del libro 4: {entero(con, 'SELECT stock FROM libros WHERE id = ?', (4,))}")
    print(f"stock del libro 3: {entero(con, 'SELECT stock FROM libros WHERE id = ?', (3,))}")
    con.close()


if __name__ == "__main__":
    main()
```

Salida:

```text
préstamo 1: correcto
préstamo 2: no hay stock
préstamo 3: error simulado a mitad de la operación
préstamos registrados: 1
stock del libro 4: 0
stock del libro 3: 5
```

El tercer préstamo falla **a propósito, después de haber insertado el préstamo**. Gracias a la transacción, el `ROLLBACK` **deshace también esa inserción**: al final solo hay **un** préstamo registrado y el stock del libro 3 sigue en 5. Sin transacción, habría un préstamo «fantasma» sin descontar.

El módulo `sqlite3` abre una transacción **automáticamente** antes de un `INSERT`, `UPDATE` o `DELETE`. Se termina con `con.commit()` (confirmar) o `con.rollback()` (deshacer).

Una transacción cumple las propiedades **ACID**:

| Propiedad | Significa |
|---|---|
| **A**tomicidad | Todo o nada |
| **C**onsistencia | Los datos siguen cumpliendo las reglas (claves, restricciones) |
| **I**slamiento | Varias operaciones simultáneas no se estorban |
| **D**urabilidad | Lo confirmado queda guardado aunque falle el equipo |

## El patrón DAO

Si el SQL está **repartido por todo el programa**, cualquier cambio en las tablas obliga a buscarlo en cien sitios, y es imposible probar la lógica sin una base de datos. El patrón **DAO** (*Data Access Object*) lo soluciona: **una clase** concentra **todo** el acceso a los datos de una tabla, y el resto del programa solo habla con ella y con **objetos**, nunca con conexiones ni con SQL.

```text
  Programa  ──▶  Servicio (reglas)  ──▶  DAO (SQL)  ──▶  Base de datos
  objetos Libro        usa objetos            convierte filas ↔ objetos
```

```python
import sqlite3
from dataclasses import dataclass


@dataclass(frozen=True)
class Libro:
    id: int | None
    titulo: str
    anio: int
    stock: int
    id_autor: int


class LibroDao:
    """Único sitio del programa que conoce el SQL y la conexión."""

    def __init__(self, con):
        self._con = con

    def guardar(self, libro):
        cursor = self._con.execute(
            "INSERT INTO libros (titulo, anio, stock, id_autor) VALUES (?, ?, ?, ?)",
            (libro.titulo, libro.anio, libro.stock, libro.id_autor))
        self._con.commit()
        return Libro(cursor.lastrowid, libro.titulo, libro.anio, libro.stock, libro.id_autor)

    def buscar(self, id):
        fila = self._con.execute("SELECT id, titulo, anio, stock, id_autor FROM libros WHERE id = ?", (id,)).fetchone()
        return None if fila is None else Libro(*fila)

    def todos(self):
        return [Libro(*f) for f in self._con.execute("SELECT id, titulo, anio, stock, id_autor FROM libros ORDER BY id")]

    def por_autor(self, nombre):
        consulta = """SELECT l.id, l.titulo, l.anio, l.stock, l.id_autor FROM libros l
                      JOIN autores a ON a.id = l.id_autor WHERE a.nombre = ? ORDER BY l.anio"""
        return [Libro(*f) for f in self._con.execute(consulta, (nombre,))]

    def cambiar_stock(self, id, diferencia):
        self._con.execute("UPDATE libros SET stock = stock + ? WHERE id = ?", (diferencia, id))
        self._con.commit()


class Catalogo:
    """Servicio: usa el DAO y no contiene SQL."""

    def __init__(self, dao):
        self._dao = dao

    def prestar_uno(self, id):
        libro = self._dao.buscar(id)
        if libro is None:
            raise ValueError("el libro no existe")
        if libro.stock < 1:
            raise ValueError("no hay stock")
        self._dao.cambiar_stock(id, -1)


def texto(libro):
    return f"Libro(id={libro.id}, titulo={libro.titulo}, anio={libro.anio}, stock={libro.stock})"


def preparar(con):
    con.execute("CREATE TABLE autores (id INTEGER PRIMARY KEY AUTOINCREMENT, nombre TEXT NOT NULL)")
    con.execute("""CREATE TABLE libros (
        id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, anio INTEGER, stock INTEGER NOT NULL,
        id_autor INTEGER NOT NULL REFERENCES autores(id))""")
    con.execute("INSERT INTO autores (nombre) VALUES ('Cervantes'), ('García Márquez'), ('Frank Herbert')")
    con.execute("""INSERT INTO libros (titulo, anio, stock, id_autor) VALUES
        ('Don Quijote', 1605, 3, 1), ('Novelas ejemplares', 1613, 2, 1),
        ('Cien años de soledad', 1967, 5, 2), ('El coronel no tiene quien le escriba', 1961, 1, 2)""")
    con.commit()


def main():
    con = sqlite3.connect(":memory:")
    preparar(con)
    dao = LibroDao(con)
    catalogo = Catalogo(dao)

    nuevo = dao.guardar(Libro(None, "Dune", 1965, 2, 3))
    print(f"guardado: {texto(nuevo)}")
    print(f"buscar(5): {dao.buscar(5).titulo}")
    print(f"buscar(99): {'no existe' if dao.buscar(99) is None else 'existe'}")
    print("del autor Cervantes: " + ", ".join(l.titulo for l in dao.por_autor("Cervantes")))
    catalogo.prestar_uno(5)
    print(f"stock de Dune tras prestar uno: {dao.buscar(5).stock}")
    print(f"total de libros: {len(dao.todos())}")
    con.close()


if __name__ == "__main__":
    main()
```

Salida:

```text
guardado: Libro(id=5, titulo=Dune, anio=1965, stock=2)
buscar(5): Dune
buscar(99): no existe
del autor Cervantes: Don Quijote, Novelas ejemplares
stock de Dune tras prestar uno: 1
total de libros: 5
```

Fíjate en el reparto de responsabilidades:

* **`Libro`** es un objeto de datos, sin SQL.
* **`LibroDao`** es el **único** sitio con SQL y con la conexión. Convierte filas en objetos `Libro` y al revés.
* **`Catalogo`** es el servicio: contiene las **reglas** («no se puede prestar sin stock») y usa el DAO, pero **no contiene SQL**.
* Si mañana cambia el motor de base de datos, solo se modifica el DAO.

!!! tip "Relación con SOLID"
    El DAO aplica la **responsabilidad única** (cada clase tiene una sola razón para cambiar) y, si el servicio recibe el DAO por el constructor, la **inversión de dependencias** (ver [6.B](../u06/t-solid-1.md) y [6.C](../u06/t-solid-2.md)): el servicio se podría probar con un DAO falso, sin base de datos.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Concatenar texto del usuario en el SQL | Parámetros `?`, siempre |
| Varias operaciones relacionadas sin transacción | Agruparlas con `BEGIN`/`COMMIT` y deshacer ante cualquier error |
| Olvidar el `ROLLBACK` cuando algo falla | Capturar el error y deshacer siempre |
| SQL esparcido por todo el programa | Una capa DAO |
| Abrir una conexión por cada operación contra un servidor | Un pool, o una conexión compartida si es SQLite |

## Para practicar

Haz los ejercicios de [U9.3 · Pool, seguridad, transacciones y DAO](buenas-practicas.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
