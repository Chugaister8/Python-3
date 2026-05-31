# ЧАСТИНА I — ОСНОВИ PYTHON

# Python від нуля
## *Середовище · Типи · ООП · Файли · Бази даних*

**Рівень:** Початківець → Junior
**Тривалість:** 14–18 тижнів · Модулі 0–4

---

# МОДУЛЬ 0 — Старт: Підготовка та Середовище

## Розділ 0.1 — Алгоритмічне мислення

### § 0.1.1 Що таке програмування

Програмування — це мистецтво давати точні інструкції комп'ютеру. Комп'ютер виконує рівно те, що ви написали, тому головна навичка — думати точно та послідовно.

**Декомпозиція задачі:** будь-яку велику задачу розбивайте на маленькі кроки. Задача «зробити сайт» розбивається на: спроектувати БД → написати API → написати UI → задеплоїти.

### § 0.1.2 Де застосовується Python

| Область | Приклади |
|---------|---------|
| Веб-розробка | Instagram, Pinterest, Reddit (Django/Flask) |
| Data Science | Jupyter, Pandas, Matplotlib |
| Machine Learning | TensorFlow, PyTorch, scikit-learn |
| Комп'ютерний зір | OpenCV, YOLO, face_recognition |
| Автоматизація | скрипти, парсери, боти |
| Desktop | Tkinter, PyQt |
| DevOps/MLOps | Ansible, Airflow, Prefect |

---

## Розділ 0.2 — Встановлення та середовище

### § 0.2.1 Python та pyenv

Замість прямого встановлення Python рекомендується **pyenv** — менеджер версій.

```bash
# macOS / Linux — встановити pyenv
curl https://pyenv.run | bash

# Встановити Python
pyenv install 3.12.3
pyenv global 3.12.3

# Перевірка
python --version    # Python 3.12.3
```

> ⚠️ **Увага (Windows):** На Windows використовуйте офіційний інсталятор з python.org. Обов'язково поставте галочку "Add Python to PATH".

### § 0.2.2 Віртуальні середовища

**Це найважливіша звичка Python-розробника.** Завжди — для кожного проєкту — окреме venv.

```bash
python -m venv venv
source venv/bin/activate      # Linux/macOS
venv\Scripts\activate         # Windows

pip install package_name
pip freeze > requirements.txt
pip install -r requirements.txt
deactivate
```

> 💡 **Порада:** Сучасна альтернатива — **uv** (ультра-швидкий менеджер пакетів від Astral). `pip install uv` → `uv venv` → `uv pip install`.

### § 0.2.3 VS Code та розширення

| Розширення | Для чого |
|-----------|---------|
| Python (Microsoft) | Інтелект, дебаггер, тести |
| Pylance | Статичний аналіз типів |
| Black Formatter | Автоформатування |
| Ruff | Швидкий лінтер (замінює flake8+isort) |
| GitLens | Розширена git-інтеграція |
| Thunder Client | REST API клієнт |
| SQLite Viewer | Перегляд SQLite |
| Docker | Управління контейнерами |

### § 0.2.4 Git та pre-commit хуки

```bash
git init
git config --global user.name "Ваше Ім'я"
git config --global user.email "email@example.com"

# .gitignore для Python
echo "venv/\n__pycache__/\n*.pyc\n.env\ndist/\n.coverage\nhtmlcov/" > .gitignore
```

```yaml
# .pre-commit-config.yaml — автоматичні перевірки перед кожним комітом
repos:
  - repo: https://github.com/psf/black
    rev: 24.4.2
    hooks:
      - id: black
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.4
    hooks:
      - id: ruff
        args: [--fix]
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.10.0
    hooks:
      - id: mypy
```

```bash
pip install pre-commit
pre-commit install      # хуки встановлено
pre-commit run --all-files  # перевірити всі файли вручну
```

| 📋 **Завдання 0 — Налаштування** |
|---|
| 1. Встановіть Python 3.12+, VS Code з розширеннями, Git |
| 2. Створіть папку `python-course/`, ініціалізуйте git та venv |
| 3. Налаштуйте `.pre-commit-config.yaml` з black та ruff |
| 4. Напишіть `hello.py` що виводить ваше ім'я та рік |
| 5. Перший коміт: `feat: init project` |
| ⭐ **Бонус:** Встановіть `uv` та ознайомтесь з його командами |

---

# МОДУЛЬ 1 — Основи Python

## Розділ 1.1 — Змінні та типи даних

### § 1.1.1 Базові типи

```python
# str
name: str = "Олексій"
greeting = f"Привіт, {name}!"          # f-string
multiline = """рядок
на кілька рядків"""

# int / float / complex
age: int = 25
price: float = 19.99
big: int = 10_000_000                  # підкреслення для читабельності

# bool
is_admin: bool = True
has_access: bool = False

# None
result = None

# Перевірка типу
print(type(name))       # <class 'str'>
print(isinstance(age, int))  # True
```

### § 1.1.2 Рядки та їх методи

