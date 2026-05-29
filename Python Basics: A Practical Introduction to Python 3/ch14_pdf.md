# Розділ 14. Створення та модифікація PDF-файлів

PDF, або portable document format («портативний формат документів»), є одним із найпоширеніших форматів для обміну документами в інтернеті. PDF-файли можуть містити текст, зображення, таблиці, форми і навіть мультимедіа — відео та анімації — усе в одному файлі.

Різноманітність типів вмісту, що PDF може містити, може ускладнити роботу з ними. Проте екосистема Python має чудові пакети для читання, маніпулювання та створення PDF-файлів!

У цьому розділі ви навчитеся:

- зчитувати текст із PDF;
- витягувати сторінки та розбивати PDF на кілька файлів;
- конкатенувати та зливати PDF-файли;
- повертати та обрізати сторінки PDF;
- шифрувати та дешифрувати PDF-файли паролями;
- створювати PDF-файли з нуля.

Розпочнемо!

## 14.1 Витягування тексту з PDF

У цьому розділі ви навчитеся читати PDF-файл і витягувати текст за допомогою пакету PyPDF2. Перш за все, встановіть PyPDF2 за допомогою pip:

```
$ python3 -m pip install PyPDF2
```

Перевірте встановлення:

```
$ python3 -m pip show PyPDF2
Name: PyPDF2
Version: 1.26.0
```

Зверніть увагу на інформацію про версію. На момент написання книги остання версія PyPDF2 — 1.26.0.

### Відкриття PDF-файлу

Розпочнемо з відкриття PDF і читання деякої інформації про нього. Використаємо файл `Pride_and_Prejudice.pdf` із папки матеріалів для практики до Розділу 14.

Відкрийте інтерактивне вікно IDLE та імпортуйте клас `PdfFileReader` із пакету PyPDF2:

```python
>>> from PyPDF2 import PdfFileReader
```

Щоб створити новий екземпляр `PdfFileReader`, знадобиться шлях до PDF-файлу:

```python
>>> from pathlib import Path
>>> pdf_path = (
...     Path.home() /
...     "python-basics-exercises" /
...     "ch13-interact-with-pdf-files" /
...     "practice_files" /
...     "Pride_and_Prejudice.pdf"
... )
```

Тепер створіть екземпляр `PdfFileReader`:

```python
>>> pdf = PdfFileReader(str(pdf_path))
```

`pdf_path` перетворюється на рядок, оскільки `PdfFileReader` не вміє читати з об'єктів `pathlib.Path`.

Метод `.getNumPages()` повертає кількість сторінок у PDF-файлі:

```python
>>> pdf.getNumPages()
234
```

Доступ до метаданих документа — через атрибут `.documentInfo`:

```python
>>> pdf.documentInfo.title
'Pride and Prejudice, by Jane Austen'
```

### Витягування тексту зі сторінки

Сторінки PDF представлені в PyPDF2 класом `PageObject`. Є два кроки для витягування тексту з окремої сторінки:

1. Отримати `PageObject` за допомогою `PdfFileReader.getPage()`.
2. Витягти текст у вигляді рядка методом `.extractText()` екземпляра `PageObject`.

```python
>>> first_page = pdf.getPage(0)
>>> type(first_page)
<class 'PyPDF2.pdf.PageObject'>
>>> first_page.extractText()
'\n \nThe Project Gutenberg EBook of Pride and Prejudice...'
```

Щоб витягти текст із усього PDF, можна використати атрибут `.pages`:

```python
>>> for page in pdf.pages:
...     print(page.extractText())
```

### Збираємо все разом

Ось програма, що витягує весь текст із PDF і зберігає його в `.txt`-файл:

```python
from pathlib import Path
from PyPDF2 import PdfFileReader

# Змініть шлях нижче на правильний для вашого комп'ютера.
pdf_path = (
    Path.home() /
    "python-basics-exercises" /
    "ch13-interact-with-pdf-files" /
    "practice-files" /
    "Pride_and_Prejudice.pdf"
)

# 1
pdf_reader = PdfFileReader(str(pdf_path))
output_file_path = Path.home() / "Pride and Prejudice.txt"

# 2
with output_file_path.open(mode="w") as output_file:
    # 3
    output_file.write(
        f"{pdf_reader.documentInfo.title}\n"
        f"Number of pages: {pdf_reader.getNumPages()}\n\n"
    )

    # 4
    for page in pdf_reader.pages:
        text = page.extractText()
        output_file.write(text)
```

---

**Завдання для повторення**

