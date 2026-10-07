# 3.1 Listas y tuplas

Una **lista** guarda varios valores **ordenados** en una sola variable, y se accede a cada uno por su posición (empezando en 0). Puede crecer y encogerse.

En Python la lista es `list` y es **modificable**. Puede mezclar tipos, aunque lo normal es que todos sean del mismo.

## Operaciones básicas

```python
lista = [5, 3, 8, 1]
print(f"lista: {lista}")
print(f"tamaño: {len(lista)}, primero: {lista[0]}, último: {lista[-1]}")

lista.append(9)
print(f"tras añadir 9: {lista}")
lista.insert(1, 7)
print(f"insertar 7 en la posición 1: {lista}")
lista.remove(3)
print(f"quitar el valor 3: {lista}")
lista.pop(0)
print(f"quitar la posición 0: {lista}")
lista.sort()
print(f"ordenada: {lista}")

suma = 0
for n in lista:
    suma += n
print(f"suma: {suma}, mayor: {max(lista)}")
```

Salida:

```text
lista: [5, 3, 8, 1]
tamaño: 4, primero: 5, último: 1
tras añadir 9: [5, 3, 8, 1, 9]
insertar 7 en la posición 1: [5, 7, 3, 8, 1, 9]
quitar el valor 3: [5, 7, 8, 1, 9]
quitar la posición 0: [7, 8, 1, 9]
ordenada: [1, 7, 8, 9]
suma: 25, mayor: 9
```

| Operación | En Python |
|---|---|
| Añadir al final | `lista.append(9)` |
| Insertar en una posición | `lista.insert(1, 7)` |
| Quitar un **valor** | `lista.remove(3)` (error si no está) |
| Quitar una **posición** | `lista.pop(0)` · `del lista[0]` |
| Ordenar (modifica la lista) | `lista.sort()` · `sorted(lista)` devuelve una nueva |
| Posición de un valor | `lista.index(8)` (error si no está) |
| ¿Contiene? | `8 in lista` |
| Tamaño · primero · último | `len(lista)` · `lista[0]` · `lista[-1]` |
| Una parte | `lista[1:3]` |
| Vaciar | `lista.clear()` |

`remove(3)` quita el **valor** 3 (la primera vez que aparece) y `pop(3)` la **posición** 3. Si el valor no está, `remove` lanza `ValueError`: comprueba antes con `in`.

!!! warning "No cambies una lista mientras la recorres"
    Añadir o quitar elementos dentro del bucle que la recorre da errores o resultados raros. Si quieres filtrar, construye una **lista nueva** con lo que sirve, o usa el método específico de borrado por condición de tu lenguaje.

## Recorrer, copiar y listas anidadas

```python
numeros = [10, 20, 30]
partes = []
for i in range(len(numeros)):
    partes.append(f"{i}:{numeros[i]}")
print("con índice: " + " ".join(partes))

alias = numeros
alias[0] = 99
print(f"alias: {numeros}")

copia = numeros.copy()
copia[0] = 1
print(f"original: {numeros}, copia: {copia}")

matriz = [[1, 2, 3], [4, 5, 6]]
for fila in matriz:
    print(f"fila: {fila}")
print(f"elemento [1][2] = {matriz[1][2]}")
```

Salida:

```text
con índice: 0:10 1:20 2:30
alias: [99, 20, 30]
original: [99, 20, 30], copia: [1, 20, 30]
fila: [1, 2, 3]
fila: [4, 5, 6]
elemento [1][2] = 6
```

Hay dos ideas importantes en este ejemplo:

* **Una lista es una referencia.** `alias = numeros` no crea otra lista: las dos variables apuntan a la **misma**, y al cambiar una cambia la otra. Para tener una lista independiente hay que **copiarla**: `numeros.copy()` (o `numeros[:]`, o `list(numeros)`).
* **La copia es superficial.** Copia los elementos, pero si estos son a su vez listas (como en una matriz), las sublistas se **comparten** entre original y copia.

Una **lista de listas** sirve como tabla o matriz: `matriz[fila][columna]`. Se recorre con dos bucles, uno dentro de otro.

## Tuplas y transformar listas

```python
persona = ("Ana", 25)
print(f"{persona[0]} tiene {persona[1]} años")
nombre, edad = persona
print(f"desempaquetado: {nombre}, {edad}")

numeros = [1, 2, 3, 4, 5, 6]
cuadrados = [n * n for n in numeros if n % 2 == 0]
print(f"cuadrados de los pares: {cuadrados}")
print(f"suma de los cuadrados: {sum(cuadrados)}")
```

Salida:

```text
Ana tiene 25 años
desempaquetado: Ana, 25
cuadrados de los pares: [4, 16, 36]
suma de los cuadrados: 56
```

Una **tupla** es como una lista pero **inmutable**: `("Ana", 25)`. Se accede por posición y se **desempaqueta** con `nombre, edad = persona`. Es lo que devuelve una función cuando devuelve «varios valores».

En Python se usa una **comprensión de listas**: `[expresión for n in lista if condición]`, que filtra y transforma a la vez; para acumular están `sum`, `max`, `min`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Acceder a una posición que no existe | Comprueba el tamaño antes: la última posición es el tamaño menos 1 |
| Creer que `b = a` copia la lista | Copia explícitamente cuando necesites una independiente |
| Ordenar una lista y perder el orden original | Ordena una **copia**, o usa la versión que devuelve una lista nueva |
| Modificar la lista mientras se recorre | Construye una lista nueva con el resultado |

## Para practicar

Haz los [ejercicios 3.1 de listas](listas.md). Compara cómo se escribe lo mismo en otro lenguaje en [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
