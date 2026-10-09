# 7.B Archivos y carpetas

Los programas guardan datos en **archivos**, organizados en **carpetas** (directorios). Saber consultar, crear, copiar, mover y borrar es la base para cualquier programa que trabaje con datos que deben sobrevivir al cierre.

!!! tip "Qué excepción salta y qué no"
    Aquí se usan las operaciones de `Path`. En [7.E](t-path.md) están las **excepciones concretas** de cada una (`FileExistsError`, `FileNotFoundError`...) y las operaciones que **no** dan error aunque el destino exista, como `shutil.copy`.

## Rutas

Una **ruta** indica dónde está un archivo o carpeta.

| Tipo | Ejemplo | Significa |
|---|---|---|
| **Absoluta** | `C:/Users/ana/datos/notas.txt` (Windows) · `/home/ana/datos/notas.txt` (Linux, macOS) | Desde la raíz del sistema |
| **Relativa** | `datos/notas.txt` | Desde la **carpeta de trabajo** del programa (la carpeta desde la que se ejecuta) |

!!! tip "Usa `/` en las rutas"
    Aunque Windows escribe las rutas con `\`, en Python (y en los otros tres lenguajes) la barra normal `/` funciona en todos los sistemas y evita problemas con el carácter de escape `\`.

## Las operaciones básicas

Se usa el módulo **`pathlib`** (**`Path`**) para las rutas y sus operaciones (`exists`, `is_dir`, `iterdir`, `mkdir`, `rename`, `stat`...) y el módulo **`shutil`** para copiar y borrar carpetas enteras (`copy`, `rmtree`).

```python
import shutil
from pathlib import Path


def nombres(carpeta):
    return sorted(p.name for p in carpeta.iterdir())


def main():
    carpeta = Path("datos")
    (carpeta / "sub").mkdir(parents=True, exist_ok=True)
    print(f"carpeta creada: {'sí' if carpeta.exists() else 'no'}")

    (carpeta / "notas.txt").write_text("uno\ndos\n", encoding="utf-8", newline="\n")
    for nombre in nombres(carpeta):
        ruta = carpeta / nombre
        if ruta.is_dir():
            print(f"{nombre}: carpeta")
        else:
            print(f"{nombre}: archivo de {ruta.stat().st_size} bytes")

    shutil.copy(carpeta / "notas.txt", carpeta / "notas_copia.txt")
    print("tras copiar: " + ", ".join(nombres(carpeta)))
    (carpeta / "notas_copia.txt").rename(carpeta / "resumen.txt")
    print("tras renombrar: " + ", ".join(nombres(carpeta)))

    shutil.rmtree(carpeta)
    print(f"datos existe tras borrar: {'sí' if carpeta.exists() else 'no'}")


if __name__ == "__main__":
    main()
```

Salida:

```text
carpeta creada: sí
notas.txt: archivo de 8 bytes
sub: carpeta
tras copiar: notas.txt, notas_copia.txt, sub
tras renombrar: notas.txt, resumen.txt, sub
datos existe tras borrar: no
```

El programa crea una carpeta con una subcarpeta, escribe un archivo, **inspecciona** lo que hay (¿archivo o carpeta? ¿cuánto ocupa?), lo copia, lo renombra y lo borra todo al final. Fíjate en que la lista de nombres se **ordena** antes de mostrarla: el sistema no garantiza ningún orden al listar una carpeta.

| Necesito... | En Python |
|---|---|
| ¿Existe? ¿Es carpeta? | `p.exists()` · `p.is_dir()` |
| Tamaño en bytes | `p.stat().st_size` |
| Crear carpeta (con las que falten) | `p.mkdir(parents=True, exist_ok=True)` |
| Listar el contenido | `p.iterdir()` |
| Copiar · renombrar o mover | `shutil.copy(a, b)` · `a.rename(b)` |
| Borrar archivo · carpeta con todo lo que contiene | `p.unlink()` · `shutil.rmtree(p)` |

## Cuando algo falla

Un archivo que no existe lanza **`FileNotFoundError`**, un archivo ya existente, **`FileExistsError`**, la falta de permisos, **`PermissionError`**, y borrar una carpeta con contenido con `rmdir`, **`OSError`**. Todas heredan de `OSError`.

Los fallos son **normales** con archivos (el usuario escribe mal una ruta, el disco está lleno, otro programa tiene el archivo abierto). Un programa robusto **comprueba antes** lo que pueda (¿existe?) y **captura** lo demás para dar un mensaje claro.

!!! danger "Cuidado con lo que borras"
    Borrar con `recursive`, `rmtree`, `deleteRecursively` o `walk` elimina **todo** lo que haya dentro, sin papelera y sin vuelta atrás. Antes de borrar:

    * Comprueba que la ruta es **la que esperas** (no vacía, no la raíz de un disco, no tu carpeta de usuario).
    * Si el borrado lo decide el usuario, **pídele confirmación**.
    * Mientras pruebas, usa una carpeta de prueba aparte.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Rutas relativas que dependen de dónde se ejecute el programa | Comprobar la carpeta de trabajo, o usar rutas absolutas |
| Asumir un orden al listar una carpeta | Ordenar los nombres |
| Sobrescribir un archivo existente sin avisar | Comprobar si existe y preguntar antes |
| Borrar una carpeta no vacía con la función de un solo archivo | Usar la versión recursiva, con confirmación |
| Olvidar cerrar lo que se abre (en Java, `Files.list`) | `try-with-resources` |

## Para practicar

Haz los ejercicios de [U7.2 · Archivos y carpetas](archivos.md). Y para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