1. У папці матеріалів для практики до Розділу 14 є PDF-файл `zen.pdf`. Створіть із нього екземпляр `PdfFileReader`.
2. За допомогою екземпляра `PdfFileReader` з Завдання 1 виведіть загальну кількість сторінок PDF.
3. Виведіть текст із першої сторінки PDF з Завдання 1.

---

## 14.2 Витягування сторінок із PDF

Тепер ви дізнаєтеся, як витягти сторінку або діапазон сторінок із наявного PDF і зберегти їх у новий PDF.

### Клас PdfFileWriter

`PdfFileWriter` використовується для створення нових PDF-файлів. Об'єкти `PdfFileWriter` є контейнерами для сторінок:

```python
>>> from PyPDF2 import PdfFileWriter
>>> pdf_writer = PdfFileWriter()
```

Щоб додати порожню сторінку:

```python
>>> pdf_writer.addBlankPage(width=72, height=72)
```

> **Важливо**
>
> Коли ви викликаєте `.addBlankPage()`, не потрібно присвоювати результат оригінальному об'єкту — він змінюється «на місці». Крім того, PDF зберігається, передаючи об'єкт файлу методу `.write()` об'єкта `PdfFileWriter`, а не навпаки:
>
> ```python
> # ПРАВИЛЬНО:
> with Path("blank.pdf").open(mode="wb") as output_file:
>     pdf_writer.write(output_file)
>
> # НЕПРАВИЛЬНО:
> with Path("blank.pdf").open(mode="wb") as output_file:
>     output_file.write(pdf_writer)
> ```

Три кроки для створення нового PDF-файлу:

1. Створити екземпляр `PdfFileWriter`.
2. Додати одну або кілька сторінок до екземпляра `PdfFileWriter`.
3. Записати у файл методом `PdfFileWriter.write()`.

### Витягування однієї сторінки

```python
>>> from pathlib import Path
>>> from PyPDF2 import PdfFileReader, PdfFileWriter

>>> input_pdf = PdfFileReader(str(pdf_path))
>>> first_page = input_pdf.getPage(0)

>>> pdf_writer = PdfFileWriter()
>>> pdf_writer.addPage(first_page)

>>> with Path("first_page.pdf").open(mode="wb") as output_file:
...     pdf_writer.write(output_file)
```

### Витягування кількох сторінок

```python
>>> pdf_writer = PdfFileWriter()
>>> for n in range(1, 4):
...     page = input_pdf.getPage(n)
...     pdf_writer.addPage(page)

>>> with Path("chapter1.pdf").open(mode="wb") as output_file:
...     pdf_writer.write(output_file)
```

Альтернативно, використовуючи зріз атрибута `.pages`:

```python
>>> pdf_writer = PdfFileWriter()
>>> for page in input_pdf.pages[1:4]:
...     pdf_writer.addPage(page)

>>> with Path("chapter1_slice.pdf").open(mode="wb") as output_file:
...     pdf_writer.write(output_file)
```

Для копіювання всіх сторінок з `PdfFileReader` є метод `.appendPagesFromReader()`:

```python
>>> pdf_writer = PdfFileWriter()
>>> pdf_writer.appendPagesFromReader(pdf_reader)
```

---

**Завдання для повторення**

1. Витягніть останню сторінку з файлу `Pride_and_Prejudice.pdf` і збережіть у новий файл `last_page.pdf`.
2. Витягніть усі сторінки з парними індексами з `Pride_and_Prejudice.pdf` і збережіть у файл `every_other_page.pdf`.
3. Розділіть `Pride_and_Prejudice.pdf` на два нові PDF-файли: перший — перші 150 сторінок (`part_1.pdf`), другий — сторінки, що залишилися (`part_2.pdf`).

---

## 14.3 Задача: Клас PdfFileSplitter

Створіть клас `PdfFileSplitter`, що зчитує PDF з наявного екземпляра `PdfFileReader` і розділяє PDF на два нові.

Клас має бути інстанційований із рядком шляху:

```python
pdf_splitter = PdfFileSplitter("mydoc.pdf")
```

Клас `PdfFileSplitter` повинен мати два методи:

1. `.split()` — з одним параметром `breakpoint`, що очікує ціле число, яке позначає номер сторінки для розділення. Після виклику `.split()` клас має мати атрибути `.writer1` і `.writer2`.

2. `.write()` — з одним параметром `filename`. При виклику записує два PDF-файли: `filename + "_1.pdf"` і `filename + "_2.pdf"`.

Наприклад:

```python
pdf_splitter.split(breakpoint=4)
pdf_splitter.write("mydoc_split")
```

Перевірте роботу класу, розділивши `Pride_and_Prejudice.pdf` на сторінці 150.

