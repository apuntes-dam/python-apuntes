# A1.A Funciones como valores

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5 (variables, estructuras de control, colecciones y clases). Si alguna idea se te resiste, vuelve a la unidad correspondiente.

Hasta ahora has **llamado** funciones. Aquí das un paso más: tratar una función **como un valor**. Se puede guardar en una variable, pasar a otra función y devolver desde una función. Es la base de cosas que ya usas sin darte cuenta: ordenar con un criterio, reaccionar a un clic o filtrar una lista.

## Tipo de una función en Python

En Python las funciones son objetos como cualquier otro: se guardan en variables, se pasan como argumento y se devuelven. No hace falta declarar su tipo. `lambda n: n * 2` crea una función anónima de una sola expresión; para algo más largo se define una función con `def` y se pasa por su nombre, sin paréntesis.

## Un ejemplo con todo

El programa define una función, la pasa a otra, devuelve una función desde una función y crea dos contadores independientes:

```python
def doble(n):
    return n * 2


# Recibe una función como parámetro
def aplicar_dos_veces(f, x):
    return f(f(x))


# Devuelve una función que "recuerda" factor
def multiplicador(factor):
    return lambda n: n * factor


# Cierre: la función interna recuerda y modifica 'cuenta' (nonlocal permite modificarla)
def crear_contador():
    cuenta = 0

    def siguiente():
        nonlocal cuenta
        cuenta += 1
        return cuenta

    return siguiente


cuadrado = lambda n: n * n  # función anónima guardada en una variable

print(f"doble(4) = {doble(4)}")
print(f"cuadrado(4) = {cuadrado(4)}")
print(f"aplicar dos veces doble a 3 = {aplicar_dos_veces(doble, 3)}")
print(f"multiplicador(5)(7) = {multiplicador(5)(7)}")

a = crear_contador()
b = crear_contador()
print(f"contador A: {a()} {a()} {a()}")
print(f"contador B: {b()}")
```

Salida:

```text
doble(4) = 8
cuadrado(4) = 16
aplicar dos veces doble a 3 = 12
multiplicador(5)(7) = 35
contador A: 1 2 3
contador B: 1
```

Qué ocurre en cada línea:

| Línea de salida | Qué demuestra |
|---|---|
| `doble(4)` y `cuadrado(4)` | Una función con nombre y otra **anónima guardada en una variable** se llaman igual |
| `aplicar dos veces doble a 3` | Una función recibe **otra función como parámetro** y la usa dos veces |
| `multiplicador(5)(7)` | Una función **devuelve otra función**, que recuerda el factor `5` |
| `contador A` y `contador B` | Cada contador **recuerda su propia cuenta**: no se pisan |

## Cierres: funciones que recuerdan

Una función definida dentro de otra es un **cierre**: recuerda las variables de la función exterior aunque esta ya haya terminado. Para **modificar** una de esas variables (no solo leerla) hay que declararla con `nonlocal`; sin esa palabra, `cuenta += 1` crearía una variable local nueva y daría error.

!!! tip "Cuándo usarlo"
    Si necesitas «una operación que se decide más tarde» (un criterio de orden, un filtro, qué hacer al terminar), pásala como función. Es más corto y más flexible que crear una clase solo para eso.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a la función al pasarla (`doble(3)` en vez de `doble`) | Pasa el nombre **sin paréntesis** si quieres pasar la función, no su resultado |
| Esperar que cada llamada comparta o no comparta estado sin comprobarlo | Crea el cierre una vez por contador, como en el ejemplo |
| Escribir lambdas largas e ilegibles | Si pasa de una o dos líneas, ponle nombre a la función |

## Para practicar

Los ejercicios [A1.1 y A1.2](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
