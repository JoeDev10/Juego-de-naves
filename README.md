# Juego-de-naves
import pygame
import random

# Inicialización de Pygame
pygame.init()

# permite la reproducción de sonidos
pygame.mixer.init()

# Dimensiones de la pantalla
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Meteoro Wars") #titulo


imagen = pygame.image.load("galaxia.jpg")
imagen = pygame.transform.scale(imagen,(WIDTH, HEIGHT))

# Colores
WHITE = (255, 255, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)

level = 1
asteroides_destruidos = 0
asteroides_por_level = 20

# Reloj para controlar la velocidad del juego
clock = pygame.time.Clock()

def draw_text(surface, text, size, x, y):
    """
    Dibuja texto en una superficie dada utilizando los parámetros proporcionados.

    Parámetros:
        surface (pygame.Surface): La superficie en la que se dibujará el texto.
        text (str): El texto a dibujar.
        size (int): El tamaño de la fuente.
        x (int): La coordenada x de la esquina superior izquierda del texto.
        y (int): La coordenada y de la esquina superior izquierda del texto.

    Retorna:
        None
    """
    font = pygame.font.SysFont("serif", size)
    text_surface = font.render(text, True, WHITE)
    text_rect = text_surface.get_rect()
    text_rect.midtop = (x, y)
    surface.blit(text_surface, text_rect)
    
def draw_shield_bar(surface, x, y, porcentaje):
    BAR_LENGHT = 100
    BAR_HEIGHT = 10
    fill = (porcentaje / 100) * BAR_LENGHT
    border = pygame.Rect(x, y, BAR_LENGHT, BAR_HEIGHT)
    fill = pygame.Rect(x, y, fill, BAR_HEIGHT)
    pygame.draw.rect(surface, GREEN, fill)
    pygame.draw.rect(surface, WHITE, border, 2)

# Clase de la nave
class Player(pygame.sprite.Sprite):
    """
    Inicializa una nueva instancia de la clase.

    Este método se llama cuando se crea un nuevo objeto de la clase. Establece el estado inicial del objeto.

    Parámetros:
        Ninguno

    Retorna:
        Nada
    """
    def __init__(self):
        super().__init__()

        # Cargar la imagen de la nave desde un archivo
        self.image = pygame.image.load("player.png")
        self.rect = self.image.get_rect()
        self.rect.centerx = WIDTH // 2
        self.rect.bottom = HEIGHT - 10
        self.shield = 100


    def update(self):
        """
        Actualiza la posición de la nave en función de la entrada del usuario.

        Esta función se encarga de mover la nave utilizando las teclas de flecha. Comprueba el estado de las teclas de flecha y actualiza la posición de la nave en consecuencia. La posición de la nave se actualiza cambiando la coordenada x del rectángulo de la nave.

        Parámetros:
        Ninguno

        Retorna:
        Ninguno
        """
        # Mover la nave con las teclas de flecha
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            self.rect.x -= 5
        if keys[pygame.K_RIGHT]:
            self.rect.x += 5
            
        #evitamos que la nave salga de la pantalla
        if self.rect.right > WIDTH:
            self.rect.right = WIDTH
        if self.rect.left < 0:
            self.rect.left = 0
            

# Clase de asteroides
# Define una lista de posibles tamaños de asteroides (ancho, alto)
asteroid_sizes = [(30, 30), (40, 40), (50, 50)]

