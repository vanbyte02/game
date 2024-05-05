import pygame
import sys

pygame.init()

screen_width, screen_height = 800, 600
screen = pygame.display.set_mode((screen_width, screen_height))

background = pygame.image.load('image/fon.png')
hero_image = pygame.image.load('image/Robot1.png')
enemy_image = pygame.image.load('image/Robot1.png')

hero = pygame.Rect(100, 500, 40, 60)
hero_speed = 5
gravity = 0.5
jump_speed = -10
hero_dy = 0
on_ground = True
bullets = []
enemies = [pygame.Rect(700, 500, 40, 60)]
obstacles = [pygame.Rect(400, 480, 50, 120)]
enemy_speeds = [1]

game_over = False
current_level = 1

def fire_bullet():
    bullet_width, bullet_height = 10, 5
    new_bullet = pygame.Rect(hero.x + hero.width, hero.y + hero.height // 2, bullet_width, bullet_height)
    bullets.append(new_bullet)

def draw_game_over():
    font = pygame.font.Font(None, 36)
    text = font.render('Game Over', True, (255, 0, 0))
    text_rect = text.get_rect(center=(screen_width / 2, screen_height / 2 - 50))
    screen.blit(text, text_rect)
    restart_text = font.render('Начать заново', True, (255, 255, 255))
    quit_text = font.render('Выйти', True, (255, 255, 255))
    restart_rect = restart_text.get_rect(center=(screen_width / 2, screen_height / 2 + 20))
    quit_rect = quit_text.get_rect(center=(screen_width / 2, screen_height / 2 + 60))
    screen.blit(restart_text, restart_rect)
    screen.blit(quit_text, quit_rect)

def load_level(level):
    global enemies, obstacles, background, enemy_speeds
    enemies.clear()  # Очистка списка врагов
    obstacles.clear()  # Очистка списка препятствий
    if level == 1:
        enemies.append(pygame.Rect(700, 500, 40, 60))
        obstacles.append(pygame.Rect(400, 480, 50, 120))
        background = pygame.image.load('image/fon.png')
        enemy_speeds = [1]
    elif level == 2:
        background = pygame.image.load('image/fon2.jpg')  # Загрузка нового фона


running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if game_over:
            if event.type == pygame.MOUSEBUTTONDOWN:
                if restart_rect and restart_rect.collidepoint(event.pos):
                    game_over = False
                    hero.x, hero.y = 100, 500
                    enemies = [pygame.Rect(700, 500, 40, 60)]
                elif quit_rect and quit_rect.collidepoint(event.pos):
                    pygame.quit()
                    sys.exit()
        if not game_over and event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE and on_ground:
                hero_dy = jump_speed
                on_ground = False
            if event.key == pygame.K_f:
                fire_bullet()

    if not game_over:
        hero_next_pos = hero.copy()
        hero_next_pos.x += hero_speed if pygame.key.get_pressed()[pygame.K_RIGHT] else -hero_speed if pygame.key.get_pressed()[pygame.K_LEFT] else 0
        hero_next_pos.y += hero_dy
        hero_dy += gravity
        if hero_next_pos.y >= 500:
            hero_next_pos.y = 500
            on_ground = True

        if hero_next_pos.x < 0 or hero_next_pos.x > screen_width:
            if current_level < 2:  # Проверяем, не последний ли это уровень
                current_level += 1
            load_level(current_level)
            hero.x, hero.y = 500, 100  # Сброс позиции героя для нового уровня

        for bullet in list(bullets):
            bullet.x += 10
            bullet_hit = False
            for obstacle in obstacles:
                if bullet.colliderect(obstacle):
                    bullets.remove(bullet)
                    bullet_hit = True
                    break
            if bullet_hit:
                continue
            for enemy in enemies:
                if bullet.colliderect(enemy):
                    enemies.remove(enemy)
                    bullets.remove(bullet)
                    break
            if bullet.x > screen_width or bullet.x < 0:
                bullets.remove(bullet)

        for index, enemy in enumerate(enemies):
            if index < len(enemy_speeds):
                enemy.x += enemy_speeds[index]
                if any(enemy.colliderect(obstacle) for obstacle in
                       obstacles) or enemy.left <= 0 or enemy.right >= screen_width:
                    enemy_speeds[index] *= -1

        if any(hero.colliderect(enemy) for enemy in enemies):
            game_over = True
            draw_game_over()

        hero.x, hero.y = hero_next_pos.x, hero_next_pos.y
        screen.blit(background, (0, 0))
        screen.blit(hero_image, hero)
        for enemy in enemies:
            screen.blit(enemy_image, enemy)
        for obstacle in obstacles:
            pygame.draw.rect(screen, (0, 0, 255), obstacle)
        for bullet in bullets:
            pygame.draw.rect(screen, (255, 0, 0), bullet)

    else:
        draw_game_over()


    pygame.display.flip()
    pygame.time.Clock().tick(120)
