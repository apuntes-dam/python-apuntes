# 3.4 JSON

**JSON** (*JavaScript Object Notation*) es un formato de **texto** para guardar e intercambiar datos. Es el formato más usado entre aplicaciones y servidores web, porque es compacto, fácil de leer para una persona y todos los lenguajes saben procesarlo.

```json
{
  "usuarios": [
    {"id": 1, "nombre": "Juan", "edad": 30},
    {"id": 2, "nombre": "Ana", "edad": 25}
  ]
}
```

## Qué contiene un JSON

| En JSON | Ejemplo | En Python es... |
|---|---|---|
| Objeto `{ }` | `{"id": 1}` | un mapa (o un objeto de una clase) |
| Array `[ ]` | `[1, 2, 3]` | una lista |
| Texto | `"Ana"` | una cadena |
| Número | `25` · `3.5` | un entero o decimal |
| Booleano | `true` · `false` | un booleano |
| Nulo | `null` | el valor nulo |

Las reglas son estrictas: las claves y los textos van **siempre entre comillas dobles**, **no se admite una coma al final** de una lista ni de un objeto y **no hay comentarios**. La mayoría de errores con JSON son una de estas tres cosas.

Python incluye el módulo **`json`**: `json.loads(texto)` convierte un texto en diccionarios y listas, `json.load(archivo)` lo hace desde un archivo, y `json.dumps(datos, indent=2)` / `json.dump(datos, archivo)` hacen lo contrario. Añade `ensure_ascii=False` para que las tildes se escriban tal cual en lugar de como `\u00e9`.

## Leer, modificar y escribir

```python
import json


def mostrar(datos):
    for u in datos["usuarios"]:
        print(f"ID: {u['id']}, Nombre: {u['nombre']}, Edad: {u['edad']}")


texto = '{"usuarios": [{"id": 1, "nombre": "Juan", "edad": 30}, {"id": 2, "nombre": "Ana", "edad": 25}]}'
datos = json.loads(texto)
mostrar(datos)

datos["usuarios"][1]["edad"] = 26                                    # actualizar
datos["usuarios"].append({"id": 3, "nombre": "Eva", "edad": 22})     # insertar
datos["usuarios"] = [u for u in datos["usuarios"] if u["id"] != 1]   # eliminar

print("--- después de los cambios ---")
mostrar(datos)
print(json.dumps(datos, indent=2, ensure_ascii=False))
```

Salida:

```text
ID: 1, Nombre: Juan, Edad: 30
ID: 2, Nombre: Ana, Edad: 25
--- después de los cambios ---
ID: 2, Nombre: Ana, Edad: 26
ID: 3, Nombre: Eva, Edad: 22
{
  "usuarios": [
    {
      "id": 2,
      "nombre": "Ana",
      "edad": 26
    },
    {
      "id": 3,
      "nombre": "Eva",
      "edad": 22
    }
  ]
}
```

El proceso siempre es el mismo en tres pasos:

1. **Convertir el texto** en estructuras del lenguaje (mapas, listas u objetos).
2. **Trabajar con ellas** como con cualquier mapa o lista. Se actualiza asignando (`datos["usuarios"][1]["edad"] = 26`), se inserta con `append` y se elimina reconstruyendo la lista sin ese elemento, como con cualquier lista o diccionario.
3. **Convertirlas de nuevo en texto**, con sangría si lo van a leer personas.

## Trabajar con archivos y controlar los errores

Los datos suelen estar en un **archivo**. Al leerlo pueden pasar dos cosas que hay que controlar siempre: que el archivo **no exista** o que su contenido **no sea un JSON válido**. Un JSON mal formado lanza **`json.JSONDecodeError`**.

```python
import json
import os


def cargar(ruta):
    if not os.path.exists(ruta):
        print(f"No existe el archivo '{ruta}'")
        return None
    try:
        with open(ruta, encoding="utf-8") as f:
            return json.load(f)["usuarios"]
    except json.JSONDecodeError:
        print(f"El archivo '{ruta}' no contiene un JSON válido")
        return None


if __name__ == "__main__":
    cargar("no_existe.json")

    with open("malo.json", "w", encoding="utf-8") as f:
        f.write("{ esto no es json")
    cargar("malo.json")

    datos = {"usuarios": [{"id": 1, "nombre": "Juan", "edad": 30}, {"id": 2, "nombre": "Ana", "edad": 25}]}
    with open("bueno.json", "w", encoding="utf-8") as f:
        json.dump(datos, f)
    usuarios = cargar("bueno.json")
    print(f"Cargados {len(usuarios)} usuarios")

    os.remove("malo.json")
    os.remove("bueno.json")
```

Salida:

```text
No existe el archivo 'no_existe.json'
El archivo 'malo.json' no contiene un JSON válido
Cargados 2 usuarios
```

Comprobar la existencia **antes** de abrir y capturar el error de formato permite dar un mensaje claro en vez de que el programa se detenga. Los archivos de texto se guardan en **UTF-8**, la codificación habitual del JSON.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Comillas simples en el texto del JSON | En JSON solo valen las **dobles** |
| Una coma de más al final | Quítala: JSON no la permite |
| Esperar un número y recibir texto (o al revés) | Comprueba el tipo o convierte |
| Pedir una clave que no existe | Comprueba con las consultas seguras del apartado 3.2 |
| Perder las tildes al guardar | Usa UTF-8 en lectura y escritura |

## Para practicar

Haz el [ejercicio 3.4 de JSON](json.md), la gestión de usuarios en un archivo.
