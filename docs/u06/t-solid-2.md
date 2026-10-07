# 6.C SOLID: Liskov, interfaces pequeñas e inversión de dependencias

Los tres principios restantes de [SOLID](t-solid-1.md). Cada ejemplo vuelve a mostrar la versión «antes» y la «después», con el mismo resultado visible.

## L · Sustitución de Liskov

Donde se pida una clase base, **tiene que poder usarse cualquiera de sus hijas** sin que el programa falle ni se comporte de forma rara. Si una hija **rompe** lo que la base prometía, la herencia está mal planteada, aunque la frase «es un» suene bien.

```python
from abc import ABC


# ANTES: DocumentoSoloLectura "es un" Documento, pero NO se puede usar en su lugar
class Documento:
    def __init__(self, nombre, contenido):
        self.nombre = nombre
        self.contenido = contenido

    def guardar(self, texto):
        self.contenido = texto


class DocumentoSoloLectura(Documento):
    def guardar(self, texto):
        raise PermissionError("es de solo lectura")


# DESPUÉS: la jerarquía refleja lo que cada clase SABE hacer
class Lectura(ABC):
    def __init__(self, nombre, contenido):
        self.nombre = nombre
        self._contenido = contenido

    def leer(self):
        return self._contenido


class Editable(Lectura):
    def guardar(self, texto):
        self._contenido = texto


class SoloLectura(Lectura):
    pass


if __name__ == "__main__":
    antes = [Documento("ficha", "v1"), DocumentoSoloLectura("contrato", "texto original")]
    for d in antes:
        try:
            d.guardar("v2")
            print(f"[antes] {d.nombre}: guardado")
        except PermissionError as e:
            print(f"[antes] {d.nombre}: Error: {e}")

    # solo se guardan documentos Editable: un verificador de tipos avisaría si se pasara un SoloLectura
    editables = [Editable("ficha", "v1")]
    for d in editables:
        d.guardar("v2")
        print(f"[después] {d.nombre}: guardado")
    contrato = SoloLectura("contrato", "texto original")
    print(f"[después] {contrato.nombre} se puede leer: {contrato.leer()}")
```

Salida:

```text
[antes] ficha: guardado
[antes] contrato: Error: es de solo lectura
[después] ficha: guardado
[después] contrato se puede leer: texto original
```

* **Antes:** `DocumentoSoloLectura` «es un» `Documento`, pero cuando se le pide guardar **falla**. Un bucle que guarda todos los documentos funciona con unos y se rompe con otros: la hija **no puede sustituir** a la base.
* **Después:** la jerarquía refleja lo que cada clase **sabe hacer**. Todos se pueden **leer** (`Lectura`); solo los `Editable` se pueden **guardar**. Ya no hay forma de pedir guardar a un documento de solo lectura.

**Señales de que se viola:** una hija que lanza «no soportado», que deja un método vacío, o código que necesita comprobar el tipo concreto antes de usar la clase base.

## I · Segregación de interfaces

Es mejor tener **varias interfaces pequeñas** y específicas que una grande. Ninguna clase debería verse obligada a implementar métodos que no usa.

```python
from abc import ABC, abstractmethod


# ANTES: una interfaz «gorda» obliga a implementar lo que no se usa
class Maquina(ABC):
    @abstractmethod
    def imprimir(self, texto):
        ...

    @abstractmethod
    def escanear(self, texto):
        ...


class ImpresoraSencillaMala(Maquina):
    def imprimir(self, texto):
        return f"impreso: {texto}"

    def escanear(self, texto):
        raise NotImplementedError("no soportado")


# DESPUÉS: contratos pequeños; cada clase cumple solo los suyos
class Impresora(ABC):
    @abstractmethod
    def imprimir(self, texto):
        ...


class Escaner(ABC):
    @abstractmethod
    def escanear(self, texto):
        ...


class ImpresoraSencilla(Impresora):
    def imprimir(self, texto):
        return f"impreso: {texto}"


class Multifuncion(Impresora, Escaner):
    def imprimir(self, texto):
        return f"impreso: {texto}"

    def escanear(self, texto):
        return f"escaneado: {texto}"


if __name__ == "__main__":
    vieja = ImpresoraSencillaMala()
    print(f"[antes] sencilla imprime: {vieja.imprimir('informe')}")
    try:
        vieja.escanear("foto")
    except NotImplementedError as e:
        print(f"[antes] sencilla escanea: Error: {e}")

    sencilla = ImpresoraSencilla()
    multi = Multifuncion()
    print(f"[después] sencilla imprime: {sencilla.imprimir('informe')}")
    # sencilla.escanear("foto")  # un AttributeError: la clase no promete escanear
    print(f"[después] multifunción escanea: {multi.escanear('foto')}")
```