```python
text = "  Привіт, Світе!  "

# Основні методи
text.strip()            # "Привіт, Світе!"
text.lower()            # "  привіт, світе!  "
text.upper()
text.replace("Світе", "Python")
text.split(",")         # ['  Привіт', ' Світе!  ']
text.startswith("  П")  # True
"Python" in text        # False

# f-рядки — завжди використовуйте замість .format()
name, age = "Іванко", 20
print(f"Мені {age} років, через рік буде {age + 1}")
print(f"Ціна: {19.9876:.2f} грн")      # форматування чисел
print(f"{'текст':>20}")                # вирівнювання
print(f"{name!r}")                      # repr()
```

### § 1.1.3 Числа та математика

```python
# Арифметика
10 / 3      # 3.3333 (float завжди!)
10 // 3     # 3 (ціле ділення)
10 % 3      # 1 (залишок)
2 ** 10     # 1024 (степінь)

import math
math.sqrt(16)       # 4.0
math.ceil(3.2)      # 4
math.floor(3.8)     # 3
math.log(100, 10)   # 2.0
round(3.14159, 2)   # 3.14
abs(-42)            # 42
```

---

## Розділ 1.2 — Колекції

### § 1.2.1 Список (list)

```python
nums = [3, 1, 4, 1, 5, 9, 2, 6]

# Зріз
nums[0]         # 3
nums[-1]        # 6
nums[1:4]       # [1, 4, 1]
nums[::2]       # [3, 4, 5, 2]  — кожен другий
nums[::-1]      # реверс

# Методи
nums.append(7)
nums.insert(0, 0)
nums.remove(1)          # перше входження
nums.pop()              # останній
nums.sort()
nums.sort(reverse=True)
sorted_copy = sorted(nums)

# List comprehension
squares = [x**2 for x in range(10)]
evens   = [x for x in range(20) if x % 2 == 0]
matrix  = [[i*j for j in range(1,4)] for i in range(1,4)]
```

### § 1.2.2 Словник (dict)

```python
user = {
    "name": "Олена",
    "age": 28,
    "skills": ["Python", "SQL"],
}

# Доступ
user["name"]                    # "Олена"
user.get("phone", "невідомо")   # безпечний доступ

# Зміна
user["city"] = "Київ"
user.setdefault("role", "viewer")  # встановити лише якщо немає

# Ітерація
for key, val in user.items():
    print(f"{key}: {val}")

# Dict comprehension
sq = {i: i**2 for i in range(1, 6)}

# Злиття словників (Python 3.9+)
defaults = {"role": "viewer", "active": True}
merged = defaults | user        # новий словник
defaults |= user                # оновити на місці
```

### § 1.2.3 Множина (set) та кортеж (tuple)

```python
# set — унікальні елементи
fruits = {"яблуко", "груша", "яблуко"}  # {"яблуко", "груша"}
a, b = {1, 2, 3, 4}, {3, 4, 5, 6}
a & b   # {3, 4}  перетин
a | b   # {1,2,3,4,5,6}  об'єднання
a - b   # {1, 2}  різниця
a ^ b   # {1, 2, 5, 6}  симетрична різниця

# tuple — незмінний список
point = (10, 20)
x, y = point            # розпакування
rgb = (255, 128, 0)

# namedtuple (з бібліотеки collections)
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])
p = Point(10, 20)
print(p.x, p.y)         # 10 20
```

### § 1.2.4 Корисні структури з collections

```python
from collections import defaultdict, Counter, deque, OrderedDict

# Counter
words = ["apple", "banana", "apple", "cherry", "apple"]
c = Counter(words)
print(c.most_common(2))     # [('apple', 3), ('banana', 1)]

# defaultdict — не кидає KeyError
graph = defaultdict(list)
graph["A"].append("B")
graph["A"].append("C")

# deque — ефективна черга з обох кінців
q = deque([1, 2, 3])
q.appendleft(0)     # O(1)
q.popleft()         # O(1), на відміну від list.pop(0) = O(n)

# deque як кільцевий буфер
last_5 = deque(maxlen=5)
for i in range(10):
    last_5.append(i)
print(list(last_5))  # [5, 6, 7, 8, 9]
```

---

## Розділ 1.3 — Функції

### § 1.3.1 Оголошення та параметри

```python
def greet(name: str, greeting: str = "Привіт") -> str:
    """Вітає користувача.

    Args:
        name: Ім'я користувача.
        greeting: Текст вітання.

    Returns:
        Рядок вітання.
    """
    return f"{greeting}, {name}!"

# *args та **kwargs
def log(level: str, *messages: str, sep: str = " | ") -> None:
    print(f"[{level.upper()}] {sep.join(messages)}")

log("info", "Запуск", "Успішно")
log("error", "Помилка", "Деталі", sep=" >> ")

# Keyword-only аргументи (після *)
def create_user(name: str, *, role: str = "viewer", active: bool = True):
    pass
create_user("Іван", role="admin")  # OK
# create_user("Іван", "admin")    # TypeError — role тільки keyword
```

### § 1.3.2 Декоратори