Розв'язання цієї задачі та інші бонусні матеріали можна знайти онлайн на realpython.com/python-basics/resources.

## 14.4 Конкатенація та злиття PDF-файлів

**Конкатенація** — об'єднання кількох PDF-файлів в один послідовно. **Злиття** — вставлення вмісту одного PDF після конкретної сторінки в інший PDF.

### Клас PdfFileMerger

```python
>>> from PyPDF2 import PdfFileMerger
>>> pdf_merger = PdfFileMerger()
```

Є два методи для додавання сторінок:

- `.append()` — конкатенує всі сторінки наявного PDF у кінець.
- `.merge()` — вставляє всі сторінки наявного PDF після конкретної сторінки.

### Конкатенація PDF за допомогою .append()

```python
>>> from pathlib import Path
>>> from PyPDF2 import PdfFileMerger

>>> reports_dir = (
...     Path.home() /
...     "python-basics-exercises" /
...     "ch13-interact-with-pdf-files" /
...     "practice_files" /
...     "expense_reports"
... )

>>> expense_reports = list(reports_dir.glob("*.pdf"))
>>> expense_reports.sort()

>>> pdf_merger = PdfFileMerger()
>>> for path in expense_reports:
...     pdf_merger.append(str(path))

>>> with Path("expense_reports.pdf").open(mode="wb") as output_file:
...     pdf_merger.write(output_file)
```

### Злиття PDF за допомогою .merge()

Метод `.merge()` схожий на `.append()`, але дозволяє вказати, куди вставити вміст. Перший аргумент — індекс сторінки, перед якою слід вставити; другий — рядок зі шляхом до PDF.

```python
>>> pdf_merger = PdfFileMerger()
>>> pdf_merger.append(str(report_path))

# Вставити зміст toc.pdf перед сторінкою з індексом 1
>>> pdf_merger.merge(1, str(toc_path))

>>> with Path("full_report.pdf").open(mode="wb") as output_file:
...     pdf_merger.write(output_file)
```

---

**Завдання для повторення**

1. У папці матеріалів є файли `merge1.pdf` і `merge2.pdf`. За допомогою екземпляра `PdfFileMerger` конкатенуйте їх і збережіть у файл `concatenated.pdf`.
2. За допомогою нового екземпляра `PdfFileMerger` злийте файл `merge3.pdf` між двома сторінками `concatenated.pdf`. Збережіть результат як `merged.pdf`. Фінальний файл повинен мати три сторінки з числами 1, 2 і 3.

---

## 14.5 Обертання та обрізання сторінок PDF

### Обертання сторінок

```python
>>> from pathlib import Path
>>> from PyPDF2 import PdfFileReader, PdfFileWriter

>>> pdf_reader = PdfFileReader(str(pdf_path))
>>> pdf_writer = PdfFileWriter()
```

Метод `PageObject.rotateClockwise()` приймає аргумент у градусах і повертає сторінку за годинниковою стрілкою. Є також `.rotateCounterClockwise()` для повернення проти годинникової стрілки.

**Спосіб 1** — за допомогою індексу:

```python
>>> for n in range(pdf_reader.getNumPages()):
...     page = pdf_reader.getPage(n)
...     if n % 2 == 0:
...         page.rotateClockwise(90)
...     pdf_writer.addPage(page)

>>> with Path("ugly_rotated.pdf").open(mode="wb") as output_file:
...     pdf_writer.write(output_file)
```

**Спосіб 2** — за допомогою ключа `/Rotate`. Об'єкти `PageObject` зберігають словник значень із інформацією про сторінку. Ключ `/Rotate` вказує поточне обертання сторінки:

```python
>>> page = pdf_reader.getPage(0)
>>> page["/Rotate"]
-90
```

```python
>>> pdf_reader = PdfFileReader(str(pdf_path))
>>> pdf_writer = PdfFileWriter()

>>> for page in pdf_reader.pages:
...     if page["/Rotate"] == -90:
...         page.rotateClockwise(90)
...     pdf_writer.addPage(page)

>>> with Path("ugly_rotated2.pdf").open(mode="wb") as output_file:
...     pdf_writer.write(output_file)
```

> **Важливо**
>
> Ключ `/Rotate` не гарантовано присутній на сторінці. Якщо він відсутній, звернення до нього спричинить `KeyError`. Обробляйте виняток за допомогою блоку `try...except`.

### Обрізання сторінок

Об'єкти `PageObject` мають атрибут `.mediaBox`, що представляє прямокутну область, яка визначає межі сторінки:

```python
>>> first_page.mediaBox
RectangleObject([0, 0, 792, 612])
```

