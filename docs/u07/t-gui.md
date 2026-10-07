# 7.D Interfaces gráficas

Hasta ahora los programas **leían** de la consola y **escribían** en ella, en un orden fijo. Una **interfaz gráfica** (GUI) cambia la idea: el programa muestra una **ventana** y **espera** a que el usuario haga algo (escribir, pulsar, elegir). Se dice que está **dirigido por eventos**.

## Ideas comunes a todas las librerías

| Idea | Qué es |
|---|---|
| **Componente** (*widget*) | Cada elemento de la ventana: texto, campo de entrada, botón, lista, imagen |
| **Contenedor y diseño** (*layout*) | Cómo se colocan los componentes: en columna, en fila, en rejilla |
| **Evento** | Algo que ocurre: una pulsación, una tecla, un clic |
| **Controlador de eventos** (*handler* o *callback*) | El código que se ejecuta cuando ocurre el evento |
| **Estado** | Los datos que cambian y que determinan lo que se ve (el texto escrito, el resultado) |
| **Bucle de eventos** | El ciclo interno que espera eventos y los reparte; se pone en marcha al abrir la ventana |

El programa no controla el orden: **reacciona**. Por eso cada botón lleva asociado su controlador.

En Python la librería incluida en la instalación es **Tkinter** (`tkinter`). Se crean **objetos** (`Tk` para la ventana, `Label`, `Entry`, `Button`...), se **colocan** con un gestor de posición (`grid`, `pack`) y se les asocia una **función** (`command=`) que se ejecuta al pulsar. Es un enfoque **imperativo**: tú creas los componentes y cambias sus propiedades (`resultado.config(text=...)`).

## Un ejemplo: la propina

La ventana tiene un campo para escribir un importe, un botón «Calcular» y un texto con el resultado (el 10 % del importe). Si lo escrito no es un número entero o es negativo, muestra un mensaje de error.

```python
import tkinter as tk


def calcular_propina(texto):
    """Devuelve el mensaje con la propina (el 10 %) del importe escrito."""
    try:
        importe = int(texto.strip())
    except ValueError:
        return "Escribe un número entero"
    if importe < 0:
        return "El importe no puede ser negativo"
    return f"Propina: {importe // 10} €"


class VentanaPropina:
    def __init__(self):
        self.ventana = tk.Tk()
        self.ventana.title("Propina")

        tk.Label(self.ventana, text="Importe (€):").grid(row=0, column=0, padx=8, pady=8)
        self.entrada = tk.Entry(self.ventana)
        self.entrada.grid(row=0, column=1, padx=8, pady=8)

        self.boton = tk.Button(self.ventana, text="Calcular", command=self.calcular)
        self.boton.grid(row=1, column=0, columnspan=2, pady=4)

        self.resultado = tk.Label(self.ventana, text="")
        self.resultado.grid(row=2, column=0, columnspan=2, pady=8)

    def calcular(self):
        self.resultado.config(text=calcular_propina(self.entrada.get()))

    def ejecutar(self):
        self.ventana.mainloop()


if __name__ == "__main__":
    VentanaPropina().ejecutar()
```

Se ejecuta con `python gui1.py`.

Hay una decisión de diseño importante: **`calcularPropina` es una función aparte**, que recibe un texto y devuelve otro, sin saber nada de ventanas. La pantalla solo se ocupa de **recoger** el texto, **llamar** a la función y **mostrar** el resultado. Esa separación entre **lógica** e **interfaz** hace el código más fácil de probar, de reutilizar (la misma función serviría en consola o en una web) y de cambiar.

## Probar la ventana sin abrirla

Como los componentes son objetos, una prueba puede **escribir en el campo y simular la pulsación** con `invoke()` sin enseñar la ventana.

```python
from gui1 import VentanaPropina

v = VentanaPropina()
v.ventana.withdraw()  # para probar no hace falta mostrar la ventana
for texto in ["50", "abc", "-5"]:
    v.entrada.delete(0, "end")
    v.entrada.insert(0, texto)
    v.boton.invoke()  # equivale a pulsar el botón
    v.ventana.update()
    print(f"{texto} -> {v.resultado.cget('text')}")
v.ventana.destroy()
```



Resultado de los tres casos:

```text
50 -> Propina: 5 €
abc -> Escribe un número entero
-5 -> El importe no puede ser negativo
```

He ejecutado esta prueba con Tkinter en Python 3.13 y devuelve exactamente esos tres resultados.

## Tres cuidados importantes

* **Nunca bloquees la ventana.** Mientras el controlador de un evento está trabajando, la ventana **no responde**. Las tareas largas (descargas, cálculos pesados, lecturas de archivos grandes) deben hacerse **fuera del hilo de la interfaz**.
* **Valida lo que escribe el usuario.** Todo lo que llega de un campo es **texto**: conviértelo y prevé que falle, como hace `calcularPropina`.
* **Cambia la interfaz desde el sitio correcto.** En Tkinter, todo debe hacerse desde el hilo principal; si necesitas trabajar en otro hilo, devuelve el resultado a la ventana con `ventana.after(...)`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| La pantalla no se actualiza al cambiar un dato | Cambiar el estado **del modo que la librería exige** (ver arriba) |
| Escribir la lógica dentro del controlador del botón | Ponerla en una función aparte |
| Congelar la ventana con una tarea larga | Hacerla fuera del hilo de la interfaz |
| Fiarse de que el usuario escribirá un número | Validar y mostrar un mensaje claro |
| Componentes que se salen o se superponen | Usar un contenedor con diseño (columna, rejilla) en vez de posiciones fijas |

## Para practicar

Haz los ejercicios de [U7.4 · Interfaces gráficas](gui.md): una ventana con botón, un contador con estado, un formulario con validación y una lista de tareas. Si quieres profundizar en Compose, tienes la [web de Android y apps móviles](https://apuntes-dam.github.io/android-apuntes/), que trata botones, listas, formularios y navegación. [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) compara el código de cada lenguaje.