```python
import functools
import time
from typing import Callable, TypeVar, ParamSpec

P = ParamSpec("P")
T = TypeVar("T")

def timer(func: Callable[P, T]) -> Callable[P, T]:
    """Вимірює час виконання функції."""
    @functools.wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__}: {elapsed:.4f}с")
        return result
    return wrapper

def retry(times: int = 3, delay: float = 1.0, exceptions: tuple = (Exception,)):
    """Повторює виклик при виключенні."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == times:
                        raise
                    print(f"Спроба {attempt}/{times} невдала: {e}. Повтор через {delay}с")
                    time.sleep(delay)
        return wrapper
    return decorator

@timer
@retry(times=3, delay=0.5, exceptions=(ConnectionError,))
def fetch_data(url: str) -> dict:
    # ...
    pass
```

### § 1.3.3 Генератори та ітератори

```python
# Генератор — функція з yield
def fibonacci(n: int):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

# Не займає пам'ять на всі n чисел
for num in fibonacci(10):
    print(num, end=" ")

# Generator expression
squares_gen = (x**2 for x in range(1_000_000))  # ліниво

# Нескінченний генератор
def counter(start: int = 0, step: int = 1):
    current = start
    while True:
        yield current
        current += step

import itertools
# itertools.islice — взяти перші N з нескінченного
first_10_odds = list(itertools.islice(counter(1, 2), 10))
print(first_10_odds)  # [1, 3, 5, 7, 9, 11, 13, 15, 17, 19]

# send() — двостороннє спілкування з генератором
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is None:
            break
        total += value

acc = accumulator()
next(acc)           # запустити
acc.send(10)        # 10
acc.send(20)        # 30
acc.send(15)        # 45
```

---

## Розділ 1.4 — ООП

### § 1.4.1 Класи та магічні методи

```python
from __future__ import annotations
from dataclasses import dataclass, field
from typing import ClassVar

class BankAccount:
    _instances: ClassVar[int] = 0  # атрибут класу

    def __init__(self, owner: str, balance: float = 0.0) -> None:
        self.owner = owner
        self.__balance = balance          # приватний
        BankAccount._instances += 1

    # Магічні методи
    def __repr__(self) -> str:
        return f"BankAccount(owner={self.owner!r}, balance={self.__balance})"

    def __str__(self) -> str:
        return f"Рахунок {self.owner}: {self.__balance:.2f} грн"

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, BankAccount):
            return NotImplemented
        return self.owner == other.owner

    def __lt__(self, other: BankAccount) -> bool:
        return self.__balance < other.__balance

    def __add__(self, other: BankAccount) -> BankAccount:
        return BankAccount(f"{self.owner}+{other.owner}",
                           self.__balance + other.__balance)

    # Property
    @property
    def balance(self) -> float:
        return self.__balance

    @balance.setter
    def balance(self, value: float) -> None:
        if value < 0:
            raise ValueError("Баланс не може бути від'ємним")
        self.__balance = value

    # Метод класу — альтернативний конструктор
    @classmethod
    def from_dict(cls, data: dict) -> BankAccount:
        return cls(data["owner"], data.get("balance", 0))

    # Статичний метод — утиліта без стану
    @staticmethod
    def validate_amount(amount: float) -> bool:
        return amount > 0


# dataclass — автоматична генерація __init__, __repr__, __eq__
@dataclass(order=True)
class Point:
    x: float
    y: float
    label: str = ""
    _id: int = field(default_factory=lambda: id(object()), repr=False, compare=False)

    def distance_to(self, other: Point) -> float:
        return ((self.x - other.x)**2 + (self.y - other.y)**2) ** 0.5

p1 = Point(0, 0, "Origin")
p2 = Point(3, 4)
print(p1.distance_to(p2))  # 5.0
print(p1 < p2)             # True (order=True — порівнює x, потім y)
```

### § 1.4.2 Наслідування та поліморфізм

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    color: str = "black"

    def __init__(self, color: str = "black") -> None:
        self.color = color

    @abstractmethod
    def area(self) -> float: ...

    @abstractmethod
    def perimeter(self) -> float: ...

    def describe(self) -> str:
        return (f"{type(self).__name__}: колір={self.color}, "
                f"площа={self.area():.2f}, периметр={self.perimeter():.2f}")


class Circle(Shape):
    import math as _math

    def __init__(self, radius: float, color: str = "black") -> None:
        super().__init__(color)
        self.radius = radius

    def area(self) -> float:
        return self._math.pi * self.radius ** 2

    def perimeter(self) -> float:
        return 2 * self._math.pi * self.radius


class Rectangle(Shape):
    def __init__(self, width: float, height: float, color: str = "black") -> None:
        super().__init__(color)
        self.width = width
        self.height = height

    def area(self) -> float:
        return self.width * self.height

    def perimeter(self) -> float:
        return 2 * (self.width + self.height)


# Поліморфізм
shapes: list[Shape] = [Circle(5, "red"), Rectangle(4, 6, "blue"), Circle(3)]
for shape in shapes:
    print(shape.describe())

total_area = sum(s.area() for s in shapes)
```

### § 1.4.3 Протоколи та Duck Typing

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> str: ...
    def resize(self, factor: float) -> None: ...

# Будь-який клас з методами draw() і resize() задовольняє протокол
class Icon:
    def draw(self) -> str:
        return "Малюю іконку"
    def resize(self, factor: float) -> None:
        print(f"Масштаб: x{factor}")

def render(item: Drawable) -> None:
    print(item.draw())

render(Icon())  # Працює — Icon реалізує протокол
print(isinstance(Icon(), Drawable))  # True — завдяки @runtime_checkable
```

