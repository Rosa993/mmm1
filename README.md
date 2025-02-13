import tkinter as tk
from tkinter import ttk
import random
import time
import pygame  # Importar pygame para la música    
import threading

# Inicializar pygame y la música
pygame.mixer.init()
pygame.mixer.music.load("musi.cs.mp3")  # Cargar el archivo de música

# Reproducir la música en bucle
pygame.mixer.music.play(loops= 1, start=0.0)  # Reproducir en bucle

# Crear la ventana principal
ventana = tk.Tk()
ventana.title("Juego de Operaciones Matemáticas Rápidas")

# Variables globales
tiempo_restante = 5
puntaje = 0
operacion_actual = ""
respuesta_correcta = 0
juego_iniciado = False
victorias = 0  # Contador de victorias

# Función para generar una nueva operación aleatoria
def generar_operacion():
    global operacion_actual, respuesta_correcta, tiempo_restante
    if not juego_iniciado:
        return  # No generar operación si el juego no ha comenzado

    operador = random.choice(["+", "-", "×", "÷"])
    num1 = random.randint(1, 10)
    num2 = random.randint(1, 10)
    
    if operador == "+":
        operacion_actual = f"{num1} + {num2} = ?"
        respuesta_correcta = num1 + num2
    elif operador == "-":
        operacion_actual = f"{num1} - {num2} = ?"
        respuesta_correcta = num1 - num2
    elif operador == "×":
        operacion_actual = f"{num1} × {num2} = ?"
        respuesta_correcta = num1 * num2
    elif operador == "÷":
        # Aseguramos que la división sea exacta
        num1 = num1 * num2
        operacion_actual = f"{num1} ÷ {num2} = ?"
        respuesta_correcta = num1 // num2
    
    etiqueta_operacion.config(text=operacion_actual)
    tiempo_restante = 5  # Reiniciar el tiempo para la nueva operación
    actualizar_tiempo()

# Función para actualizar el temporizador
def actualizar_tiempo():
    global tiempo_restante
    if not juego_iniciado:
        return  # No actualizar el tiempo si el juego no ha comenzado
    
    if tiempo_restante > 0:
        tiempo_restante -= 1
        etiqueta_tiempo.config(text=f"Tiempo: {tiempo_restante}s")
        ventana.after(1000, actualizar_tiempo)  # Llamar nuevamente después de 1 segundo
    else:
        # Si se acaba el tiempo, finalizar la pregunta
        mostrar_resultado(False)

# Función para verificar la respuesta
def verificar_respuesta():
    global puntaje, victorias
    if not juego_iniciado:
        return  # No verificar respuesta si el juego no ha comenzado
    
    try:
        respuesta_usuario = int(entrada_respuesta.get())
        if respuesta_usuario == respuesta_correcta:
            puntaje += 1
            etiqueta_puntaje.config(text=f"Puntaje: {puntaje}")
            mostrar_resultado(True)
            # Verificar si el jugador ha ganado 3 partidas
            if puntaje == 3:
                mostrar_felicidades()  # Mostrar mensaje creativo cuando gane 3 partidas
                victorias += 1
                etiqueta_resultado.config(text="¡Felicidades! Has ganado 3 partidas.")
                reiniciar_juego()
        else:
            mostrar_resultado(False)
    except ValueError:
        mostrar_resultado(False)

# Función para mostrar el resultado de la pregunta
def mostrar_resultado(acertada):
    if acertada:
        resultado = "¡Correcto!"
        boton_iniciar.config(bg="green")  # Cambiar color a verde si es correcto
    else:
        resultado = f"Incorrecto. La respuesta era {respuesta_correcta}."
        boton_iniciar.config(bg="red")  # Cambiar color a rojo si es incorrecto
    
    etiqueta_resultado.config(text=resultado)
    entrada_respuesta.delete(0, tk.END)
    
    # Actualizar el puntaje
    etiqueta_puntaje.config(text=f"Puntaje: {puntaje}")
    
    # Habilitar el botón de inicio después de que termine la operación
    boton_iniciar.config(state=tk.NORMAL)  # Volver a habilitar el botón de inicio

# Función para iniciar el juego
def iniciar_juego():
    global juego_iniciado
    juego_iniciado = True
    boton_iniciar.config(state=tk.DISABLED)  # Deshabilitar el botón de inicio
    boton_iniciar.config(bg="blue")  # Resetear el color a azul al iniciar
    generar_operacion()  # Generar la primera operación

# Función para reiniciar el juego (sin reiniciar el puntaje)
def reiniciar_juego():
    generar_operacion()  # Generar una nueva operación sin reiniciar el puntaje

