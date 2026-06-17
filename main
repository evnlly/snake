"""Игра Изгиб Питона."""

import sys
import random
from typing import Optional

import pygame


CELL_SIZE = 20

GRID_WIDTH = 32
GRID_HEIGHT = 24

SCREEN_W = CELL_SIZE * GRID_WIDTH
SCREEN_H = CELL_SIZE * GRID_HEIGHT

SCREEN_CENTER = (SCREEN_W // 2, SCREEN_H // 2)

BG_COLOR = (0, 0, 0)

UP = (0, -CELL_SIZE)
DOWN = (0, CELL_SIZE)
LEFT = (-CELL_SIZE, 0)
RIGHT = (CELL_SIZE, 0)

OPPOSITE = {UP: DOWN, DOWN: UP, LEFT: RIGHT, RIGHT: LEFT}

GAME_SPEED = 20

pygame.init()
screen = pygame.display.set_mode((SCREEN_W, SCREEN_H))
pygame.display.set_caption("Изгиб Питона")
clock = pygame.time.Clock()


class GameObject:
    """Базовый класс.

    Здесь только позиция и цвет — больше ничего общего у объектов нет.
    """

    def __init__(self, position: tuple = SCREEN_CENTER,
                 body_color: tuple = (255, 255, 255)) -> None:
        """Запоминаем позицию и цвет.

        Args:
            position: координаты верхнего левого угла ячейки (x, y).
            body_color: цвет в формате RGB.
        """
        self.position = position
        self.body_color = body_color

    def draw(self) -> None:
        """Отрисовка объекта — переопределяется в дочерних классах."""
        pass


class Apple(GameObject):
    """Яблоко появляется в случайном месте и ждёт, пока змейка его съест.

    Как только съели — перемещается в новую случайную клетку.
    """

    APPLE_COLOR = (220, 20, 60)

    def __init__(self, occupied: Optional[list] = None) -> None:
        """Создаём яблоко и ставим его на свободную клетку."""
        super().__init__(body_color=self.APPLE_COLOR)
        self.randomize_position(occupied)

    def randomize_position(self, occupied: Optional[list] = None) -> None:
        """Выбираем случайную клетку, которая не занята телом змейки.

        Args:
            occupied: список позиций тела змейки — туда яблоко не ставим.
        """
        occupied = occupied or []
        while True:
            col = random.randint(0, GRID_WIDTH - 1)
            row = random.randint(0, GRID_HEIGHT - 1)
            candidate = (col * CELL_SIZE, row * CELL_SIZE)
            if candidate not in occupied:
                self.position = candidate
                break

    def draw(self) -> None:
        """Рисуем яблоко: закрашенный квадрат с тонкой чёрной рамкой."""
        rect = pygame.Rect(self.position, (CELL_SIZE, CELL_SIZE))
        pygame.draw.rect(screen, self.body_color, rect)
        pygame.draw.rect(screen, BG_COLOR, rect, 1)


class Snake(GameObject):
    """Змейка.

    Тело хранится списком координат: первый элемент — голова,
    последний — хвост. Каждый ход добавляем новую голову спереди
    и убираем хвост сзади — так змейка и движется.
    """

    SNAKE_COLOR = (50, 205, 50)

    def __init__(self) -> None:
        """Стартуем с одной головой посередине экрана, едем вправо."""
        super().__init__(position=SCREEN_CENTER, body_color=self.SNAKE_COLOR)
        self.length: int = 1
        self.positions: list[tuple] = [SCREEN_CENTER]
        self.direction: tuple = RIGHT
        self.next_direction: tuple | None = None
        self._last_tail: tuple | None = None

    def get_head_position(self) -> tuple:
        """Возвращает координаты головы змейки."""
        return self.positions[0]

    def update_direction(self) -> None:
        """Применяем новое направление, если игрок что-то нажал."""
        if self.next_direction is not None:
            self.direction = self.next_direction
            self.next_direction = None

    def move(self) -> None:
        """Двигаем змейку на одну клетку вперёд."""
        hx, hy = self.get_head_position()
        dx, dy = self.direction

        # остаток от деления даёт "телепортацию" через стену
        new_head = (
            (hx + dx) % SCREEN_W,
            (hy + dy) % SCREEN_H,
        )
        self.positions.insert(0, new_head)

        if len(self.positions) > self.length:
            self._last_tail = self.positions.pop()
        else:
            self._last_tail = None

    def reset(self) -> None:
        """Сбрасываем всё в начало, если змейка врезалась в себя."""
        self.length = 1
        self.positions = [SCREEN_CENTER]
        self.direction = RIGHT
        self.next_direction = None
        self._last_tail = None
        screen.fill(BG_COLOR)

    def draw(self) -> None:
        """Рисуем голову змейки и стираем след от хвоста."""
        # новая голова
        head_rect = pygame.Rect(
            self.get_head_position(), (CELL_SIZE, CELL_SIZE)
        )
        pygame.draw.rect(screen, self.body_color, head_rect)
        pygame.draw.rect(screen, BG_COLOR, head_rect, 1)

        # стираем хвост, если он остался
        if self._last_tail is not None:
            tail_rect = pygame.Rect(self._last_tail, (CELL_SIZE, CELL_SIZE))
            pygame.draw.rect(screen, BG_COLOR, tail_rect)


def handle_keys(snake: Snake) -> None:
    """Читаем события pygame и меняем направление змейки."""
    key_to_dir = {
        pygame.K_UP: UP,
        pygame.K_w: UP,
        pygame.K_DOWN: DOWN,
        pygame.K_s: DOWN,
        pygame.K_LEFT: LEFT,
        pygame.K_a: LEFT,
        pygame.K_RIGHT: RIGHT,
        pygame.K_d: RIGHT,
    }

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN:
            chosen = key_to_dir.get(event.key)
            # игнорируем нажатие, если это разворот назад
            if chosen and chosen != OPPOSITE.get(snake.direction):
                snake.next_direction = chosen


def main() -> None:
    """Создаём объекты и запускаем игровой цикл."""
    snake = Snake()
    apple = Apple(occupied=snake.positions)

    screen.fill(BG_COLOR)

    while True:
        clock.tick(GAME_SPEED)

        handle_keys(snake)
        snake.update_direction()
        snake.move()

        # голова совпала с яблоком — съели
        if snake.get_head_position() == apple.position:
            snake.length += 1
            apple.randomize_position(occupied=snake.positions)

        # голова попала на тело — начинаем заново
        head = snake.get_head_position()
        if head in snake.positions[1:]:
            snake.reset()
            apple.randomize_position()

        apple.draw()
        snake.draw()
        pygame.display.update()


if __name__ == "__main__":
    main()