---

## Розділ 1.5 — Обробка помилок

```python
class AppError(Exception):
    """Базовий клас для помилок застосунку."""
    def __init__(self, message: str, code: int = 500) -> None:
        super().__init__(message)
        self.code = code

class NotFoundError(AppError):
    def __init__(self, resource: str, identifier) -> None:
        super().__init__(f"{resource} '{identifier}' не знайдено", code=404)

class ValidationError(AppError):
    def __init__(self, field: str, message: str) -> None:
        super().__init__(f"Поле '{field}': {message}", code=422)
        self.field = field

# ExceptionGroup (Python 3.11+)
def validate_form(data: dict) -> None:
    errors = []
    if not data.get("name"):
        errors.append(ValidationError("name", "обов'язкове"))
    if not data.get("email"):
        errors.append(ValidationError("email", "обов'язкове"))
    if errors:
        raise ExceptionGroup("Помилки валідації", errors)

try:
    validate_form({})
except* ValidationError as eg:
    for exc in eg.exceptions:
        print(f"  {exc}")
```

---

## Розділ 1.6 — Context Managers

```python
from contextlib import contextmanager, asynccontextmanager
import time

# Через клас
class Timer:
    def __enter__(self) -> Timer:
        self._start = time.perf_counter()
        return self

    def __exit__(self, *args) -> bool:
        self.elapsed = time.perf_counter() - self._start
        print(f"Час: {self.elapsed:.4f}с")
        return False  # не пригнічуємо виключення

with Timer() as t:
    sum(range(1_000_000))
# Час: 0.0234с

# Через contextlib
@contextmanager
def temp_file(suffix: str = ".tmp"):
    import tempfile, os
    path = tempfile.mktemp(suffix=suffix)
    try:
        yield path
    finally:
        if os.path.exists(path):
            os.remove(path)

with temp_file(".json") as path:
    with open(path, "w") as f:
        f.write('{"key": "value"}')
    # файл видалиться автоматично
```

---

## Модуль 2 — Модулі, файли та серіалізація

### § 2.1 Структура пакетів

```
mypackage/
├── __init__.py
├── py.typed              # маркер для mypy
├── models/
│   ├── __init__.py
│   └── user.py
├── services/
│   ├── __init__.py
│   └── email.py
└── utils/
    ├── __init__.py
    ├── strings.py
    └── dates.py
```

```python
# __init__.py — контролюйте публічний API пакету
from .models.user import User
from .services.email import EmailService

__all__ = ["User", "EmailService"]
__version__ = "1.0.0"
```

### § 2.2 Файли та pathlib

```python
from pathlib import Path

# pathlib — сучасний спосіб роботи з шляхами
base = Path(__file__).parent
data_dir = base / "data"
config = base / "config.json"

# Перевірки
config.exists()         # True/False
config.is_file()
data_dir.is_dir()

# Читання / запис
text = config.read_text(encoding="utf-8")
config.write_text('{"key": "val"}', encoding="utf-8")

# Обхід директорії
for py_file in base.rglob("*.py"):
    print(py_file.relative_to(base))

# Файли через open()
with open(data_dir / "users.txt", "w", encoding="utf-8") as f:
    f.writelines(f"{line}\n" for line in ["Іван", "Марія", "Петро"])
```

### § 2.3 JSON, CSV, TOML

```python
import json
from pathlib import Path

# JSON
data = {"name": "Іван", "tags": ["python", "django"]}
Path("user.json").write_text(
    json.dumps(data, ensure_ascii=False, indent=2),
    encoding="utf-8"
)
loaded = json.loads(Path("user.json").read_text(encoding="utf-8"))

# CSV з csv.DictWriter
import csv
users = [
    {"name": "Іван", "age": 25, "city": "Київ"},
    {"name": "Марія", "age": 30, "city": "Львів"},
]
with open("users.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age", "city"])
    writer.writeheader()
    writer.writerows(users)

# TOML (Python 3.11+, вбудований)
import tomllib
with open("pyproject.toml", "rb") as f:
    config = tomllib.load(f)
```

### § 2.4 Серіалізація з Pickle та Protocol Buffers

```python
import pickle

# Pickle — для Python-об'єктів (НЕ для зовнішніх даних!)
data = {"model": [1.0, 2.5, 3.7], "version": 2}
with open("model.pkl", "wb") as f:
    pickle.dump(data, f, protocol=pickle.HIGHEST_PROTOCOL)

with open("model.pkl", "rb") as f:
    loaded = pickle.load(f)

# 🔒 БЕЗПЕКА: ніколи не завантажуйте pickle з ненадійних джерел!
# Pickle може виконати довільний код при десеріалізації.
```

---

## Модуль 3 — Бази даних

### § 3.1 SQLite

