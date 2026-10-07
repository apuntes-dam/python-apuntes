# 3.5 XML

**XML** (*eXtensible Markup Language*) es otro formato de texto para guardar datos **con estructura de árbol**. Es más antiguo y más verboso que JSON, pero sigue muy presente en configuraciones, documentos, servicios web antiguos y en el propio Android.

```xml
<usuarios>
  <usuario>
    <id>1</id>
    <nombre>Juan</nombre>
    <edad>30</edad>
  </usuario>
</usuarios>
```

## Partes de un XML

| Parte | Qué es | Ejemplo |
|---|---|---|
| **Elemento** | Una etiqueta de apertura, su contenido y la de cierre | `<nombre>Juan</nombre>` |
| **Atributo** | Un dato dentro de la etiqueta de apertura | `<usuario id="1">` |
| **Texto** | El contenido de un elemento | `Juan` |
| **Raíz** | El elemento que contiene a todos los demás (solo hay **uno**) | `<usuarios>` |

Para que un XML sea **válido** (*bien formado*): hay una sola raíz, cada etiqueta que se abre **se cierra** en el orden correcto, los atributos van entre comillas y se distingue entre mayúsculas y minúsculas. Los caracteres especiales se escriben con entidades: `&lt;` (`<`), `&gt;` (`>`) y `&amp;` (`&`).

Python incluye **`xml.etree.ElementTree`**: el documento se carga como un árbol de elementos, fácil de recorrer y de modificar.

## Leer, modificar y escribir

```python
import xml.etree.ElementTree as ET


def mostrar(raiz):
    for u in raiz.findall("usuario"):
        print(f"ID: {u.findtext('id')}, Nombre: {u.findtext('nombre')}, Edad: {u.findtext('edad')}")


xml = ("<usuarios><usuario><id>1</id><nombre>Juan</nombre><edad>30</edad></usuario>"
       "<usuario><id>2</id><nombre>Ana</nombre><edad>25</edad></usuario></usuarios>")
raiz = ET.fromstring(xml)
mostrar(raiz)

for u in raiz.findall("usuario"):
    if u.findtext("nombre") == "Ana":
        u.find("edad").text = "26"                                   # actualizar

nuevo = ET.SubElement(raiz, "usuario")                               # insertar
for etiqueta, valor in (("id", "3"), ("nombre", "Eva"), ("edad", "22")):
    ET.SubElement(nuevo, etiqueta).text = valor

for u in raiz.findall("usuario"):                                    # eliminar
    if u.findtext("id") == "1":
        raiz.remove(u)

print("--- después de los cambios ---")
mostrar(raiz)
print(ET.tostring(raiz, encoding="unicode"))
```

Salida:

```text
ID: 1, Nombre: Juan, Edad: 30
ID: 2, Nombre: Ana, Edad: 25
--- después de los cambios ---
ID: 2, Nombre: Ana, Edad: 26
ID: 3, Nombre: Eva, Edad: 22
<usuarios><usuario><id>2</id><nombre>Ana</nombre><edad>26</edad></usuario><usuario><id>3</id><nombre>Eva</nombre><edad>22</edad></usuario></usuarios>
```

La idea es la misma que con JSON: cargar el texto como un **árbol**, buscar los elementos que interesan (`usuario`, y dentro `id`, `nombre`, `edad`), modificar el árbol y volver a convertirlo en texto. El texto de entrada de este ejemplo está en **una sola línea** para que la salida sea idéntica en los cuatro lenguajes; en un archivo real lo normal es escribirlo con sangría.

## Crear un árbol, atributos y errores

```python
import xml.etree.ElementTree as ET

raiz = ET.Element("usuarios")
usuario = ET.SubElement(raiz, "usuario", id="1")
usuario.text = "Juan"
print(ET.tostring(raiz, encoding="unicode"))
print(f"atributo id: {usuario.get('id')}")

try:
    ET.fromstring("<usuarios><usuario></usuarios>")
except ET.ParseError:
    print("El texto no es un XML válido")
```

Salida:

```text
<usuarios><usuario id="1">Juan</usuario></usuarios>
atributo id: 1
El texto no es un XML válido
```

Este ejemplo muestra tres cosas: **crear** un árbol desde cero (un XML vacío con su raíz es el punto de partida cuando un archivo no existe), **leer un atributo** (`id`) y **detectar un XML inválido**. Un XML mal formado lanza **`xml.etree.ElementTree.ParseError`**.

## JSON o XML

| | JSON | XML |
|---|---|---|
| Aspecto | Compacto | Más verboso (etiquetas de apertura y cierre) |
| Estructura | Objetos y arrays | Árbol de elementos, con atributos |
| Comentarios | No | Sí |
| Uso típico | APIs web, configuración | Documentos, configuraciones, Android |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Olvidar cerrar una etiqueta o cerrarla en otro orden | Comprueba que el XML sea válido antes de procesarlo |
| Más de un elemento raíz | Un documento tiene **una** sola raíz |
| `&` o `<` sueltos dentro del texto | Escríbelos como `&amp;` y `&lt;` |
| Asumir que un elemento existe | Comprueba que el resultado de la búsqueda no sea nulo |

## Para practicar

Haz el [ejercicio 3.5 de XML](xml.md), igual que el de JSON pero con un árbol de elementos.
