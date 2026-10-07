# 3.2 Mapas (diccionarios)

Un **mapa** (o **diccionario**) guarda **pares clave → valor**. En lugar de buscar por posición, como en una lista, se busca por clave: dado un nombre, su edad; dada una palabra, cuántas veces aparece. Las **claves son únicas**: si asignas otra vez la misma clave, se sobrescribe el valor.

En Python es el **diccionario** `dict`: `{"Ana": 25}`. Conserva el **orden de inserción**. Las claves tienen que ser inmutables (texto, números, tuplas), no listas.

## Crear, consultar, modificar y borrar

```python
edades = {"Ana": 25, "Luis": 30}
print(f"edad de Ana: {edades['Ana']}")
print(f"Pepe: {edades.get('Pepe', 'no está')}")
print(f"Pepe con valor por defecto: {edades.get('Pepe', 0)}")

edades["Eva"] = 22  # añadir
edades["Ana"] = 26  # modificar
for clave, valor in edades.items():
    print(f"{clave} -> {valor}")
del edades["Luis"]
print(f"tras borrar a Luis quedan {len(edades)}")

cuentas = {}
for palabra in "a b a c b a".split(" "):
    cuentas[palabra] = cuentas.get(palabra, 0) + 1
print("frecuencias:")
for clave, valor in cuentas.items():
    print(f"{clave} -> {valor}")
```

Salida:

```text
edad de Ana: 25
Pepe: no está
Pepe con valor por defecto: 0
Ana -> 26
Luis -> 30
Eva -> 22
tras borrar a Luis quedan 2
frecuencias:
a -> 3
b -> 2
c -> 1
```

| Operación | En Python |
|---|---|
| Consultar (**error** si no está) | `edades["Ana"]` → `KeyError` si falta |
| Consulta segura / valor por defecto | `edades.get("Pepe", 0)` |
| Añadir o modificar | `edades["Eva"] = 22` |
| ¿Existe la clave? | `"Ana" in edades` |
| Borrar | `del edades["Luis"]` · `edades.pop("Luis")` |
| Tamaño | `len(edades)` |
| Solo claves · solo valores | `edades.keys()` · `edades.values()` |
| Recorrer pares | `for clave, valor in edades.items():` |

!!! warning "Consultar una clave que no existe"
    **Consultar con corchetes una clave que no existe lanza `KeyError`.** Si la clave puede no estar, usa `get(clave, valor_por_defecto)` o comprueba antes con `in`.

## Contar con un mapa

La segunda mitad del ejemplo es un patrón que se usa constantemente: **contar cuántas veces aparece cada elemento**. Para cada palabra, se lee su cuenta actual (o 0 si es la primera vez), se suma 1 y se guarda de nuevo.

## Claves, valores y mapas de listas

```python
notas = {
    "Ana": [7, 9],
    "Luis": [5, 6, 7],
}
print("claves: " + ", ".join(notas.keys()))
for nombre, lista in notas.items():
    suma = 0
    for n in lista:
        suma += n
    print(f"{nombre}: media {suma / len(lista)}")
print(f"total de notas: {sum(len(lista) for lista in notas.values())}")
```

Salida:

```text
claves: Ana, Luis
Ana: media 8.0
Luis: media 6.0
total de notas: 5
```

Los valores pueden ser de cualquier tipo, incluidas **listas** u otros mapas. Aquí cada alumno tiene una lista de notas; se recorre el mapa y, para cada par, se recorre su lista.

## ¿Mapa, lista o conjunto?

| Necesito... | Estructura |
|---|---|
| Datos ordenados a los que accedo por posición | Lista |
| Buscar un dato a partir de otro (nombre → edad) | **Mapa** |
| Saber si algo está, sin repeticiones | Conjunto |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar el valor de una clave que no existe | Comprueba, o usa el valor por defecto |
| Esperar un orden concreto | Usa una versión ordenada si el orden importa |
| Modificar el mapa mientras lo recorres | Recorre una copia de las claves o construye un mapa nuevo |
| Repetir una clave pensando que añade otra entrada | Las claves son únicas: la segunda **sobrescribe** |

## Para practicar

Haz los [ejercicios 3.2 de mapas](mapas.md). Y para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
