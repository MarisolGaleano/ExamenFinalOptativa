import pygame
import random
import time

# Inicializar Pygame
pygame.init()

# Configurar la pantalla
ANCHO = 800
ALTO = 600
pantalla = pygame.display.set_mode((ANCHO, ALTO))
pygame.display.set_caption("POU")

# Colores
NEGRO = (0, 0, 0)
BLANCO = (255, 255, 255)
ROJO = (255, 0, 0)
AZUL = (0, 0, 255)
VERDE = (0, 255, 0)

# Cargar imágenes
jugador_img = pygame.image.load("jugador.png")
objetivo_img = pygame.image.load("objetivo.png")
fondo_img = pygame.image.load("fondo.png")
penalizador_img = pygame.image.load("penalizador.png")

# Escalar imágenes al tamaño adecuado
jugador_tam = 130
objetivo_tam = 60
penalizador_tam = 60
jugador_img = pygame.transform.scale(jugador_img, (jugador_tam, jugador_tam))
objetivo_img = pygame.transform.scale(objetivo_img, (objetivo_tam, objetivo_tam))
penalizador_img = pygame.transform.scale(penalizador_img, (penalizador_tam, penalizador_tam))
fondo_img = pygame.transform.scale(fondo_img, (ANCHO, ALTO))

# Fuente para texto
fuente = pygame.font.Font(None, 36)
fuente_grande = pygame.font.Font(None, 72)

# Configurar el jugador y objetos
def reiniciar_juego():
    global jugador_x, jugador_y, jugador_vel, objetivos, penalizadores, puntuacion, penalizaciones, tiempo_inicial, juego_terminado, juego_iniciado
    jugador_x = ANCHO // 3
    jugador_y = ALTO // 3
    jugador_vel = 5
    objetivos = []
    penalizadores = []
    puntuacion = 0
    penalizaciones = 0  # Contador de penalizaciones
    tiempo_inicial = time.time()
    juego_terminado = False
    juego_iniciado = True

    # Crear lluvia de objetivos y penalizadores
    for _ in range(5):  # Número de objetivos
        x = random.randint(0, ANCHO - objetivo_tam)
        y = random.randint(-ALTO, -objetivo_tam)  # Iniciar fuera de la pantalla
        objetivos.append([x, y])

    for _ in range(2):  # Número de penalizadores
        x = random.randint(0, ANCHO - penalizador_tam)
        y = random.randint(-ALTO, -penalizador_tam)  # Iniciar fuera de la pantalla
        penalizadores.append([x, y])

