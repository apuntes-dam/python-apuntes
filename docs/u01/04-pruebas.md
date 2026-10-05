# 1.4 Pruebas con `pytest`

Una **prueba unitaria** comprueba automáticamente que una función devuelve lo esperado. En Python se usa **pytest**.

## Instalación

```bash
pip install pytest
```

## La función

```python
# calculadora.py
def suma(a, b):
    return a + b


def dividir(a, b):
    if b == 0:
        raise ValueError("b no puede ser 0")
    return a / b
```

## La prueba

```python
# test_calculadora.py
import pytest
from calculadora import suma, dividir


def test_suma_dos_positivos():
    assert suma(2, 3) == 5


def test_suma_con_cero():
    assert suma(5, 0) == 5


def test_dividir_por_cero_lanza_error():
    with pytest.raises(ValueError):
        dividir(1, 0)
```

```bash
pytest
```

!!! tip "Patrón AAA"
    **A**rrange (preparar), **A**ct (ejecutar), **A**ssert (comprobar). Los archivos de prueba empiezan por `test_` y las funciones también.