```python
import sqlite3
from contextlib import contextmanager
from typing import Generator

@contextmanager
def get_db(path: str = "app.db") -> Generator[sqlite3.Connection, None, None]:
    conn = sqlite3.connect(path)
    conn.row_factory = sqlite3.Row
    conn.execute("PRAGMA foreign_keys = ON")
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()

def init_db() -> None:
    with get_db() as conn:
        conn.executescript("""
            CREATE TABLE IF NOT EXISTS users (
                id      INTEGER PRIMARY KEY AUTOINCREMENT,
                name    TEXT    NOT NULL,
                email   TEXT    UNIQUE NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
            CREATE TABLE IF NOT EXISTS posts (
                id      INTEGER PRIMARY KEY AUTOINCREMENT,
                title   TEXT NOT NULL,
                content TEXT,
                user_id INTEGER REFERENCES users(id) ON DELETE CASCADE
            );
        """)
```

### § 3.2 SQLAlchemy 2.x (сучасний стиль)

```python
from sqlalchemy import create_engine, String, Integer, ForeignKey, select, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship, Session
from datetime import datetime
from typing import Optional

engine = create_engine("postgresql+psycopg2://user:pass@localhost/db", echo=False)

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100))
    email: Mapped[str] = mapped_column(String(150), unique=True)
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)

    posts: Mapped[list["Post"]] = relationship(back_populates="author", cascade="all, delete-orphan")

    def __repr__(self) -> str:
        return f"User(id={self.id}, email={self.email!r})"

class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(200))
    content: Mapped[Optional[str]]
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))

    author: Mapped[User] = relationship(back_populates="posts")

Base.metadata.create_all(engine)

# Сучасний стиль запитів (SQLAlchemy 2.x)
with Session(engine) as session:
    # Додати
    user = User(name="Іван", email="ivan@example.com")
    session.add(user)
    session.flush()  # отримати id без commit

    # Знайти
    stmt = select(User).where(User.email == "ivan@example.com")
    user = session.scalars(stmt).one()

    # Агрегація
    count = session.scalar(select(func.count()).select_from(User))

    session.commit()
```

### § 3.3 Alembic — міграції

```bash
pip install alembic
alembic init alembic           # ініціалізація
alembic revision --autogenerate -m "add users table"
alembic upgrade head           # застосувати
alembic downgrade -1           # відкотити один крок
alembic history                # переглянути історію
```

```python
# alembic/env.py — налаштування
from models import Base
target_metadata = Base.metadata
```

| 📋 **Завдання 1 — Бібліотека з БД** |
|---|
| Реалізуйте систему бібліотеки: |
| 1. Моделі SQLAlchemy 2.x: `Book`, `Member`, `Borrow` |
| 2. Репозиторії: `BookRepo`, `MemberRepo`, `BorrowRepo` з повним CRUD |
| 3. Alembic міграції |
| 4. Серіалізація у JSON (backup/restore) |
| 5. pytest тести з in-memory SQLite |
| ⭐ **Бонус:** Повнотекстовий пошук через SQLite FTS5 |

---

## Підсумок Частини I

| Навичка | Де використовується |
|---------|-------------------|
| Всі типи даних та колекції | Скрізь |
| ООП, протоколи, dataclass | Flask, Django, FastAPI |
| Декоратори та генератори | Всі фреймворки |
| SQLAlchemy 2.x + Alembic | Flask, FastAPI |
| pathlib, JSON, CSV | Скрипти, ETL |
| Обробка виключень | Всі частини |

> 🔗 **Далі:** Частина II — Типізація та якість коду

---

## Розділ 1.5 — Enum, itertools, functools, dataclasses (поглиблено)

### § 1.5.1 Enum — типізовані константи

