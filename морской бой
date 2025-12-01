import tkinter as tk
import random
from collections import deque

SIZE = 10
CELL = 32

# Проверка корректной постановки корабля
def can_place(board, x, y, dx, dy, size):
    for i in range(size):
        nx, ny = x + dx*i, y + dy*i
        if not (0 <= nx < SIZE and 0 <= ny < SIZE):
            return False
        if board[ny][nx] != 0:
            return False

        # Проверяем окружение
        for ax in range(nx - 1, nx + 2):
            for ay in range(ny - 1, ny + 2):
                if 0 <= ax < SIZE and 0 <= ay < SIZE:
                    if board[ay][ax] == 1:
                        return False

    return True


def place_enemy(board):
    ships = [4,3,3,2,2,2,1,1,1,1]
    for s in ships:
        placed = False
        while not placed:
            x = random.randrange(SIZE)
            y = random.randrange(SIZE)
            for dx, dy in [(1,0),(0,1)]:
                if can_place(board, x, y, dx, dy, s):
                    for i in range(s):
                        board[y+dy*i][x+dx*i] = 1
                    placed = True
                    break
    return board


# ==============================
#       ГЛАВНОЕ МЕНЮ
# ==============================
class MainMenu:
    def __init__(self):
        self.win = tk.Tk()
        self.win.title("Морской Бой — Меню")

        tk.Label(self.win, text="МОРСКОЙ БОЙ", font=("Arial", 24)).pack(pady=20)

        tk.Button(self.win, text="Играть", font=("Arial", 20),
                  command=self.start_game, width=15).pack(pady=10)

        tk.Button(self.win, text="Справка", font=("Arial", 20),
                  command=self.show_help, width=15).pack(pady=10)

        self.win.mainloop()

    def show_help(self):
        top = tk.Toplevel(self.win)
        top.title("Справка")
        msg = (
            "Правила:\n"
            "- Расставьте корабли с помощью ЛКМ.\n"
            "- Нажмите R, чтобы повернуть корабль.\n"
            "- После расстановки начнётся игра.\n"
            "- Вы и бот ходите по одному разу.\n"
            "- Побеждает тот, кто уничтожит все корабли противника."
        )
        tk.Label(top, text=msg, justify="left", font=("Arial", 14)).pack(padx=20, pady=20)

    def start_game(self):
        self.win.destroy()
        SeaBattle()


