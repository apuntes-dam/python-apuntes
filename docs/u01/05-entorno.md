# 1.5 Entorno: instalar y preparar un proyecto

## Qué hace falta

| Pieza | Para qué |
|---|---|
| **Python 3.x** | El intérprete |
| **Editor / IDE** | VS Code (extensión Python) o PyCharm |
| **`pip`** | Instalar librerías |
| **`venv`** | Entorno virtual por proyecto |

## Comprobar la instalación

```bash
python --version
pip --version
```

En Windows también existe `py --version`.

## Primer programa

```python
# hola.py
print("Hola, Python")
```

```bash
python hola.py
```

## Entorno virtual

Un entorno virtual aísla las librerías de cada proyecto.

```bash
python -m venv .venv
.venv\Scripts\activate        # Windows
source .venv/bin/activate     # Linux / macOS
pip install pytest
pip freeze > requirements.txt
```

!!! note "Errores frecuentes"
    * `python` no se reconoce: no está en el `PATH`; reinstala marcando *Add Python to PATH*.
    * `ModuleNotFoundError`: la librería no está instalada en el entorno activo.
    * `IndentationError`: mezcla de tabuladores y espacios o bloque mal indentado.
