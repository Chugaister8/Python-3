# Розділ 18. Графічний інтерфейс користувача

Упродовж усієї книги ви створювали застосунки командного рядка — програми, що запускаються з терміналу і виводять результат у термінальному вікні.

Застосунки командного рядка добре підходять для утиліт, якими можуть користуватися розробники, але переважна більшість кінцевих користувачів ніколи не захочуть відкривати термінал!

Графічні інтерфейси користувача, що скорочено називаються GUI («гуі»), мають вікна з такими компонентами, як кнопки та текстові поля.

У цьому розділі ви навчитеся:

- додавати простий GUI до застосунку командного рядка за допомогою EasyGUI;
- створювати повнофункціональні GUI-застосунки за допомогою Tkinter.

## 18.1 Додавання GUI-елементів за допомогою EasyGUI

```
$ pip3 install easygui
```

EasyGUI чудово підходить для відображення діалогових вікон. Можна уявляти EasyGUI як заміну функцій input() і print().

Типовий потік програми: відображається візуальний елемент → виконання призупиняється → користувач надає введення → виконання відновлюється.

```python
>>> import easygui as gui
>>> gui.msgbox(msg="Hello!", title="My first message box")
'OK'
```

`msgbox()` повертає мітку кнопки. Якщо вікно закривається без натискання кнопки, повертається `None`.

### Набір GUI-елементів EasyGUI

| Функція | Опис |
|---------|------|
| `msgbox()` | Повідомлення з однією кнопкою. Повертає мітку кнопки. |
| `buttonbox()` | Кілька кнопок. Повертає мітку вибраної кнопки. |
| `indexbox()` | Кілька кнопок. Повертає індекс вибраної кнопки. |
| `enterbox()` | Поле введення тексту. Повертає введений текст. |
| `fileopenbox()` | Вибір файлу для відкриття. Повертає шлях до файлу. |
| `diropenbox()` | Вибір каталогу. Повертає шлях до каталогу. |
| `filesavebox()` | Збереження файлу. Повертає шлях для збереження. |

**buttonbox():**

```python
>>> gui.buttonbox(
...     msg="What is your favorite color?",
...     title="Choose wisely...",
...     choices=("Red", "Yellow", "Blue"),
... )
'Yellow'
```

**indexbox():** повертає індекс вибраної кнопки:

```python
>>> colors = ("Red", "Yellow", "Blue")
>>> choice = gui.indexbox(
...     msg="What's your favorite color?",
...     title="Favorite color",
...     choices=colors,
... )
>>> colors[choice]
'Yellow'
```

**enterbox():** для збору текстового введення:

```python
>>> gui.enterbox(
...     msg="What is your favorite color?",
...     title="Favorite color",
... )
'Yellow'
```

> **Важливо**
>
> `fileopenbox()` фактично не відкриває файл — він лише повертає шлях. Для відкриття файлу потрібно використовувати вбудовану функцію `open()`.

Якщо користувач закриває діалогове вікно без вибору, функції повертають `None`. Неуважне ставлення до цього може призвести до аварійного завершення програми.

### Коректне завершення програми

```python
import easygui as gui

path = gui.fileopenbox(title="Select a file")
if path is None:
    exit()
```

---

**Завдання для повторення**

Розв'язання цих завдань та інші бонусні матеріали можна знайти онлайн на realpython.com/python-basics/resources.

1. Створіть діалогове вікно з трьома кнопками-кольорами.
2. Створіть діалогове вікно для вибору файлу та відобразіть шлях.

---

## 18.2 Приклад застосунку: Обертач сторінок PDF

Повний код застосунку для обертання сторінок PDF:

```python
import easygui as gui
from PyPDF2 import PdfFileReader, PdfFileWriter

# 1. Вибрати PDF для відкриття.
input_path = gui.fileopenbox(
    title="Select a PDF to rotate...",
    default="*.pdf"
)

# 2. Якщо скасували — завершити.
if input_path is None:
    exit()

# 3. Обрати кут повороту.
choices = ("90", "180", "270")
degrees = gui.buttonbox(
    msg="Rotate the PDF clockwise by how many degrees?",
    title="Choose rotation...",
    choices=choices,
)
degrees = int(degrees)

# 4. Вибрати місце збереження.
save_title = "Save the rotated PDF as..."
file_type = "*.pdf"
output_path = gui.filesavebox(title=save_title, default=file_type)

# 5. Якщо намагаються зберегти з тим самим ім'ям — попередити.
while input_path == output_path:
    gui.msgbox(msg="Cannot overwrite original file!")
    output_path = gui.filesavebox(title=save_title, default=file_type)

# 6. Якщо скасували збереження — завершити.
if output_path is None:
    exit()

# 7. Виконати обертання.
input_file = PdfFileReader(input_path)
output_pdf = PdfFileWriter()

for page in input_file.pages:
    page = page.rotateClockwise(degrees)
    output_pdf.addPage(page)

with open(output_path, "wb") as output_file:
    output_pdf.write(output_file)
```

---

**Завдання для повторення**

1. Програма аварійно завершується, якщо користувач закриває `buttonbox()` для вибору градусів. Виправте це за допомогою циклу `while`.

---

## 18.3 Задача: Застосунок для витягування сторінок PDF

За допомогою EasyGUI напишіть GUI-застосунок, що:

1. Просить вибрати PDF-файл.
2. Запитує початковий і кінцевий номери сторінок.
3. Перевіряє коректність введених номерів.
4. Зберігає витягнуті сторінки у новий файл.

Розв'язання цієї задачі та інші бонусні матеріали можна знайти онлайн на realpython.com/python-basics/resources.

## 18.4 Вступ до Tkinter

Tkinter — єдиний GUI-фреймворк, що вбудований у стандартну бібліотеку Python. Він є кросплатформним: той самий код працює на Windows, macOS та Linux.

> **Примітка**
>
> IDLE написаний на Tkinter. Якщо GUI-вікно несподівано зависає в IDLE, спробуйте запустити скрипт із командного рядка.

### Ваш перший застосунок Tkinter

```python
>>> import tkinter as tk
>>> window = tk.Tk()
>>> greeting = tk.Label(text="Hello, Tkinter")
>>> greeting.pack()
>>> window.mainloop()
```

`window.mainloop()` запускає цикл подій і блокує код, що слідує після нього, до закриття вікна.

> **Важливо**
>
> Якщо `window.mainloop()` не включено в кінці програми у файлі Python, застосунок Tkinter ніколи не запуститься.

---

**Завдання для повторення**

1. Створіть вікно з `Label` із текстом `"GUIs are great!"`.

---

## 18.5 Робота з віджетами

| Клас | Опис |
|------|------|
| `Label` | Відображення тексту. |
| `Button` | Кнопка з дією при натисканні. |
| `Entry` | Однорядкове поле введення тексту. |
| `Text` | Багаторядкове поле введення тексту. |
| `Frame` | Область для групування віджетів. |

### Віджети Label

```python
label = tk.Label(
    text="Hello, Tkinter",
    fg="white",    # foreground
    bg="black",    # background
    width=10,
    height=10
)
```

Ширина і висота вимірюються в текстових одиницях. Одна горизонтальна одиниця — ширина символу `"0"` у шрифті за замовчуванням.

### Віджети Button

```python
button = tk.Button(
    text="Click me!",
    width=25,
    height=5,
    bg="blue",
    fg="yellow",
)
```

### Віджети Entry

Три основні операції:

```python
# Отримати текст
name = entry.get()

# Видалити перший символ
entry.delete(0)

# Видалити кілька символів
entry.delete(0, 4)

# Видалити весь текст
entry.delete(0, tk.END)

# Вставити текст
entry.insert(0, "Python")
```

### Віджети Text

Індекси мають формат `"<рядок>.<символ>"`. Рядки починаються з `1`, символи — з `0`.

```python
# Отримати слово з першого рядка
text_box.get("1.0", "1.5")  # 'Hello'

# Отримати весь текст
text_box.get("1.0", tk.END)

# Вставити текст у кінець
text_box.insert(tk.END, "\nNew line")
```

### Призначення віджетів фреймам

```python
frame_a = tk.Frame()
label_a = tk.Label(master=frame_a, text="I'm in Frame A")
label_a.pack()
frame_a.pack()
```