# Función para mostrar la portada de inicio
def mostrar_portada():
    # Crear un Frame para la portada
    portada_frame = ttk.Frame(ventana)
    portada_frame.pack(fill="both", expand=True)

    # Crear un mensaje de bienvenida
    etiqueta_bienvenida = tk.Label(portada_frame, text="¡Bienvenido al Juego de Operaciones Matemáticas!", font=("Arial", 20))
    etiqueta_bienvenida.grid(row=0, column=0, columnspan=2, pady=20)

    # Instrucciones
    etiqueta_instrucciones = tk.Label(portada_frame, text="Resuelve las operaciones matemáticas lo más rápido posible.", font=("Arial", 16))
    etiqueta_instrucciones.grid(row=1, column=0, columnspan=2, pady=10)

    # Botón para pasar a la siguiente pestaña (Juego)
    boton_siguiente = tk.Button(portada_frame, text="Siguiente", font=("Arial", 16), command=lambda: mostrar_juego(portada_frame), bg="green", fg="white")
    boton_siguiente.grid(row=2, column=0, columnspan=2, pady=20)

# Función para mostrar la pestaña del juego
def mostrar_juego(portada_frame):
    portada_frame.destroy()  # Ocultar la portada

    # Crear el Frame del juego
    juego_frame = ttk.Frame(ventana)
    juego_frame.pack(fill="both", expand=True)

    # Configuración de los elementos gráficos para el juego
    global etiqueta_operacion, etiqueta_tiempo, entrada_respuesta, boton_verificar, boton_iniciar, etiqueta_puntaje, etiqueta_resultado
    etiqueta_operacion = tk.Label(juego_frame, text="Operación", font=("Arial", 24))
    etiqueta_operacion.grid(row=1, column=0, columnspan=2, pady=20)

    etiqueta_tiempo = tk.Label(juego_frame, text="Tiempo: 5s", font=("Arial", 16))
    etiqueta_tiempo.grid(row=2, column=0, columnspan=2)

    entrada_respuesta = tk.Entry(juego_frame, font=("Arial", 16))
    entrada_respuesta.grid(row=3, column=0, pady=10)

    # Botón de "Verificar Respuesta"
    boton_verificar = tk.Button(juego_frame, text="Verificar Respuesta", font=("Arial", 16), command=verificar_respuesta)
    boton_verificar.grid(row=3, column=1, padx=10)

    # Botón de "Iniciar Juego" (este será deshabilitado durante la portada)
    boton_iniciar = tk.Button(juego_frame, text="Iniciar Juego", font=("Arial", 16), command=iniciar_juego, bg="blue", fg="white")
    boton_iniciar.grid(row=4, column=0, columnspan=2, pady=20)

    etiqueta_puntaje = tk.Label(juego_frame, text="Puntaje: 0", font=("Arial", 16))
    etiqueta_puntaje.grid(row=5, column=0, columnspan=2, pady=10)

    etiqueta_resultado = tk.Label(juego_frame, text="", font=("Arial", 16))
    etiqueta_resultado.grid(row=6, column=0, columnspan=2, pady=10)

# Función creativa cuando gane
def mostrar_felicidades():
    # Mostrar un mensaje de felicitaciones con un efecto de expansión y cambio de color
    etiqueta_felicidades = tk.Label(ventana, text="¡Felicidades! Has ganado 3 partidas.", font=("Arial", 14, "bold"), fg="blue", relief="solid", bd=2, padx=20, pady=10)
    etiqueta_felicidades.pack(pady=50)
    
    # Función para cambiar el tamaño y color del texto de forma animada
    def animar_texto():
        for i in range(14, 22):  # Ajustar el tamaño para que no sea tan grande
            etiqueta_felicidades.config(font=("Arial", i, "bold"))
            etiqueta_felicidades.config(fg=random.choice(["red", "green", "blue", "yellow", "purple"]))
            ventana.update()
            time.sleep(0.05)  # Pausar brevemente antes de actualizar el tamaño

        # El mensaje se queda fijo después de la animación
        etiqueta_felicidades.config(font=("Arial", 22, "bold"), fg="gold")
        
        # Hacer que el mensaje se oculte después de 6 segundos
        time.sleep(6)  # Esperar 6 segundos
        etiqueta_felicidades.pack_forget()  # Ocultar el mensaje

    # Lanzar la animación en un hilo para no bloquear la interfaz
    threading.Thread(target=animar_texto).start()

# Crear un contenedor de pestañas (Notebook)
notebook = ttk.Notebook(ventana)
notebook.pack(fill="both", expand=True)

# Agregar la pestaña de portada
portada_tab = ttk.Frame(notebook)
notebook.add(portada_tab, text="Portada")

# Llamar a la función para mostrar la portada
mostrar_portada()

# Iniciar el bucle de la interfaz gráfica
ventana.mainloop()