def pantalla_inicio():
    pantalla.blit(fondo_img, (0, 0))
    texto_inicio = fuente_grande.render("POU", True, AZUL)
    pantalla.blit(texto_inicio, (ANCHO // 2 - texto_inicio.get_width() // 2, ALTO // 2 - 100))
    
    boton_inicio_rect = pygame.Rect(ANCHO // 2 - 100, ALTO // 2, 200, 50)
    pygame.draw.rect(pantalla, VERDE, boton_inicio_rect)
    texto_boton_inicio = fuente.render("Iniciar Juego", True, NEGRO)
    pantalla.blit(texto_boton_inicio, (boton_inicio_rect.x + 25, boton_inicio_rect.y + 10))
    
    pygame.display.flip()
    return boton_inicio_rect

# Estado inicial del juego
juego_iniciado = False
juego_terminado = False

# Reloj para controlar la velocidad de actualización
reloj = pygame.time.Clock()

# Bucle principal del juego
ejecutando = True
while ejecutando:
    for evento in pygame.event.get():
        if evento.type == pygame.QUIT:
            ejecutando = False
        if evento.type == pygame.MOUSEBUTTONDOWN:
            mouse_x, mouse_y = evento.pos
            if not juego_iniciado:
                if boton_inicio_rect.collidepoint(mouse_x, mouse_y):
                    reiniciar_juego()
            elif juego_terminado:
                if boton_rect.collidepoint(mouse_x, mouse_y):
                    reiniciar_juego()

    if not juego_iniciado:
        boton_inicio_rect = pantalla_inicio()
    elif not juego_terminado:
        # Obtener las teclas presionadas
        teclas = pygame.key.get_pressed()
        if teclas[pygame.K_LEFT]:
            jugador_x -= jugador_vel
        if teclas[pygame.K_RIGHT]:
            jugador_x += jugador_vel
        if teclas[pygame.K_UP]:
            jugador_y -= jugador_vel
        if teclas[pygame.K_DOWN]:
            jugador_y += jugador_vel

        # Limitar el movimiento del jugador dentro de la pantalla
        if jugador_x < 0:
            jugador_x = 0
        if jugador_x > ANCHO - jugador_tam:
            jugador_x = ANCHO - jugador_tam
        if jugador_y < 0:
            jugador_y = 0
        if jugador_y > ALTO - jugador_tam:
            jugador_y = ALTO - jugador_tam

        # Mover objetivos y penalizadores
        for obj in objetivos:
            obj[1] += 5  # Velocidad de caída
            if obj[1] > ALTO:
                obj[0] = random.randint(0, ANCHO - objetivo_tam)
                obj[1] = random.randint(-ALTO, -objetivo_tam)  # Reiniciar fuera de la pantalla

        for pen in penalizadores:
            pen[1] += 5  # Velocidad de caída
            if pen[1] > ALTO:
                pen[0] = random.randint(0, ANCHO - penalizador_tam)
                pen[1] = random.randint(-ALTO, -penalizador_tam)  # Reiniciar fuera de la pantalla

        # Detectar colisiones
        jugador_rect = pygame.Rect(jugador_x, jugador_y, jugador_tam, jugador_tam)
        for obj in objetivos:
            objetivo_rect = pygame.Rect(obj[0], obj[1], objetivo_tam, objetivo_tam)
            if jugador_rect.colliderect(objetivo_rect):
                obj[0] = random.randint(0, ANCHO - objetivo_tam)
                obj[1] = random.randint(-ALTO, -objetivo_tam)
                puntuacion += 1
                jugador_vel += 0.5  # Aumentar la velocidad del jugador

        for pen in penalizadores:
            penalizador_rect = pygame.Rect(pen[0], pen[1], penalizador_tam, penalizador_tam)
            if jugador_rect.colliderect(penalizador_rect):
                penalizadores.remove(pen)
                penalizaciones += 1
                if penalizaciones >= 3:  # El juego termina en la tercera penalización
                    juego_terminado = True
                else:
                    puntuacion -= 1  # Reducir la puntuación
                    # Volver a añadir el penalizador a la lista
                    x = random.randint(0, ANCHO - penalizador_tam)
                    y = random.randint(-ALTO, -penalizador_tam)
                    penalizadores.append([x, y])

        # Dibujar la pantalla
        pantalla.blit(fondo_img, (0, 0))
        pantalla.blit(jugador_img, (jugador_x, jugador_y))
        for obj in objetivos:
            pantalla.blit(objetivo_img, (obj[0], obj[1]))
        for pen in penalizadores:
            pantalla.blit(penalizador_img, (pen[0], pen[1]))

        # Dibujar la puntuación
        texto_puntuacion = fuente.render(f"Puntuación: {puntuacion}", True, BLANCO)
        pantalla.blit(texto_puntuacion, (10, 10))

        # Calcular el tiempo restante
        tiempo_transcurrido = time.time() - tiempo_inicial
        tiempo_restante = max(0, 40 - int(tiempo_transcurrido))
        texto_tiempo = fuente.render(f"Tiempo: {tiempo_restante}", True, BLANCO)
        pantalla.blit(texto_tiempo, (ANCHO - 150, 10))

        # Verificar si el tiempo se ha agotado
        if tiempo_restante <= 0:
            juego_terminado = True

        pygame.display.flip()

        # Controlar la velocidad de actualización
        reloj.tick(30)
    else:
        pantalla.fill(NEGRO)
        texto_fin = fuente_grande.render("¡Juego Terminado!", True, ROJO)
        pantalla.blit(texto_fin, (ANCHO // 2 - texto_fin.get_width() // 2, ALTO // 2 - 100))
        
        texto_puntuacion_final = fuente.render(f"Puntuación: {puntuacion}", True, BLANCO)
        pantalla.blit(texto_puntuacion_final, (ANCHO // 2 - texto_puntuacion_final.get_width() // 2, ALTO // 2 - 20))
        
        boton_rect = pygame.Rect(ANCHO // 2 - 100, ALTO // 2 + 50, 200, 50)
        pygame.draw.rect(pantalla, VERDE, boton_rect)
        texto_boton = fuente.render("Reiniciar", True, NEGRO)
        pantalla.blit(texto_boton, (boton_rect.x + 50, boton_rect.y + 10))

        pygame.display.flip()

pygame.quit()
