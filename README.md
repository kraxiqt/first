import pygame
import random
import copy

pygame.init()

columns = 11
strings = 21

screen_x = 250
screen_y = 500

screen = pygame.display.set_mode((screen_x, screen_y))
pygame.display.set_caption("Tetris CODE")
clock = pygame.time.Clock()

cell_x = screen_x / (columns - 1)
cell_y = screen_y / (strings - 1)

fps = 60
grid = []

for i in range(columns):
    grid.append([])

    for j in range(strings):
       grid[i].append([1])

for i in range(columns):
   for j in range(strings):
       grid[i][j].append(pygame.Rect(i * cell_x, j * cell_y, cell_x, cell_y))
       grid[i][j].append(pygame.Color("Gray"))

# описание фигур Тетриса
details = [
   # линия
   [[-2, 0], [-1, 0], [0, 0], [1, 0]],
   # L-образная
   [[-1, 1], [-1, 0], [0, 0], [1, 0]],
   # обратная L-образная
   [[1, 1], [-1, 0], [0, 0], [1, 0]],
   # квадрат
   [[-1, 1], [0, 1], [0, 0], [-1, 0]],
   # Z-образная
   [[1, 0], [1, 1], [0, 0], [-1, 0]],
   # обратная Z-образная
   [[0, 1], [-1, 0], [0, 0], [1, 0]],
   # T-образная
   [[-1, 1], [0, 1], [0, 0], [1, 0]],
]

det = [[], [], [], [], [], [], []]

# Список цветов для фигур
colors = [
    pygame.Color("cyan"),      # линия
    pygame.Color("orange"),    # L-образная
    pygame.Color("blue"),      # обратная L-образная
    pygame.Color("yellow"),    # квадрат
    pygame.Color("red"),       # Z-образная
    pygame.Color("green"),     # обратная Z-образная
    pygame.Color("purple"),    # T-образная
]

for i in range(len(details)):
   for j in range(4):
       det[i].append(pygame.Rect(details[i][j][0] * cell_x + cell_x * (columns // 2), details[i][j][1] * cell_y, cell_x, cell_y))

detail = pygame.Rect(0, 0, cell_x, cell_y)
det_choice = copy.deepcopy(random.choice(det))

# Назначаем фигуре случайный цвет
current_color = random.choice(colors)

count = 0
game = True
rotate = False

def check_game_over():
    # Проверка на проигрыш: если хотя бы одна клетка в верхней строке занята, то игра закончена
    for i in range(columns):
        if grid[i][1][0] == 0:  # Проверяем на вторую строку, а не на первую
            return True
    return False

def check_collision():
    """Проверяем столкновение фигуры с другими блоками или нижней границей"""
    for i in range(4):
        x = int(det_choice[i].x // cell_x)
        y = int(det_choice[i].y // cell_y)

        # Проверка на границу экрана
        if x < 0 or x >= columns or y >= strings:
            return True

        # Проверка на столкновение с занятыми клетками
        if grid[x][y][0] == 0:
            return True

    return False

while game:
    delta_x = 0
    delta_y = 1

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            exit()

        if event.type == pygame.KEYDOWN:

            if event.key == pygame.K_LEFT:
                delta_x = -1

            elif event.key == pygame.K_RIGHT:
                delta_x = 1

            elif event.key == pygame.K_UP:
                rotate = True

    key = pygame.key.get_pressed()

    if key[pygame.K_DOWN]:
        count = 31 * fps

    screen.fill(pygame.Color(222, 248, 116, 100))

    # Отрисовываем сетку
    for i in range(columns):
        for j in range(strings):
            pygame.draw.rect(screen, grid[i][j][2], grid[i][j][1], grid[i][j][0])

    # Двигаем фигуру по осям
    for i in range(4):
        det_choice[i].x += delta_x * cell_x

    # Проверяем столкновение с границами и другими фигурами
    if check_collision():
        for i in range(4):
            x = int(det_choice[i].x // cell_x)
            y = int(det_choice[i].y // cell_y)

            # Если фигура столкнулась, откатываем изменения
            det_choice = copy.deepcopy(random.choice(det))
            current_color = random.choice(colors)
            break

    # Проверка на падение фигуры
    for i in range(4):
        if det_choice[i].y + cell_y >= screen_y or grid[int(det_choice[i].x // cell_x)][int(det_choice[i].y // cell_y) + 1][0] == 0:
            # Фигура приземлилась
            for i in range(4):
                x = int(det_choice[i].x // cell_x)
                y = int(det_choice[i].y // cell_y)
                grid[x][y][0] = 0
                grid[x][y][2] = current_color

            # Генерируем новую фигуру и выбираем её цвет
            det_choice = copy.deepcopy(random.choice(det))
            current_color = random.choice(colors)

    count += fps

    # Если время прошло, двигаем фигуру вниз
    if count > 30 * fps:
        for i in range(4):
            det_choice[i].y += delta_y * cell_y
        count = 0

    # Отрисовываем фигуру
    for i in range(4):
        detail.x = det_choice[i].x
        detail.y = det_choice[i].y
        pygame.draw.rect(screen, current_color, detail)

    # Вращаем фигуру
    C = det_choice[2]
    if rotate:
        for i in range(4):
            x = det_choice[i].y - C.y
            y = det_choice[i].x - C.x

            det_choice[i].x = C.x - x
            det_choice[i].y = C.y + y
        rotate = False

    # Проверка заполнения линии
    for j in range(strings - 1, -1, -1):
        count_cells = 0

        for i in range(columns):
            if grid[i][j][0] == 0:
                count_cells += 1
            elif grid[i][j][0] == 1:
                break

        if count_cells == (columns - 1):
            for l in range(columns):
                grid[l][0][0] = 1
            for k in range(j, -1, -1):
                for l in range(columns):
                    grid[l][k][0] = grid[l][k - 1][0]

    # Проверка на проигрыш
    if check_game_over():
        game = False
        # Печатаем "Game Over"
        font = pygame.font.SysFont("Arial", 36)
        text = font.render("GAME OVER!", True, pygame.Color("Red"))
        screen.blit(text, (screen_x // 6, screen_y // 2))

    pygame.display.flip()
    clock.tick(fps)

pygame.quit()
