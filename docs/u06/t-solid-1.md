# 6.B SOLID: responsabilidad única y abierto/cerrado

**SOLID** es el nombre de **cinco principios de diseño** con objetos. No son leyes ni se aplican al pie de la letra, pero resumen lo que se ha comprobado que hace el código más fácil de **entender, cambiar y probar**. Cada letra es un principio:

| Letra | Principio | En una frase |
|---|---|---|
| **S** | Responsabilidad única | Una clase, **una razón para cambiar** |
| **O** | Abierto/cerrado | Ampliar **sin modificar** lo que ya funciona |
| **L** | Sustitución de Liskov | Una hija debe poder usarse **donde se pida su base** sin sorpresas |
| **I** | Segregación de interfaces | **Contratos pequeños**, no obligar a implementar lo que no se usa |
| **D** | Inversión de dependencias | Depender de **abstracciones**, no de clases concretas |

En esta página, las dos primeras; en la [siguiente](t-solid-2.md), las otras tres. En cada ejemplo hay primero una versión **«antes»** con el problema y después una versión **«después»** ya corregida.

!!! info "Cómo leer los ejemplos"
    Los dos programas hacen **lo mismo** y producen el mismo resultado. Lo que cambia es **cómo está organizado el código**, y eso se nota cuando hay que modificarlo.

## S · Responsabilidad única

Una clase debería tener **una sola razón para cambiar**. Si una clase hace varias cosas (validar, guardar, avisar), cualquier cambio en una de ellas obliga a tocar la clase y puede romper las demás.

```python
# ANTES: una sola clase valida, guarda y avisa (tres responsabilidades)
class RegistroMalo:
    def __init__(self):
        self.usuarios = []
        self.enviados = []

    def registrar(self, correo):
        if "@" not in correo:
            return "correo no válido"  # validar
        self.usuarios.append(correo)  # guardar
        self.enviados.append(f"Bienvenido, {correo}")  # avisar
        return "registrado"


# DESPUÉS: cada clase tiene una sola razón para cambiar
class ValidadorCorreo:
    def es_valido(self, correo):
        return "@" in correo


class RepositorioUsuarios:
    def __init__(self):
        self._correos = []

    def guardar(self, correo):
        self._correos.append(correo)

    @property
    def total(self):
        return len(self._correos)


class Avisador:
    def __init__(self):
        self.enviados = []

    def bienvenida(self, correo):
        self.enviados.append(f"Bienvenido, {correo}")


class ServicioRegistro:
    def __init__(self, validador, repositorio, avisador):
        self.validador = validador
        self.repositorio = repositorio
        self.avisador = avisador

    def registrar(self, correo):
        if not self.validador.es_valido(correo):
            return "correo no válido"
        self.repositorio.guardar(correo)
        self.avisador.bienvenida(correo)
        return "registrado"


if __name__ == "__main__":
    correos = ["ana@correo.es", "correo-malo"]

    malo = RegistroMalo()
    for c in correos:
        print(f"[antes] {c} -> {malo.registrar(c)}")

    repositorio = RepositorioUsuarios()
    avisador = Avisador()
    servicio = ServicioRegistro(ValidadorCorreo(), repositorio, avisador)
    for c in correos:
        print(f"[después] {c} -> {servicio.registrar(c)}")
    print(f"usuarios guardados (después): {repositorio.total}")
    print("avisos enviados (después): " + ", ".join(avisador.enviados))
```

Salida:

```text
[antes] ana@correo.es -> registrado
[antes] correo-malo -> correo no válido
[después] ana@correo.es -> registrado
[después] correo-malo -> correo no válido
usuarios guardados (después): 1
avisos enviados (después): Bienvenido, ana@correo.es
```

* **Antes:** `RegistroMalo` tiene tres responsabilidades. Si cambia la regla de validación, **o** el sitio donde se guardan los usuarios, **o** el texto del aviso, hay que editar la misma clase.
* **Después:** cada clase hace una cosa (`ValidadorCorreo`, `RepositorioUsuarios`, `Avisador`) y `ServicioRegistro` las **coordina**. Un cambio afecta a una sola clase, y cada pieza se puede probar por separado.

**Cómo detectarlo:** si para describir una clase necesitas decir «y» («valida **y** guarda **y** avisa»), probablemente son varias clases.

## O · Abierto/cerrado

El código debería estar **abierto a la ampliación** (se pueden añadir comportamientos nuevos) pero **cerrado a la modificación** (no hace falta tocar lo que ya funciona). Cada vez que se edita código que ya estaba probado, se arriesga a romperlo.

```python
from abc import ABC, abstractmethod


# ANTES: añadir un formato obliga a EDITAR esta función
def exportar(formato, datos):
    if formato == "txt":
        return " ".join(datos)
    elif formato == "csv":
        return ",".join(datos)
    else:
        raise ValueError(f"formato desconocido: {formato}")


# DESPUÉS: añadir un formato es añadir una clase; lo que ya existe no cambia
class Exportador(ABC):
    nombre: str

    @abstractmethod
    def exportar(self, datos):
        ...


class ExportadorTxt(Exportador):
    nombre = "txt"

    def exportar(self, datos):
        return " ".join(datos)


class ExportadorCsv(Exportador):
    nombre = "csv"

    def exportar(self, datos):
        return ",".join(datos)


# formato nuevo, escrito sin tocar el código anterior
class ExportadorJson(Exportador):
    nombre = "json"

    def exportar(self, datos):
        return "[" + ",".join(f'"{d}"' for d in datos) + "]"


if __name__ == "__main__":
    datos = ["uno", "dos", "tres"]
    for formato in ["txt", "csv", "json"]:
        try:
            print(f"[antes] {formato}: {exportar(formato, datos)}")
        except ValueError as e:
            print(f"[antes] {formato}: Error: {e}")

    exportadores = [ExportadorTxt(), ExportadorCsv(), ExportadorJson()]
    for e in exportadores:
        print(f"[después] {e.nombre}: {e.exportar(datos)}")
```

Salida:

```text
[antes] txt: uno dos tres
[antes] csv: uno,dos,tres
[antes] json: Error: formato desconocido: json
[después] txt: uno dos tres
[después] csv: uno,dos,tres
[después] json: ["uno","dos","tres"]
```

* **Antes:** para añadir el formato `json` hay que **editar** la función `exportar`. Y así con cada formato nuevo: la cadena de `if` no deja de crecer.
* **Después:** cada formato es una clase que cumple un contrato (`Exportador`). Para añadir `json` se **escribe una clase nueva** y las demás **no se tocan**.

**Cómo detectarlo:** una cadena de `if`/`else if` o un `switch` que cambia según un «tipo» y que hay que ampliar cada vez que aparece uno nuevo. La solución casi siempre combina una **interfaz** ([5.C](../u05/t-interfaces.md)) con **polimorfismo**.

!!! warning "No te pases"
    No hace falta una interfaz por cada `if`. Con dos o tres casos que probablemente no cambiarán, un `if` sencillo es perfectamente válido. Aplica el principio cuando notes que **de verdad** se añaden variantes con frecuencia.

## Para practicar

Haz los ejercicios S y O de [U6.2 · SOLID](solid.md). Y la comparación entre lenguajes de interfaces y funciones está en [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