# ==============================
#       ИГРА
# ==============================
class SeaBattle:
    def __init__(self):
        self.window = tk.Tk()
        self.window.title("Морской Бой")

        # Кнопка «Главное меню»
        tk.Button(self.window, text="Главное меню", font=("Arial", 14),
                  command=self.back_to_menu).grid(row=0, column=0, columnspan=2, pady=5)

        # Доски
        self.player = [[0]*SIZE for _ in range(SIZE)]
        self.enemy  = [[0]*SIZE for _ in range(SIZE)]
        place_enemy(self.enemy)

        # Канвасы
        self.c_player = tk.Canvas(self.window, width=SIZE*CELL, height=SIZE*CELL, bg="lightblue")
        self.c_enemy  = tk.Canvas(self.window, width=SIZE*CELL, height=SIZE*CELL, bg="lightyellow")

        self.c_player.grid(row=1, column=0, padx=10, pady=10)
        self.c_enemy.grid(row=1, column=1, padx=10, pady=10)

        self.info = tk.Label(self.window,
            text="Поставьте корабль на 4 клетки. Ориентация: горизонтальная",
            font=("Arial", 14))
        self.info.grid(row=2, column=0, columnspan=2)

        # Рисуем сетки
        self.draw_grid(self.c_player)
        self.draw_grid(self.c_enemy)

        # Корабли игрока
        self.ships = [4,3,3,2,2,2,1,1,1,1]
        self.index = 0
        self.orient = "h"

        # Управление
        self.c_player.bind("<Button-1>", self.place_ship)
        self.window.bind("r", self.toggle_orient)

        # bot
        self.ai_shots = set()
        self.ai_mode = "hunt"
        self.ai_targets = deque()

        self.window.mainloop()

    # -------------------- Главное меню --------------------

    def back_to_menu(self):
        self.window.destroy()
        MainMenu()

    # -------------------- UI --------------------

    def draw_grid(self, c):
        for y in range(SIZE):
            for x in range(SIZE):
                c.create_rectangle(x*CELL, y*CELL, (x+1)*CELL, (y+1)*CELL, fill="white")

    def fill(self, canvas, x, y, color):
        canvas.create_rectangle(x*CELL, y*CELL, (x+1)*CELL, (y+1)*CELL, fill=color)

    def redraw_player(self):
        self.draw_grid(self.c_player)
        for y in range(SIZE):
            for x in range(SIZE):
                v = self.player[y][x]
                if v == 1:
                    self.fill(self.c_player, x, y, "gray")
                elif v == 2:
                    self.fill(self.c_player, x, y, "red")
                elif v == 3:
                    self.fill(self.c_player, x, y, "blue")

    # -------------------- Расстановка --------------------

    def toggle_orient(self, _):
        self.orient = "v" if self.orient == "h" else "h"
        self.info.config(
            text=f"Поставьте корабль на {self.ships[self.index]} клетки. "
                 f"Ориентация: {'верт.' if self.orient=='v' else 'гор.'}"
        )

    def place_ship(self, ev):
        x = ev.x // CELL
        y = ev.y // CELL
        size = self.ships[self.index]
        dx, dy = (1,0) if self.orient == "h" else (0,1)

        if not can_place(self.player, x, y, dx, dy, size):
            self.info.config(text="Нельзя поставить здесь!")
            return

        for i in range(size):
            self.player[y+dy*i][x+dx*i] = 1

        self.redraw_player()
        self.index += 1

        if self.index == len(self.ships):
            self.start_game()
        else:
            self.info.config(
                text=f"Поставьте корабль на {self.ships[self.index]} клетки."
            )

    # -------------------- Start --------------------

    def start_game(self):
        self.info.config(text="Игра началась! Ваш ход.")
        self.c_enemy.bind("<Button-1>", self.player_shoot)

    # -------------------- Player turn --------------------

    def player_shoot(self, ev):
        x = ev.x // CELL
        y = ev.y // CELL

        if self.enemy[y][x] in (2,3):
            return

        if self.enemy[y][x] == 1:
            self.enemy[y][x] = 2
            self.fill(self.c_enemy, x, y, "red")

            if self.check_win(self.enemy):
                self.info.config(text="Вы победили!")
                self.c_enemy.unbind("<Button-1>")
                return

        else:
            self.enemy[y][x] = 3
            self.fill(self.c_enemy, x, y, "blue")

        self.info.config(text="Ход врага...")
        self.window.after(400, self.enemy_turn)

    # -------------------- AI --------------------

    def select_hunt_cell(self):
        cells = [(x,y) for y in range(SIZE) for x in range(SIZE)
                 if (x+y)%2==0 and (x,y) not in self.ai_shots]
        if not cells:
            cells = [(x,y) for y in range(SIZE) for x in range(SIZE)
                     if (x,y) not in self.ai_shots]
        return random.choice(cells)

    def add_neighbors(self, x, y):
        for dx, dy in [(1,0),(-1,0),(0,1),(0,-1)]:
            nx, ny = x+dx, y+dy
            if 0<=nx<SIZE and 0<=ny<SIZE and (nx,ny) not in self.ai_shots:
                self.ai_targets.append((nx,ny))

    def enemy_turn(self):
        if self.ai_mode == "target" and self.ai_targets:
            x, y = self.ai_targets.popleft()
        else:
            x, y = self.select_hunt_cell()

        self.ai_shots.add((x,y))

        if self.player[y][x] == 1:
            self.player[y][x] = 2
            self.fill(self.c_player, x, y, "red")

            self.ai_mode = "target"
            self.add_neighbors(x, y)

            if self.check_win(self.player):
                self.info.config(text="Враг победил!")
                self.c_enemy.unbind("<Button-1>")
                return
        else:
            self.player[y][x] = 3
            self.fill(self.c_player, x, y, "blue")
            self.ai_mode = "hunt"
            self.ai_targets.clear()

        self.info.config(text="Ваш ход!")

    # -------------------- Win check --------------------

    def check_win(self, board):
        for row in board:
            if 1 in row:
                return False
        return True


# Запуск программы
MainMenu()