`RectangleObject` має чотири атрибути для кутів: `.lowerLeft`, `.lowerRight`, `.upperLeft`, `.upperRight`.

Щоб обрізати сторінку, змінюємо координати:

```python
>>> import copy
>>> first_page = pdf_reader.getPage(0)

# Витягти ліву половину сторінки
>>> left_side = copy.deepcopy(first_page)
>>> current_coords = left_side.mediaBox.upperRight
>>> new_coords = (current_coords[0] / 2, current_coords[1])
>>> left_side.mediaBox.upperRight = new_coords

# Витягти праву половину сторінки
>>> right_side = copy.deepcopy(first_page)
>>> right_side.mediaBox.upperLeft = new_coords

>>> pdf_writer = PdfFileWriter()
>>> pdf_writer.addPage(left_side)
>>> pdf_writer.addPage(right_side)

>>> with Path("cropped_pages.pdf").open(mode="wb") as output_file:
...     pdf_writer.write(output_file)
```

---

**Завдання для повторення**

1. У папці матеріалів є PDF `split_and_rotate.pdf`. Створіть новий PDF `rotated.pdf` із усіма сторінками, повернутими на 90 градусів проти годинникової стрілки.
2. Використовуючи `rotated.pdf`, розбийте кожну сторінку вертикально посередині. Створіть `split.pdf` із усіма розділеними сторінками. Фінальний файл повинен мати чотири сторінки з числами 1, 2, 3 і 4.

---

## 14.6 Шифрування та дешифрування PDF-файлів

### Шифрування PDF

Метод `.encrypt()` екземпляра `PdfFileWriter` використовується для додавання захисту паролем:

1. `user_pwd` — пароль користувача для відкриття та читання PDF.
2. `owner_pwd` — пароль власника для відкриття без обмежень, включно з редагуванням.

```python
>>> from pathlib import Path
>>> from PyPDF2 import PdfFileReader, PdfFileWriter

>>> pdf_reader = PdfFileReader(str(pdf_path))
>>> pdf_writer = PdfFileWriter()
>>> pdf_writer.appendPagesFromReader(pdf_reader)

>>> pdf_writer.encrypt(user_pwd="SuperSecret")

>>> output_path = Path.home() / "newsletter_protected.pdf"
>>> with output_path.open(mode="wb") as output_file:
...     pdf_writer.write(output_file)
```

Для встановлення окремого пароля власника:

```python
>>> pdf_writer.encrypt(user_pwd="SuperSecret", owner_pwd="ReallySuperSecret")
```

### Дешифрування PDF

Для дешифрування захищеного PDF використовується метод `.decrypt()` екземпляра `PdfFileReader`:

```python
>>> pdf_path = Path.home() / "newsletter_protected.pdf"
>>> pdf_reader = PdfFileReader(str(pdf_path))

>>> pdf_reader.decrypt(password="SuperSecret")
1
```

`.decrypt()` повертає ціле число:
- `0` — пароль неправильний;
- `1` — збіг із паролем користувача;
- `2` — збіг із паролем власника.

Після дешифрування можна отримати доступ до вмісту PDF:

```python
>>> pdf_reader.getPage(0)
```

---

**Завдання для повторення**

1. У папці матеріалів є файл `top_secret.pdf`. За допомогою `PdfFileWriter.encrypt()` зашифруйте його паролем `Unguessable` і збережіть як `top_secret_encrypted.pdf`.
2. Відкрийте файл `top_secret_encrypted.pdf`, дешифруйте його та виведіть текст із першої сторінки.

---

## 14.7 Задача: Розплутайте PDF

У папці матеріалів є PDF-файл `scrambled.pdf` із сімома сторінками. Кожна сторінка містить число від 1 до 7, але вони розташовані не по порядку. Крім того, деякі сторінки повернуті на 90, 180 або 270 градусів.

Напишіть скрипт, що розплутує PDF: сортує сторінки за числом із тексту та повертає сторінки, якщо потрібно, щоб вони були у правильному положенні.

> **Примітка**
>
> Можна вважати, що кожен об'єкт `PageObject` з `scrambled.pdf` має ключ `/Rotate`.

Збережіть розплутаний PDF у файл `unscrambled.pdf` у домашній директорії.

Розв'язання цієї задачі та інші бонусні матеріали можна знайти онлайн на realpython.com/python-basics/resources.

## 14.8 Створення PDF-файлу з нуля

PyPDF2 чудово підходить для читання та модифікації наявних PDF-файлів, але має суттєве обмеження: за його допомогою не можна створювати нові PDF-файли.