```python
from enum import Enum, auto, IntEnum, StrEnum, Flag, unique
from typing import ClassVar

# ─── Базовий Enum ────────────────────────────────────────────────────────────
class Color(Enum):
    RED   = 1
    GREEN = 2
    BLUE  = 3

c = Color.RED
print(c)          # Color.RED
print(c.name)     # "RED"
print(c.value)    # 1
print(c is Color.RED)   # True

# ─── auto() — автоматична нумерація ──────────────────────────────────────────
class Direction(Enum):
    NORTH = auto()   # 1
    SOUTH = auto()   # 2
    EAST  = auto()   # 3
    WEST  = auto()   # 4

# ─── IntEnum / StrEnum — сумісні з int / str ─────────────────────────────────
class Status(IntEnum):
    PENDING  = 0
    ACTIVE   = 1
    BANNED   = 2

# Порівняння з int (IntEnum дає сумісність)
print(Status.ACTIVE == 1)   # True
print(Status.ACTIVE > Status.PENDING)  # True

class Priority(StrEnum):
    LOW    = "low"
    MEDIUM = "medium"
    HIGH   = "high"

# Порівняння з рядком
print(Priority.HIGH == "high")   # True
print(f"Пріоритет: {Priority.HIGH}")   # "Пріоритет: high"

# ─── @unique — заборонити дублікати значень ───────────────────────────────────
@unique
class HttpMethod(StrEnum):
    GET    = "GET"
    POST   = "POST"
    PUT    = "PUT"
    PATCH  = "PATCH"
    DELETE = "DELETE"

# ─── Flag — бітові маски (комбінації прав) ───────────────────────────────────
class Permission(Flag):
    READ    = auto()
    WRITE   = auto()
    DELETE  = auto()
    ADMIN   = READ | WRITE | DELETE   # комбінація

user_perms = Permission.READ | Permission.WRITE
print(Permission.READ in user_perms)    # True
print(Permission.DELETE in user_perms)  # False
print(user_perms)                       # Permission.READ|WRITE

# ─── Методи у Enum ────────────────────────────────────────────────────────────
class Planet(Enum):
    MERCURY = (3.303e+23, 2.4397e6)
    VENUS   = (4.869e+24, 6.0518e6)
    EARTH   = (5.976e+24, 6.37814e6)

    def __init__(self, mass: float, radius: float) -> None:
        self.mass   = mass
        self.radius = radius

    @property
    def surface_gravity(self) -> float:
        G = 6.67430e-11
        return G * self.mass / (self.radius ** 2)

print(f"Земля: {Planet.EARTH.surface_gravity:.2f} m/s²")   # 9.80 m/s²

# ─── Ітерація та пошук ────────────────────────────────────────────────────────
for s in Status:
    print(f"{s.name}: {s.value}")

# З рядка
status = Status["ACTIVE"]     # Status.ACTIVE
status = Status(1)            # Status.ACTIVE

# ─── Django TextChoices / IntegerChoices (Django 3.0+) ───────────────────────
from django.db import models

class OrderStatus(models.TextChoices):
    DRAFT     = "draft",     "Чернетка"
    CONFIRMED = "confirmed", "Підтверджено"
    SHIPPED   = "shipped",   "Відправлено"
    DELIVERED = "delivered", "Доставлено"
    CANCELLED = "cancelled", "Скасовано"

class Order(models.Model):
    status = models.CharField(
        max_length=20,
        choices=OrderStatus.choices,
        default=OrderStatus.DRAFT,
    )

# Використання
order = Order(status=OrderStatus.CONFIRMED)
print(order.get_status_display())   # "Підтверджено"
print(OrderStatus.CONFIRMED.label)  # "Підтверджено"
```

---

### § 1.5.2 itertools — ефективна робота з ітераторами

```python
import itertools
from collections.abc import Iterable, Iterator
from typing import TypeVar

T = TypeVar("T")

# ─── Нескінченні ітератори ────────────────────────────────────────────────────
# count: 0, 1, 2, 3, ...
for i in itertools.islice(itertools.count(10, 2), 5):
    print(i)   # 10, 12, 14, 16, 18

# cycle: a, b, c, a, b, c, ...
colors = list(itertools.islice(itertools.cycle(["red", "green", "blue"]), 7))
# ['red', 'green', 'blue', 'red', 'green', 'blue', 'red']

# repeat: 42, 42, 42 (або нескінченно)
list(itertools.repeat(0, 5))   # [0, 0, 0, 0, 0]

# ─── Комбінаторні ────────────────────────────────────────────────────────────
# product: декартовий добуток
list(itertools.product("AB", repeat=2))
# [('A','A'), ('A','B'), ('B','A'), ('B','B')]

list(itertools.product([1, 2], [3, 4]))
# [(1,3), (1,4), (2,3), (2,4)]

# combinations: C(n,k) без повторень
list(itertools.combinations("ABCD", 2))
# [('A','B'), ('A','C'), ('A','D'), ('B','C'), ('B','D'), ('C','D')]

# permutations: P(n,k)
list(itertools.permutations("ABC", 2))
# [('A','B'), ('A','C'), ('B','A'), ('B','C'), ('C','A'), ('C','B')]

# combinations_with_replacement: з повтореннями
list(itertools.combinations_with_replacement("AB", 2))
# [('A','A'), ('A','B'), ('B','B')]

# ─── Об'єднання та фільтрація ────────────────────────────────────────────────
# chain: об'єднати кілька ітераторів
list(itertools.chain([1, 2], [3, 4], [5]))   # [1, 2, 3, 4, 5]
list(itertools.chain.from_iterable([[1,2],[3,4]]))  # flatten на один рівень

# takewhile / dropwhile
list(itertools.takewhile(lambda x: x < 5, [1,2,3,4,5,6,1]))  # [1,2,3,4]
list(itertools.dropwhile(lambda x: x < 5, [1,2,3,4,5,6,1]))  # [5,6,1]

# compress: фільтр за бітовою маскою
list(itertools.compress("ABCDE", [1,0,1,0,1]))   # ['A', 'C', 'E']

# ─── Групування ──────────────────────────────────────────────────────────────
# groupby: ВИМАГАЄ попереднього сортування!
data = [
    {"dept": "IT",  "name": "Іван"},
    {"dept": "IT",  "name": "Марія"},
    {"dept": "HR",  "name": "Петро"},
    {"dept": "HR",  "name": "Оля"},
]
data.sort(key=lambda x: x["dept"])   # обов'язково!

for dept, group in itertools.groupby(data, key=lambda x: x["dept"]):
    members = [p["name"] for p in group]
    print(f"{dept}: {members}")
# HR: ['Петро', 'Оля']
# IT: ['Іван', 'Марія']

# ─── Накопичення ─────────────────────────────────────────────────────────────
# accumulate: накопичувальна операція
import operator
list(itertools.accumulate([1,2,3,4,5]))                      # [1,3,6,10,15]
list(itertools.accumulate([1,2,3,4,5], operator.mul))        # [1,2,6,24,120]
list(itertools.accumulate([3,1,4,1,5,9], max))               # running max

# ─── Практичний приклад: батчінг ─────────────────────────────────────────────
def batched(iterable: Iterable[T], n: int) -> Iterator[tuple[T, ...]]:
    """Розбити ітератор на chunks по n елементів. (вбудований у 3.12+)"""
    it = iter(iterable)
    while chunk := tuple(itertools.islice(it, n)):
        yield chunk

for batch in batched(range(10), 3):
    print(batch)
# (0, 1, 2), (3, 4, 5), (6, 7, 8), (9,)

# Python 3.12+ має вбудований itertools.batched
```