Salida:

```text
[antes] sencilla imprime: impreso: informe
[antes] sencilla escanea: Error: no soportado
[después] sencilla imprime: impreso: informe
[después] multifunción escanea: escaneado: foto
```

* **Antes:** la interfaz `Maquina` obliga a toda impresora a «escanear», aunque no pueda: la implementación acaba lanzando un error.
* **Después:** `Impresora` y `Escaner` son contratos separados. La impresora sencilla cumple uno; la multifunción, los dos. Pedir escanear a una impresora sencilla **ni siquiera se puede escribir**.

**Señales de que se viola:** métodos que lanzan «no soportado» o que están vacíos solo para cumplir la interfaz.

## D · Inversión de dependencias

Las clases importantes no deberían depender de **detalles concretos** (el reloj del sistema, una base de datos, un servicio de correo), sino de **abstracciones** (un contrato) que se les entregan **desde fuera**. Así se puede cambiar el detalle o, muy importante, **sustituirlo por uno falso en las pruebas**.

```python
from abc import ABC, abstractmethod
from datetime import datetime


# ANTES: el Saludador crea su propio reloj: no se puede probar con una hora concreta
class SaludadorMalo:
    def saludo(self):
        hora = datetime.now().hour
        if hora < 12:
            return "Buenos días"
        if hora < 20:
            return "Buenas tardes"
        return "Buenas noches"


# DESPUÉS: depende de una abstracción que le dan desde fuera
class Reloj(ABC):
    @abstractmethod
    def hora(self):
        ...


class RelojReal(Reloj):
    def hora(self):
        return datetime.now().hour


class RelojFijo(Reloj):
    def __init__(self, hora):
        self._hora = hora

    def hora(self):
        return self._hora


class Saludador:
    def __init__(self, reloj):
        self.reloj = reloj

    def saludo(self):
        hora = self.reloj.hora()
        if hora < 12:
            return "Buenos días"
        if hora < 20:
            return "Buenas tardes"
        return "Buenas noches"


if __name__ == "__main__":
    SaludadorMalo().saludo()
    print("[antes] el resultado depende de la hora del equipo")
    for hora in [9, 15, 22]:
        print(f"[después] {hora} h -> {Saludador(RelojFijo(hora)).saludo()}")
    real = Saludador(RelojReal()).saludo()
    print(f"con el reloj real: {'funciona' if real else 'falla'}")
```

Salida:

```text
[antes] el resultado depende de la hora del equipo
[después] 9 h -> Buenos días
[después] 15 h -> Buenas tardes
[después] 22 h -> Buenas noches
con el reloj real: funciona
```

* **Antes:** `SaludadorMalo` pregunta la hora al sistema. Para comprobar que a las 22 h dice «Buenas noches» habría que esperar a las 22 h: no se puede probar.
* **Después:** `Saludador` recibe un `Reloj` (un contrato). En el programa real se le da el `RelojReal`; en las pruebas, un `RelojFijo` con la hora que se quiera. El `Saludador` **no cambia**.

A esta técnica de entregar las dependencias desde fuera, normalmente por el **constructor**, se le llama **inyección de dependencias**. Fíjate en que `Saludador` no escribe `new RelojReal()` por dentro: *recibe* un `Reloj`.

!!! tip "Un objeto falso se llama «doble de prueba»"
    `RelojFijo` es un **doble de prueba** (*fake*): hace de reloj pero controlas lo que devuelve. Es la base de las pruebas automáticas serias, y solo es posible si se diseña con este principio.

## Resumen de los cinco principios

| Principio | Señal de que falla | Remedio habitual |
|---|---|---|
| **S** | Una clase hace varias cosas («y») | Dividirla; una clase coordina |
| **O** | Cadena de `if` según un tipo que crece | Interfaz + una clase por variante |
| **L** | Una hija lanza «no soportado» o necesita comprobaciones de tipo | Rehacer la jerarquía según lo que **sabe hacer** cada clase |
| **I** | Métodos vacíos o que fallan solo para cumplir la interfaz | Dividir la interfaz |
| **D** | La clase crea por dentro sus dependencias y no se puede probar | Recibir una abstracción por el constructor |

## Para practicar

Haz los ejercicios L, I y D de [U6.2 · SOLID](solid.md): en todos hay que partir de un código con el problema y refactorizarlo, como en estos ejemplos. [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) compara cómo se escriben interfaces y funciones en cada lenguaje.
