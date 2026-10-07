# A5.A Patrones de creación

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 6 (clases, herencia, interfaces y SOLID) y [A1](../a1/index.md) (funciones como valores).

Un **patrón de diseño** es una solución conocida a un problema que se repite al diseñar programas. No es una librería ni código que copiar: es una **forma de organizar clases** que otros programadores ya reconocen. Saber su nombre sirve para algo muy práctico: decir «esto es una fábrica» comunica en tres palabras lo que sin él costaría un párrafo.

Los patrones **de creación** tratan de **cómo se crean los objetos**. Todos los ejemplos de esta unidad usan una cafetería.

## Singleton: una sola instancia

Es una clase de la que **solo puede existir un objeto**, y todos reciben el mismo. Sirve para algo realmente único: la configuración de la aplicación, un registro de mensajes.

En Python se puede sobrescribir **`__new__`**, que es el método que *crea* el objeto, para que la primera vez lo cree y las siguientes devuelva el mismo. Pero lo más sencillo suele ser un **módulo**: se importa una sola vez, así que sus variables ya son únicas. Con `a is b` se comprueba que son el mismo objeto.

```python
# Singleton: una clase de la que solo puede existir UNA instancia (aquí, la configuración de la cafetería).
class Configuracion:
    _unica = None

    def __new__(cls):
        # __new__ crea el objeto: la primera vez lo crea, y las siguientes devuelve el mismo
        if cls._unica is None:
            cls._unica = super().__new__(cls)
            cls._unica.local = "Café Central"
            cls._unica.iva = 10
        return cls._unica


# En Python lo más sencillo suele ser un módulo: cada módulo se importa una sola vez, así que sus variables ya son únicas.

a = Configuracion()
b = Configuracion()
print(f"misma instancia: {'sí' if a is b else 'no'}")
print(f"local: {a.local}, IVA {a.iva}%")
```

Salida:

```text
misma instancia: sí
local: Café Central, IVA 10%
```

!!! warning "El patrón más discutido"
    Un singleton es **estado global con buena presentación**: cualquier parte del programa puede cambiarlo, es difícil de sustituir en las pruebas ([A4](../a4/t-aislar.md)) y esconde dependencias. Úsalo solo si de verdad **tiene que** haber uno, y si puedes, **pásalo como parámetro** en lugar de pedírselo a la clase desde cualquier sitio.

## Fábrica: decidir qué clase crear

Una **fábrica** es una función que, a partir de un dato (aquí un nombre), **decide qué clase concreta crear**. Quien la usa solo conoce el tipo común (`Bebida`), no las clases concretas: añadir un tipo nuevo no obliga a cambiar el resto del programa.

Un **diccionario** de nombres a clases sustituye una cadena de `if/elif`: añadir una bebida es añadir una línea.

```python
# Fábrica: una función decide QUÉ clase concreta crear, y quien la usa solo conoce lo que tienen en común.
class Cafe:
    nombre = "café"
    precio = 150  # en céntimos


class Te:
    nombre = "té"
    precio = 120


class Zumo:
    nombre = "zumo"
    precio = 200


CARTA = {"café": Cafe, "té": Te, "zumo": Zumo}  # un diccionario evita una cadena de if/elif


def fabricar(nombre):
    if nombre not in CARTA:
        raise ValueError(f"{nombre}: no está en la carta")
    return CARTA[nombre]()


for nombre in ["café", "té", "zumo", "chocolate"]:
    try:
        bebida = fabricar(nombre)
        print(f"{bebida.nombre}: {bebida.precio} céntimos")
    except ValueError as e:
        print(e)
```

Salida:

```text
café: 150 céntimos
té: 120 céntimos
zumo: 200 céntimos
chocolate: no está en la carta
```

Es la «D» de SOLID ([unidad 6](../../u06/t-solid-2.md)) en acción: el código depende de una **abstracción** (`Bebida`), no de las clases concretas. Y los datos desconocidos se rechazan **en un solo lugar**, con un mensaje claro.

## Builder: construir paso a paso

Un **builder** construye un objeto con **muchos datos opcionales**, con llamadas encadenadas que dicen qué es cada cosa. Lo que no se indica toma su valor por defecto.

En Python los **argumentos con nombre y valores por defecto** (`Pedido(bebida="capuchino", leche="avena")`) resuelven casi siempre el mismo problema sin escribir un builder. Úsalo cuando construir tenga **pasos o reglas**. Cada método devuelve **`self`**, y por eso se pueden encadenar.

```python
# Builder: construir un objeto con muchos datos opcionales paso a paso, con llamadas encadenadas.
# (En Python lo habitual es usar parámetros con nombre y valores por defecto: Pedido(bebida="capuchino", leche="avena").
#  El builder sigue siendo útil cuando la construcción tiene pasos o reglas.)
class Pedido:
    def __init__(self, bebida, tamano, leche, azucar):
        self.bebida = bebida
        self.tamano = tamano
        self.leche = leche
        self.azucar = azucar

    def __str__(self):
        return f"{self.bebida} ({self.tamano}), leche: {self.leche}, {'con' if self.azucar else 'sin'} azúcar"


class PedidoBuilder:
    def __init__(self):
        self._bebida = "café"
        self._tamano = "pequeño"
        self._leche = "ninguna"
        self._azucar = False

    def bebida(self, b):
        self._bebida = b
        return self  # devolver «self» permite encadenar

    def tamano(self, t):
        self._tamano = t
        return self

    def leche(self, l):
        self._leche = l
        return self

    def con_azucar(self):
        self._azucar = True
        return self

    def construir(self):
        return Pedido(self._bebida, self._tamano, self._leche, self._azucar)


print(PedidoBuilder().bebida("capuchino").tamano("grande").leche("avena").construir())
print(PedidoBuilder().con_azucar().construir())  # lo que no se indica toma el valor por defecto
```

Salida:

```text
capuchino (grande), leche: avena, sin azúcar
café (pequeño), leche: ninguna, con azúcar
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar un singleton «porque es cómodo» | Pregúntate si de verdad tiene que haber una sola instancia; si no, pásala como parámetro |
| Una fábrica con un `switch` enorme que crece sin parar | Cuando crezca, usa un diccionario o un registro de clases |
| Un builder para una clase de dos campos | Si bastan parámetros con nombre o un constructor, no lo uses |
| Olvidar validar al final de la construcción | La validación va en `construir()`: un objeto a medias no debería existir |

## Para practicar

Los ejercicios [A5.1, A5.2 y A5.3](ejercicios.md) piden una fábrica de figuras, un registro único y un correo con builder. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