---

### § 1.5.3 functools — функціональні інструменти

```python
import functools
from typing import TypeVar, Callable, Any

T = TypeVar("T")

# ─── reduce ──────────────────────────────────────────────────────────────────
from functools import reduce
import operator

# sum через reduce
reduce(operator.add, [1, 2, 3, 4, 5])          # 15
reduce(operator.mul, [1, 2, 3, 4, 5], 1)       # 120

# Flatten через reduce
nested = [[1,2],[3,4],[5]]
reduce(operator.iadd, nested, [])               # [1,2,3,4,5]

# ─── partial — часткове застосування ─────────────────────────────────────────
from functools import partial

def power(base: float, exp: float) -> float:
    return base ** exp

square = partial(power, exp=2)
cube   = partial(power, exp=3)
print(square(5))   # 25
print(cube(3))     # 27

# Практично: логер з префіксом
import logging
log_error = partial(logging.log, logging.ERROR)
log_info  = partial(logging.log, logging.INFO)

# ─── singledispatch — перевантаження функцій ─────────────────────────────────
from functools import singledispatch

@singledispatch
def serialize(value: Any) -> str:
    return str(value)

@serialize.register(int)
def _(value: int) -> str:
    return f"{value:,}"   # 1,234,567

@serialize.register(float)
def _(value: float) -> str:
    return f"{value:.2f}"

@serialize.register(list)
def _(value: list) -> str:
    return "[" + ", ".join(serialize(v) for v in value) + "]"

print(serialize(1234567))         # "1,234,567"
print(serialize(3.14159))         # "3.14"
print(serialize([1, 2.5, "x"]))  # "[1, 2.50, x]"

# ─── cache / lru_cache ────────────────────────────────────────────────────────
from functools import cache, lru_cache

@cache   # необмежений (Python 3.9+)
def fib(n: int) -> int:
    return n if n < 2 else fib(n-1) + fib(n-2)

@lru_cache(maxsize=128)
def get_user_perms(user_id: int) -> frozenset[str]:
    # дорогий запит до БД — виконається тільки раз на user_id
    return frozenset(db.get_permissions(user_id))

# Статистика кешу
print(fib.cache_info())   # CacheInfo(hits=..., misses=..., maxsize=None, currsize=...)
fib.cache_clear()

# ─── total_ordering — автодоповнення методів порівняння ──────────────────────
from functools import total_ordering

@total_ordering
class Version:
    def __init__(self, major: int, minor: int, patch: int) -> None:
        self.major = major
        self.minor = minor
        self.patch = patch

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Version):
            return NotImplemented
        return (self.major, self.minor, self.patch) == (other.major, other.minor, other.patch)

    def __lt__(self, other: "Version") -> bool:
        return (self.major, self.minor, self.patch) < (other.major, other.minor, other.patch)

    # total_ordering автоматично додає __le__, __gt__, __ge__

v1 = Version(1, 2, 3)
v2 = Version(1, 3, 0)
print(v1 < v2)    # True
print(v1 <= v2)   # True (автоматично!)
print(v2 > v1)    # True (автоматично!)
```

---

### § 1.5.4 dataclasses — повний арсенал

