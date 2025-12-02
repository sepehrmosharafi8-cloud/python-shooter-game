# python-shooter-game
shooter-game
import pygame
import random
import sys

pygame.init()

# Window
WIDTH = 800
HEIGHT = 600
WIN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Python Shooter Game")

# Colors
WHITE = (255, 255, 255)
RED = (255, 60, 60)
BLUE = (80, 200, 255)
YELLOW = (255, 255, 120)
BLACK = (0, 0, 0)

# Player
player_width = 50
player_height = 50
player_x = WIDTH // 2 - player_width // 2
player_y = HEIGHT - player_height - 20
player_speed = 5

# Lists
bullets = []
enemies = []

# Game variables
score = 0
lives = 3
enemy_spawn_timer = 0
GAME_OVER = False

clock = pygame.time.Clock()


def draw_player():
    pygame.draw.rect(WIN, BLUE, (player_x, player_y, player_width, player_height))


def draw_bullet(b):
    pygame.draw.rect(WIN, YELLOW, (b[0], b[1], 6, 12))


def draw_enemy(e):
    pygame.draw.rect(WIN, RED, (e[0], e[1], e[2], e[2]))


def spawn_enemy():
    size = random.randint(30, 50)
    x = random.randint(0, WIDTH - size)
    speed = random.uniform(1.5, 3.5)
    enemies.append([x, -size, size, speed])


def reset_game():
    global bullets, enemies, score, lives, GAME_OVER
    bullets = []
    enemies = []
    score = 0
    lives = 3
    GAME_OVER = False


def draw_game():
    WIN.fill(BLACK)

    # Player
    draw_player()

    # Bullets
    for b in bullets:
        draw_bullet(b)

    # Enemies
    for e in enemies:
        draw_enemy(e)

    # Score & lives
    font = pygame.font.SysFont(None, 28)
    score_text = font.render(f"Score: {score}", True, WHITE)
    lives_text = font.render(f"Lives: {lives}", True, WHITE)

    WIN.blit(score_text, (10, 10))
    WIN.blit(lives_text, (10, 40))

    pygame.display.update()


def draw_game_over():
    WIN.fill((20, 20, 20))
    font = pygame.font.SysFont(None, 50)
    small_font = pygame.font.SysFont(None, 30)

    t1 = font.render("GAME OVER", True, RED)
    t2 = small_font.render(f"Final Score: {score}", True, WHITE)
    t3 = small_font.render("Press R to Restart", True, WHITE)

    WIN.blit(t1, (WIDTH//2 - t1.get_width()//2, HEIGHT//2 - 50))
    WIN.blit(t2, (WIDTH//2 - t2.get_width()//2, HEIGHT//2))
    WIN.blit(t3, (WIDTH//2 - t3.get_width()//2, HEIGHT//2 + 40))

    pygame.display.update()


# Game Loop
running = True
while running:
    clock.tick(60)

    # Game Over Screen
    if GAME_OVER:
        draw_game_over()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN and event.key == pygame.K_r:
                reset_game()
        continue

    # Normal Game Mode
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # Player movement
    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT] and player_x > 0:
        player_x -= player_speed
    if keys[pygame.K_RIGHT] and player_x + player_width < WIDTH:
        player_x += player_speed

    # Shooting
    if keys[pygame.K_SPACE]:
        if len(bullets) == 0 or bullets[-1][1] < player_y - 40:
            bullets.append([player_x + player_width//2 - 3, player_y])

    # Move bullets
    for b in bullets:
        b[1] -= 8
    bullets = [b for b in bullets if b[1] > -10]

    # Spawn enemies
    enemy_spawn_timer += 1
    if enemy_spawn_timer > 40:
        spawn_enemy()
        enemy_spawn_timer = 0

    # Move enemies
    for e in enemies:
        e[1] += e[3]

    # Bullet–enemy collision
    for b in bullets:
        for e in enemies:
            if (b[0] < e[0] + e[2] and b[0] + 6 > e[0] and
                    b[1] < e[1] + e[2] and b[1] + 12 > e[1]):
                score += 1
                bullets.remove(b)
                enemies.remove(e)
                break

    # Enemy–player or enemy leaving screen
    for e in enemies:
        if e[1] > HEIGHT:
            enemies.remove(e)
            lives -= 1

        # Collision with player
        if (player_x < e[0] + e[2] and player_x + player_width > e[0] and
                player_y < e[1] + e[2] and player_y + player_height > e[1]):
            lives -= 1
            enemies.remove(e)

    if lives <= 0:
        GAME_OVER = True

    draw_game()

pygame.quit()
sys.exit()
