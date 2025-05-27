import random


class BoardException(Exception):
    pass


class BoardOutException(BoardException):
    def __str__(self):
        return "Выстрел за пределы доски!"


class UsedPointException(BoardException):
    def __str__(self):
        return "Вы уже стреляли в эту клетку!"


class WrongShipPositionException(BoardException):
    pass


class Dot:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __repr__(self):
        return f"({self.x}, {self.y})"


class Ship:
    def __init__(self, bow, length, orientation):
        self.bow = bow
        self.length = length
        self.orientation = orientation
        self.lives = length

    @property
    def dots(self):
        ship_dots = []
        for i in range(self.length):
            cur_x = self.bow.x
            cur_y = self.bow.y
            if self.orientation == 0:
                cur_y += i
            else:
                cur_x += i
            ship_dots.append(Dot(cur_x, cur_y))
        return ship_dots


class Board:
    def __init__(self, hid=False):
        self.grid = [["О"] * 6 for _ in range(6)]
        self.ships = []
        self.hid = hid
        self.busy_dots = set()
        self.alive_ships = 0

    def add_ship(self, ship):
        for d in ship.dots:
            if self.out(d) or d in self.busy_dots:
                raise WrongShipPositionException()

        for d in ship.dots:
            self.grid[d.x][d.y] = "■"
            self.busy_dots.add(d)

        self.ships.append(ship)
        self.alive_ships += 1
        self.contour(ship)

    def contour(self, ship):
        around = [(-1, -1), (-1, 0), (-1, 1),
                  (0, -1), (0, 0), (0, 1),
                  (1, -1), (1, 0), (1, 1)]

        for d in ship.dots:
            for dx, dy in around:
                cur = Dot(d.x + dx, d.y + dy)
                if not self.out(cur):
                    self.busy_dots.add(cur)

    def out(self, dot):
        return not (0 <= dot.x < 6 and 0 <= dot.y < 6)

    def __str__(self):
        res = "  | 1 | 2 | 3 | 4 | 5 | 6 |\n"
        for i, row in enumerate(self.grid):
            res += f"{i + 1} | " + " | ".join(row) + " |\n"
        if self.hid:
            res = res.replace("■", "О")
        return res

    def shot(self, dot):
        if self.out(dot):
            raise BoardOutException()
        if dot in self.busy_dots:
            raise UsedPointException()

        self.busy_dots.add(dot)
        for ship in self.ships:
            if dot in ship.dots:
                ship.lives -= 1
                self.grid[dot.x][dot.y] = "X"
                if ship.lives == 0:
                    self.alive_ships -= 1
                    print("Корабль уничтожен!")
                else:
                    print("Корабль ранен!")
                return True

        self.grid[dot.x][dot.y] = "T"
        print("Промах!")
        return False


class Player:
    def __init__(self, board, enemy):
        self.board = board
        self.enemy = enemy

    def ask(self):
        raise NotImplementedError()

    def move(self):
        while True:
            try:
                target = self.ask()
                return self.enemy.shot(target)
            except BoardException as e:
                print(e)


class AI(Player):
    def ask(self):
        d = Dot(random.randint(0, 5), random.randint(0, 5))
        print(f"Ход компьютера: {d.x + 1} {d.y + 1}")
        return d


class User(Player):
    def ask(self):
        while True:
            coords = input("Ваш ход (формат: x y): ").split()
            if len(coords) != 2:
                print("Введите 2 координаты!")
                continue
            x, y = coords
            if not (x.isdigit()) or not (y.isdigit()):
                print("Введите числа!")
                continue
            return Dot(int(x) - 1, int(y) - 1)


class Game:
    def __init__(self):
        self.user_board = self.random_board()
        self.ai_board = self.random_board()
        self.ai_board.hid = True
        self.user = User(self.user_board, self.ai_board)
        self.ai = AI(self.ai_board, self.user_board)

    def random_board(self):
        board = None
        while board is None:
            board = self.try_generate()
        return board

    def try_generate(self):
        ships = [3, 2, 2, 1, 1, 1, 1]
        board = Board()
        for length in ships:
            while True:
                ship = Ship(
                    bow=Dot(random.randint(0, 5), random.randint(0, 5)),
                    length=length,
                    orientation=random.randint(0, 1)
                )
                try:
                    board.add_ship(ship)
                    break
                except WrongShipPositionException:
                    pass
        return board

    def greet(self):
        print("-------------------")
        print("   Морской бой")
        print("-------------------")
        print(" Формат ввода: x y")
        print(" x - номер строки")
        print(" y - номер столбца")

    def loop(self):
        num = 0
        while True:
            print("-" * 20)
            print("Доска пользователя:")
            print(self.user_board)
            print("Доска компьютера:")
            print(self.ai_board)

            if num % 2 == 0:
                print("Ваш ход!")
                repeat = self.user.move()
            else:
                print("Ход компьютера!")
                repeat = self.ai.move()

            if not repeat:
                num += 1

            if self.ai_board.alive_ships == 0:
                print("-" * 20)
                print("Вы выиграли!")
                break

            if self.user_board.alive_ships == 0:
                print("-" * 20)
                print("Компьютер выиграл!")
                break

    def start(self):
        self.greet()
        self.loop()

class Dot:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __hash__(self):
        return hash((self.x, self.y))

    def __repr__(self):
        return f"({self.x}, {self.y})"


if __name__ == "__main__":
    g = Game()
    g.start()
