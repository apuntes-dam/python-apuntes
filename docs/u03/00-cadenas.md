# 3.0 Cadenas de texto

Una **cadena** es una secuencia de caracteres: un nombre, una frase, el contenido de un archivo. Casi todo programa las maneja, y casi siempre se hace lo mismo: **recorrerlas, buscar dentro, partirlas y transformarlas**.

En Python el texto es un `str`. Las cadenas son **inmutables**: ningún método modifica la original, todos **devuelven una nueva**.

## Recorrer y buscar

```python
texto = "banana"
print(f"longitud: {len(texto)}")
print(f"primera: {texto[0]}, última: {texto[-1]}")
print(f"subcadena: {texto[2:5]}")

cuenta = 0
for letra in texto:
    if letra == "a":
        cuenta += 1
print(f"letras a: {cuenta}")
print(f'posición de "na": {texto.find("na")}')
print(f'contiene "nan": {"sí" if "nan" in texto else "no"}')

al_reves = ""
j = len(texto) - 1
while j >= 0:
    al_reves += texto[j]
    j -= 1
print(f"al revés: {al_reves}")
```

Salida:

```text
longitud: 6
primera: b, última: a
subcadena: nan
letras a: 3
posición de "na": 2
contiene "nan": sí
al revés: ananab
```

Los índices empiezan en **0** y el último es `len(texto) - 1`; además, los **negativos** cuentan desde el final (`texto[-1]` es la última letra). Acceder fuera de rango lanza `IndexError`; en cambio, una porción (`texto[2:50]`) nunca falla: se recorta sola.

Fíjate en el patrón de **recorrido**: una variable que va de la primera posición a la última (o al revés) y, dentro, una pregunta sobre la letra actual. Con él se cuenta, se busca y se invierte; es la base de los ejercicios de este apartado. Las cadenas ya traen métodos que lo hacen por ti (`indexOf`/`find`, `contains`...), pero conviene saber hacerlo a mano.

!!! warning "El final de una subcadena no se incluye"
    `substring(2, 5)` (o `texto[2:5]`) toma las posiciones **2, 3 y 4**. Calcula el tamaño restando: `5 - 2 = 3` letras.

## Transformar

```python
original = "  Hola, Mundo DAM  "
limpio = original.strip()
print(f"recortado: [{limpio}]")
print(f"mayúsculas: {limpio.upper()}")
print(f"minúsculas: {limpio.lower()}")
print(f"reemplazado: {limpio.replace('Mundo', 'Clase')}")

partes = limpio.split(", ")
print(f"partes: {len(partes)} -> {partes[0]} | {partes[1]}")
print(f"unido: {'-'.join(partes)}")
print(f'empieza por "Hola": {"sí" if limpio.startswith("Hola") else "no"}')
print(f"con ceros: {7:03d}")
print(f"original sigue igual: [{original}]")
```

Salida:

```text
recortado: [Hola, Mundo DAM]
mayúsculas: HOLA, MUNDO DAM
minúsculas: hola, mundo dam
reemplazado: Hola, Clase DAM
partes: 2 -> Hola | Mundo DAM
unido: Hola-Mundo DAM
empieza por "Hola": sí
con ceros: 007
original sigue igual: [  Hola, Mundo DAM  ]
```

Observa la última línea: tras todos los cambios, **`original` no ha variado**, porque cada método devuelve una cadena nueva. Si quieres quedarte con el resultado, guárdalo en una variable (`limpio = original.strip()`), no basta con llamar al método.

## Los métodos más útiles

| Operación | En Python |
|---|---|
| Longitud | `len(texto)` |
| Un carácter | `texto[i]` · `texto[-1]` (el último) |
| Subcadena (el final no se incluye) | `texto[2:5]` |
| Buscar posición (`-1` si no está) | `texto.find("na")` |
| ¿Contiene? ¿Empieza? ¿Acaba? | `"nan" in texto` · `startswith("ba")` · `endswith("na")` |
| Reemplazar | `replace("a", "o")` |
| Trocear / unir | `texto.split(", ")` · `"-".join(partes)` |
| Quitar espacios de los extremos | `strip()` |
| Mayúsculas / minúsculas | `upper()` · `lower()` |
| Rellenar con ceros | `f"{7:03d}"` · `"7".zfill(3)` |
| Repetir | `"ab" * 3` |

Para darle la vuelta rápidamente: `texto[::-1]`. Para construir un texto grande con muchos trozos, mete los trozos en una lista y únelos al final con `"".join(lista)`, que es más rápido que `+=` repetido.

## Unicode

`len` cuenta **caracteres Unicode** (un emoji cuenta 1), pero algunos símbolos compuestos (una letra con acento escrita en dos partes, una bandera) siguen siendo varios caracteres.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a un método y esperar que cambie la cadena | Recoge el valor devuelto en una variable |
| Pasarse un puesto al recorrer (`<=` en lugar de `<`) | El último índice es la longitud menos 1 |
| Olvidar que el final de una subcadena no se incluye | Cuenta las letras: `final - inicio` |
| Comparar mayúsculas con minúsculas | Pasa los dos textos a minúsculas antes de comparar |

## Para practicar

Haz los [ejercicios 3.0 de cadenas](cadenas.md). Para ver cómo se escribe lo mismo en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