У цьому розділі ви будете використовувати інструментарій ReportLab для генерування PDF-файлів з нуля. ReportLab — це повнофункціональне рішення для створення PDF. Є комерційна версія, що потребує оплати, і версія з відкритим кодом із обмеженою функціональністю.

### Встановлення reportlab

```
$ python3 -m pip install reportlab
```

### Клас Canvas

Основний інтерфейс для створення PDF у ReportLab — клас `Canvas`, що знаходиться в модулі `reportlab.pdfgen.canvas`:

```python
>>> from reportlab.pdfgen.canvas import Canvas
>>> canvas = Canvas("hello.pdf")
```

Щоб додати текст до PDF, використовується метод `.drawString()`:

```python
>>> canvas.drawString(72, 72, "Hello World")
```

Перші два аргументи визначають позицію тексту на аркуші. Перший — у точках від лівого краю, другий — від нижнього краю. Одна точка дорівнює 1/72 дюйма. Тобто 72 точки — це 1 дюйм.

Щоб зберегти PDF:

```python
>>> canvas.save()
```

### Налаштування розміру сторінки

За замовчуванням розмір сторінки — A4. Щоб змінити, використовується параметр `pagesize`:

```python
>>> from reportlab.lib.pagesizes import LETTER
>>> canvas = Canvas("hello.pdf", pagesize=LETTER)
```

Або з одиницями вимірювання:

```python
>>> from reportlab.lib.units import inch, cm
>>> canvas = Canvas("hello.pdf", pagesize=(8.5 * inch, 11 * inch))
```

Деякі стандартні розміри сторінок:

| Розмір | Розміри |
|--------|---------|
| A4 | 210 мм × 297 мм |
| LETTER | 8,5 дюйма × 11 дюймів |
| LEGAL | 8,5 дюйма × 14 дюймів |
| TABLOID | 11 дюймів × 17 дюймів |

### Налаштування параметрів шрифту

Для зміни шрифту та розміру використовується `Canvas.setFont()`:

```python
>>> canvas = Canvas("font-example.pdf", pagesize=LETTER)
>>> canvas.setFont("Times-Roman", 18)
>>> canvas.drawString(1 * inch, 10 * inch, "Times New Roman (18 pt)")
>>> canvas.save()
```

Доступні шрифти за замовчуванням: `"Courier"`, `"Helvetica"`, `"Times-Roman"` (і їхні варіанти жирного та курсивного накреслення).

Для зміни кольору шрифту використовується `Canvas.setFillColor()`:

```python
from reportlab.lib.colors import blue
from reportlab.lib.pagesizes import LETTER
from reportlab.lib.units import inch
from reportlab.pdfgen.canvas import Canvas

canvas = Canvas("font-colors.pdf", pagesize=LETTER)

# Встановити шрифт Times New Roman розміром 12 пунктів
canvas.setFont("Times-Roman", 12)

# Намалювати синій текст на відстані 1 дюйм зліва
# та 10 дюймів від нижнього краю
canvas.setFillColor("blue")
canvas.drawString(1*inch, 10*inch, "Black text")

# Зберегти PDF-файл
canvas.save()
```

За допомогою ReportLab можна також створювати таблиці, форми та навіть високоякісну графіку з нуля. Посібник користувача ReportLab (ReportLab User Guide) містить безліч прикладів.

## 14.9 Підсумок та додаткові ресурси

У цьому розділі ви навчилися створювати та модифікувати PDF-файли за допомогою пакетів PyPDF2 і reportlab.

За допомогою PyPDF2 ви навчилися:

- читати PDF-файли та витягувати текст за допомогою класу `PdfFileReader`;
- записувати нові PDF-файли за допомогою `PdfFileWriter`;
- конкатенувати та зливати PDF-файли за допомогою класу `PdfFileMerger`;
- повертати та обрізати сторінки PDF;
- шифрувати та дешифрувати PDF-файли паролями.

Також ви отримали вступ до створення PDF-файлів з нуля за допомогою пакету reportlab та познайомилися з класом `Canvas`, методами `.drawString()`, `.setFont()` і `.setFillColor()`.

---

**Інтерактивний тест**

До цього розділу додається безкоштовний онлайн-тест для перевірки прогресу у навчанні. Пройти тест можна з телефону або комп'ютера за адресою:

realpython.com/quizzes/python-basics-13

**Додаткові ресурси**

Щоб дізнатися більше про роботу з PDF-файлами в Python, зверніться до таких матеріалів:

- How to Work with a PDF in Python
- ReportLab PDF Library User Guide
- Рекомендовані ресурси на realpython.com