### Рельєфи рамок Frame

```python
frame = tk.Frame(master=window, relief=tk.RAISED, borderwidth=5)
```

Доступні значення: `tk.FLAT`, `tk.SUNKEN`, `tk.RAISED`, `tk.GROOVE`, `tk.RIDGE`.

### Конвенції іменування

| Клас | Префікс | Приклад |
|------|---------|---------|
| `Label` | `lbl` | `lbl_name` |
| `Button` | `btn` | `btn_submit` |
| `Entry` | `ent` | `ent_age` |
| `Text` | `txt` | `txt_notes` |
| `Frame` | `frm` | `frm_address` |

---

**Завдання для повторення**

1. Напишіть програму з кнопкою 50×25 текстових одиниць, білим фоном і синім текстом `"Click here"`.
2. Напишіть програму з полем `Entry` шириною 40 із текстом `"What is your name?"`.

---

## 18.6 Керування макетом за допомогою менеджерів геометрії

Tkinter надає три менеджери геометрії: `.pack()`, `.place()` і `.grid()`.

### .pack()

```python
frame.pack(fill=tk.X)           # Горизонтальне заповнення
frame.pack(fill=tk.Y)           # Вертикальне заповнення
frame.pack(fill=tk.BOTH, expand=True)  # Обидва напрямки
frame.pack(side=tk.LEFT)        # Розміщення збоку
```

### .place()

```python
label.place(x=75, y=75)         # Точне розміщення в пікселях
```

`.place()` рідко використовується через складність керування макетом і відсутність адаптивності.

### .grid()

```python
import tkinter as tk

window = tk.Tk()
for i in range(3):
    window.columnconfigure(i, weight=1, minsize=75)
    window.rowconfigure(i, weight=1, minsize=50)
    for j in range(3):
        frame = tk.Frame(
            master=window,
            relief=tk.RAISED,
            borderwidth=1
        )
        frame.grid(row=i, column=j, padx=5, pady=5)
        label = tk.Label(master=frame, text=f"Row {i}\nColumn {j}")
        label.pack(padx=5, pady=5)

window.mainloop()
```

Параметр `sticky` керує вирівнюванням у комірці:

```python
label.grid(row=0, column=0, sticky="nsew")  # Заповнює всю комірку
```

| `.grid()` | `.pack()` |
|-----------|-----------|
| `sticky="ns"` | `fill=tk.Y` |
| `sticky="ew"` | `fill=tk.X` |
| `sticky="nsew"` | `fill=tk.BOTH` |

---

**Завдання для повторення**

1. Відтворіть усі скріншоти з цього розділу без підказок із початкового коду.

---

## 18.7 Робимо застосунки інтерактивними

### .bind()

```python
import tkinter as tk

window = tk.Tk()

def handle_keypress(event):
    """Вивести символ натиснутої клавіші."""
    print(event.char)

window.bind("<Key>", handle_keypress)
window.mainloop()
```

### Атрибут command

```python
import tkinter as tk

def increase():
    value = int(lbl_value["text"])
    lbl_value["text"] = f"{value + 1}"

def decrease():
    value = int(lbl_value["text"])
    lbl_value["text"] = f"{value - 1}"

window = tk.Tk()
window.rowconfigure(0, minsize=50, weight=1)
window.columnconfigure([0, 1, 2], minsize=50, weight=1)

btn_decrease = tk.Button(master=window, text="-", command=decrease)
btn_decrease.grid(row=0, column=0, sticky="nsew")

lbl_value = tk.Label(master=window, text="0")
lbl_value.grid(row=0, column=1)

btn_increase = tk.Button(master=window, text="+", command=increase)
btn_increase.grid(row=0, column=2, sticky="nsew")

window.mainloop()
```

---

**Завдання для повторення**

1. Кнопка `"Click me"`, що при натисканні змінює колір фону на випадковий.
2. Програма-симулятор кидання кубика.

---

## 18.8 Приклад застосунку: Конвертер температур