class Asteroid(pygame.sprite.Sprite):
    """
    Inicializa una nueva instancia de la clase Asteroid.

    Este método es el constructor de la clase Asteroid. Se llama cuando se crea una nueva instancia de la clase.

    Parámetros:
    - Ninguno

    Retorna:
    - Ninguno
    """
    def __init__(self):
        super().__init__()

        # Elige aleatoriamente un tamaño de asteroide de la lista de tamaños
        size = random.choice(asteroid_sizes)

        # Carga la imagen del asteroide según el tamaño elegido
        if size == (30, 30):
            self.image = pygame.image.load("asteroid_small.png")
        elif size == (40, 40):
            self.image = pygame.image.load("asteroid_medium.png")
        elif size == (50, 50):
            self.image = pygame.image.load("asteroid_big.png")

        self.rect = self.image.get_rect()
        self.speedy = random.randrange(1, 8)
        


    def update(self):
        """
        Actualiza la posición del asteroide.

        Esta función se encarga de mover el asteroide hacia abajo en la pantalla.
        Actualiza la coordenada y del rectángulo del asteroide sumando el valor de speedy.
        Si el asteroide desaparece de la pantalla, se posiciona de forma aleatoria en la pantalla nuevamente.

        Parámetros:
            self (Asteroid): El objeto asteroide.

        Retorna:
            None
        """
        # Hacer que los asteroides caigan hacia abajo
        self.rect.y += self.speedy
        if self.rect.top > HEIGHT:
            self.rect.x = random.randrange(0, WIDTH - self.rect.width + 1)
            self.rect.y = random.randrange(-30, -10)
            
            
#clase de proyectiles
class Proyectil(pygame.sprite.Sprite):
    """
    Inicializa una nueva instancia de la clase.

    Parámetros:
        x (int): La coordenada x del centro de la imagen.
        y (int): La coordenada y del centro de la imagen.

    Retorna:
        None
    """
    def __init__(self, x, y):
        super().__init__()
        
        self.image = pygame.image.load("proyectil.png")
        self.rect = self.image.get_rect()
        self.rect.centerx = x
        self.rect.centery = y
        self.speedy = -10
    
    def update(self):
        """
        Actualiza la posición del proyectil.

        Esta función mueve el proyectil hacia arriba en la cantidad especificada por `self.speedy`.
        Si el proyectil sale por debajo de la pantalla, se elimina del juego.

        Parámetros:
        - Ninguno

        Retorna:
        - Ninguno
        """
        #mover el proyectil hacia arriba
        self.rect.y += self.speedy
        if self.rect.bottom < 0:
            self.kill()

sonido_laser = pygame.mixer.Sound("laser.ogg")


# Crear grupos de sprites
all_sprites = pygame.sprite.Group()
asteroids = pygame.sprite.Group()
proyectiles = pygame.sprite.Group()

# Crear una nave
player = Player()
all_sprites.add(player)

# Generar asteroides
for _ in range(8):
    asteroid = Asteroid()
    all_sprites.add(asteroid)
    asteroids.add(asteroid)
    
score = 0

# Bucle principal del juego
running = True
while running:
    screen.blit(imagen, (0, 0))

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        
        # Detectar cuando se presiona la tecla de disparo (espacio)
        elif event.type == pygame.KEYDOWN:
            if  event.key == pygame.K_SPACE:
                proyectil = Proyectil(player.rect.centerx, player.rect.y)
                all_sprites.add(proyectil)
                proyectiles.add(proyectil)
                
                sonido_laser.play()
    
    # Actualizar
    all_sprites.update()
    
    hits = pygame.sprite.groupcollide(asteroids, proyectiles, True, True)
    for hit in hits:
        score += 10
        asteroid = Asteroid()
        all_sprites.add(asteroid)
        asteroids.add(asteroid)

    # Comprobar colisiones entre la nave y los asteroides
    hits = pygame.sprite.spritecollide(player, asteroids, False)
    for hit in hits:
        player.shield -= 25
        asteroid = Asteroid()
        all_sprites.add(asteroid)
        asteroids.add(asteroid)
        if player.shield <= 0:
            running = False
        
    #verificar si se ha alcanzado el proximo nivel
    if hits:
        #incrementar el contador de asteroides destruidos
        asteroides_destruidos += len(hits)
        
        if asteroides_destruidos >= asteroides_por_level:
            level += 1
            asteroides_destruidos = 0

    hits = pygame.sprite.spritecollide(player, asteroids, True)
    
    # Renderizar
    all_sprites.draw(screen)

    # Mostrar el puntaje
    draw_text(screen, str(score), 25, WIDTH // 2, 10)
    
    #escudo
    draw_shield_bar(screen, 5, 5, player.shield)

    pygame.display.flip()

    clock.tick(60)
    
pygame.quit()