```python
from dataclasses import dataclass, field, fields, asdict, astuple, replace, KW_ONLY
from typing import ClassVar, InitVar

# ─── field() повністю ────────────────────────────────────────────────────────
@dataclass
class Config:
    host:     str  = "localhost"
    port:     int  = 8080
    # default_factory — для мутабельних дефолтів
    tags:     list[str] = field(default_factory=list)
    # repr=False — не показувати у __repr__
    password: str = field(default="", repr=False)
    # compare=False — не враховувати при ==
    _id:      int = field(default=0, compare=False, repr=False)
    # init=False — не передавати в __init__, встановлюється в __post_init__
    created:  str = field(default="", init=False)

    def __post_init__(self) -> None:
        import datetime
        self.created = datetime.datetime.utcnow().isoformat()
        if self.port < 1 or self.port > 65535:
            raise ValueError(f"Invalid port: {self.port}")

# ─── KW_ONLY — тільки keyword аргументи (Python 3.10+) ───────────────────────
@dataclass
class Point3D:
    x: float
    y: float
    _: KW_ONLY          # всі після цього — тільки keyword
    z: float = 0.0
    label: str = ""

p = Point3D(1.0, 2.0, z=3.0, label="origin")
# Point3D(1.0, 2.0) — теж OK, z та label мають дефолти

# ─── InitVar — аргумент тільки для __init__ ───────────────────────────────────
@dataclass
class HashedPassword:
    username: str
    raw_password: InitVar[str]        # не зберігається як атрибут
    password_hash: str = field(init=False)

    def __post_init__(self, raw_password: str) -> None:
        import hashlib
        self.password_hash = hashlib.sha256(raw_password.encode()).hexdigest()

u = HashedPassword("ivan", "secret123")
print(u.password_hash)   # sha256 hash
# u.raw_password  — AttributeError: немає такого атрибута

# ─── frozen=True — незмінний (hashable) ──────────────────────────────────────
@dataclass(frozen=True)
class Coordinate:
    lat: float
    lng: float

    def distance_to(self, other: "Coordinate") -> float:
        return ((self.lat - other.lat)**2 + (self.lng - other.lng)**2) ** 0.5

coord = Coordinate(50.45, 30.52)
# coord.lat = 51.0  → FrozenInstanceError!

# Frozen dataclass можна використовувати як ключ словника або в set
locations = {Coordinate(50.45, 30.52), Coordinate(49.84, 24.03)}

# ─── slots=True — Python 3.10+ ────────────────────────────────────────────────
@dataclass(slots=True)   # __slots__ автоматично — менше пам'яті
class Sensor:
    device_id: str
    value:     float
    timestamp: float

# ─── replace() — копія з зміненими полями ────────────────────────────────────
original = Config(host="db.local", port=5432)
updated  = replace(original, port=5433)
print(original.port)  # 5432
print(updated.port)   # 5433

# ─── asdict / astuple / fields ────────────────────────────────────────────────
d = asdict(original)    # {"host": "db.local", "port": 5432, ...}
t = astuple(original)   # ("db.local", 5432, ...)

for f in fields(Config):
    print(f.name, f.type, f.default)

# ─── ClassVar — не поле, не в __init__ ────────────────────────────────────────
@dataclass
class Counter:
    _count: ClassVar[int] = 0   # спільна для всіх екземплярів

    name: str

    def __post_init__(self) -> None:
        Counter._count += 1

    @classmethod
    def total(cls) -> int:
        return cls._count
```

---

### § 1.5.5 Timezone-aware datetime

```python
from datetime import datetime, timezone, timedelta
from zoneinfo import ZoneInfo   # Python 3.9+ (вбудований)

# ─── UTC-aware datetime ───────────────────────────────────────────────────────
now_utc = datetime.now(timezone.utc)
print(now_utc)   # 2025-01-15 14:30:00+00:00

# ─── Конкретна timezone ───────────────────────────────────────────────────────
kyiv = ZoneInfo("Europe/Kyiv")
now_kyiv = datetime.now(kyiv)
print(now_kyiv)  # 2025-01-15 16:30:00+02:00

# ─── Конвертація між timezone ────────────────────────────────────────────────
ny    = ZoneInfo("America/New_York")
tokyo = ZoneInfo("Asia/Tokyo")

event = datetime(2025, 6, 15, 10, 0, tzinfo=kyiv)   # 10:00 Kyiv
print(event.astimezone(ny))     # 03:00 New York
print(event.astimezone(tokyo))  # 16:00 Tokyo

# ─── Naive vs Aware ──────────────────────────────────────────────────────────
naive  = datetime(2025, 1, 15, 10, 0)          # ❌ немає timezone
aware  = datetime(2025, 1, 15, 10, 0, tzinfo=timezone.utc)   # ✅

# Перетворити naive → aware (якщо знаємо, що це UTC)
aware = naive.replace(tzinfo=timezone.utc)

# ─── Django: USE_TZ = True ────────────────────────────────────────────────────
# settings.py
USE_TZ = True
TIME_ZONE = "Europe/Kyiv"

# В коді завжди використовувати django.utils.timezone
from django.utils import timezone as dj_tz

now = dj_tz.now()                        # UTC-aware datetime
local_now = dj_tz.localtime(now)         # конвертувати до TIME_ZONE
local_now = dj_tz.localtime(now, ZoneInfo("America/New_York"))  # інша TZ

# ⚠️ Ніколи не використовувати datetime.datetime.now() в Django з USE_TZ!
# datetime.datetime.now(tz=dj_tz.utc)  ← правильно

# ─── Список доступних timezone ───────────────────────────────────────────────
import zoneinfo
zones = sorted(zoneinfo.available_timezones())
ukraine_zones = [z for z in zones if "Kiev" in z or "Kyiv" in z]
```

📋 **Завдання 1.5** — Закріплення:
1. Написати `Permission` Flag Enum з правами READ/WRITE/DELETE/ADMIN і функцію `check_access(user_perms, required)`.
2. Реалізувати `pipeline(*funcs)` через `functools.reduce` — послідовне застосування функцій.
3. Написати `@dataclass(frozen=True, slots=True) Point` і покласти 1000 екземплярів у set — порівняти пам'ять з звичайним class.
4. Написати `format_schedule(events)` що групує події по даті через `itertools.groupby`.