```python
import tkinter as tk

def fahrenheit_to_celsius():
    """Перетворити значення з Фаренгейтів на Цельсій."""
    fahrenheit = ent_temperature.get()
    celsius = (5/9) * (float(fahrenheit) - 32)
    lbl_result["text"] = f"{round(celsius, 2)} \N{DEGREE CELSIUS}"

window = tk.Tk()
window.title("Temperature Converter")
window.resizable(width=False, height=False)

frm_entry = tk.Frame(master=window)
ent_temperature = tk.Entry(master=frm_entry, width=10)
lbl_temp = tk.Label(master=frm_entry, text="\N{DEGREE FAHRENHEIT}")

ent_temperature.grid(row=0, column=0, sticky="e")
lbl_temp.grid(row=0, column=1, sticky="w")

btn_convert = tk.Button(
    master=window,
    text="\N{RIGHTWARDS BLACK ARROW}",
    command=fahrenheit_to_celsius
)
lbl_result = tk.Label(master=window, text="\N{DEGREE CELSIUS}")

frm_entry.grid(row=0, column=0, padx=10)
btn_convert.grid(row=0, column=1, pady=10)
lbl_result.grid(row=0, column=2, padx=10)

window.mainloop()
```

---

## 18.9 Приклад застосунку: Текстовий редактор

```python
import tkinter as tk
from tkinter.filedialog import askopenfilename, asksaveasfilename

def open_file():
    """Відкрити файл для редагування."""
    filepath = askopenfilename(
        filetypes=[("Text Files", "*.txt"), ("All Files", "*.*")]
    )
    if not filepath:
        return
    txt_edit.delete(1.0, tk.END)
    with open(filepath, "r") as input_file:
        text = input_file.read()
    txt_edit.insert(tk.END, text)
    window.title(f"Simple Text Editor - {filepath}")

def save_file():
    """Зберегти поточний файл як новий файл."""
    filepath = asksaveasfilename(
        defaultextension="txt",
        filetypes=[("Text Files", "*.txt"), ("All Files", "*.*")],
    )
    if not filepath:
        return
    with open(filepath, "w") as output_file:
        text = txt_edit.get(1.0, tk.END)
        output_file.write(text)
    window.title(f"Simple Text Editor - {filepath}")

window = tk.Tk()
window.title("Simple Text Editor")
window.rowconfigure(0, minsize=800, weight=1)
window.columnconfigure(1, minsize=800, weight=1)

txt_edit = tk.Text(window)
fr_buttons = tk.Frame(window, relief=tk.RAISED, bd=2)
btn_open = tk.Button(fr_buttons, text="Open", command=open_file)
btn_save = tk.Button(fr_buttons, text="Save As...", command=save_file)

btn_open.grid(row=0, column=0, sticky="ew", padx=5, pady=5)
btn_save.grid(row=1, column=0, sticky="ew", padx=5)

fr_buttons.grid(row=0, column=0, sticky="ns")
txt_edit.grid(row=0, column=1, sticky="nsew")

window.mainloop()
```

---

## 18.10 Задача: Повернення поета

Напишіть GUI-застосунок для генерування поезії, заснований на генераторі з Розділу 9:

1. Поля введення для кожного типу слів (мінімум: 3 іменники, 3 дієслова, 3 прикметники, 3 прийменники, 1 прислівник).
2. При нестачі слів — повідомлення про помилку.
3. Генерація вірша за шаблоном.
4. Можливість експорту вірша у файл.

Розв'язання цієї задачі та інші бонусні матеріали можна знайти онлайн на realpython.com/python-basics/resources.

## 18.11 Підсумок та додаткові ресурси

У цьому розділі ви навчилися будувати прості графічні інтерфейси.

За допомогою EasyGUI ви навчилися створювати діалогові вікна для повідомлень, введення та вибору файлів. Потім ви опанували Tkinter — вбудований GUI-фреймворк Python.

Ви навчилися працювати з віджетами `Frame`, `Label`, `Button`, `Entry` і `Text`, використовувати менеджери геометрії `.pack()`, `.place()` і `.grid()`, а також обробляти події та зв'язувати їх із функціями.

---

**Інтерактивний тест**

realpython.com/quizzes/python-basics-17

**Додаткові ресурси**

- Tkinter tutorial
- Рекомендовані ресурси на realpython.com

---

