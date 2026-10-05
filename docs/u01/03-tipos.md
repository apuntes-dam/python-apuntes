# 1.3 Tipos de datos

Python es de **tipado dinámico**: el tipo lo tiene el valor, no la variable, y se comprueba al ejecutar.

## Tipos simples

| Tipo | Ejemplo | Nota |
|---|---|---|
| `int` | `42` | entero de tamaño ilimitado |
| `float` | `3.14` | decimal de 64 bits |
| `str` | `"hola"` | texto, inmutable |
| `bool` | `True` | `True` / `False` |
| `NoneType` | `None` | ausencia de valor |

## Cadenas

```python
s = "Python"
print(len(s))            # 6
print(s.upper())         # PYTHON
print(f"Hola, {s} y {len(s)}")   # f-string
print(s[0], s[-1])       # P n
```

## Colecciones

```python
lista = [3, 1, 2]          # list: ordenada, mutable, admite repetidos
tupla = (3, 1, 2)          # tuple: ordenada, inmutable
conjunto = {1, 1, 2}       # set: sin repetidos -> {1, 2}
diccionario = {"a": 1}     # dict: clave -> valor

lista.append(4)
print(lista)               # [3, 1, 2, 4]
print(len(conjunto))       # 2
print(diccionario["a"])    # 1
```

## Conversiones

```python
print(int("42"))        # 42
print(float("3.5"))     # 3.5
print(str(42))          # 42
print(int(3.9))         # 3 (trunca)
print(float(3))         # 3.0
print(bool(""))         # False (texto vacío)
```

`int("abc")` lanza `ValueError`: hay que controlarlo con `try`/`except` cuando el dato viene del usuario.

!!! note "Conversión implícita"
    Python convierte solo entre números en operaciones mixtas: `1 + 2.5` da `3.5` (`float`). Pero **no** mezcla número y texto: `"a" + 1` es un `TypeError`.
