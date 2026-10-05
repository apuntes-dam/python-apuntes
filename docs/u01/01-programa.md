# 1.1 Un programa

Un **programa** es una secuencia de instrucciones que un ordenador ejecuta para resolver un problema. Antes de escribirlo hay que tener claro el **algoritmo**: los pasos, en orden, que llevan de unos datos de entrada a un resultado.

## Ciclo de desarrollo

1. **Analizar** el problema: qué entra, qué debe salir.
2. **Diseñar** el algoritmo (pseudocódigo o diagrama).
3. **Codificar** en un lenguaje (aquí, Python).
4. **Probar** y corregir.
5. **Documentar** y mantener.

## Pseudocódigo

```text
ALGORITMO areaRectangulo
  LEER base
  LEER altura
  area <- base * altura
  ESCRIBIR area
FIN
```

## Del pseudocódigo a Python

```python
base = float(input("Base: "))
altura = float(input("Altura: "))

area = base * altura
print(f"Área: {area}")
```

| Pseudocódigo | Python |
|---|---|
| `LEER x` | `input("mensaje")` (devuelve texto; se convierte con `int()`, `float()`) |
| `x <- expresión` | `x = expresión` |
| `ESCRIBIR x` | `print(x)` |

!!! warning "Errores típicos al diseñar"
    Olvidar un caso (por ejemplo, base cero), usar una variable antes de asignarla (`NameError`) y olvidar convertir lo que devuelve `input`, que siempre es texto.
