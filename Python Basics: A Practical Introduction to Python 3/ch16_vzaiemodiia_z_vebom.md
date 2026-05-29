# Розділ 16. Взаємодія з вебом

Інтернет є, мабуть, найбільшим джерелом інформації — і дезінформації — на планеті.

Багато дисциплін, таких як наука про дані, бізнес-аналітика та журналістські розслідування, можуть отримати величезну вигоду зі збору та аналізу даних із вебсайтів.

Вебскрейпінг (web scraping) — це процес збору та розбору необроблених даних із вебу. Спільнота Python розробила кілька потужних інструментів для цього.

У цьому розділі ви навчитеся:

- розбирати дані вебсайтів за допомогою методів рядків і регулярних виразів;
- розбирати дані вебсайтів за допомогою HTML-парсера;
- взаємодіяти з формами та іншими компонентами вебсайтів.

> **Важливо**
>
> Певний досвід роботи з HTML (HyperText Markup Language) буде корисним при читанні цього розділу.

Вперед!

## 16.1 Скрейпінг і розбір тексту з вебсайтів

Збирання даних із вебсайтів за допомогою автоматизованого процесу відоме як вебскрейпінг. Деякі вебсайти явно забороняють скрейпінг своїх даних автоматизованими інструментами.

> **Важливо**
>
> Завжди перевіряйте прийнятну політику використання вебсайту перед скрейпінгом його даних, щоб дізнатися, чи є використання автоматизованих інструментів порушенням умов використання. З правової точки зору вебскрейпінг проти бажань вебсайту є дуже «сірою зоною».
>
> Пам'ятайте: наведені нижче техніки можуть бути незаконними, якщо їх застосовувати на вебсайтах, що забороняють вебскрейпінг.

Розпочнемо зі збирання всього HTML-коду з окремої вебсторінки:

```python
from urllib.request import urlopen

url = "http://olympus.realpython.org/profiles/aphrodite"
html_page = urlopen(url)
html_text = html_page.read().decode("utf-8")
print(html_text)
```

Це відображає повний HTML сторінки:

```html
<html>
<head>
<title>Profile: Aphrodite</title>
</head>
<body bgcolor="yellow">
<center>
<br><br>
<img src="/static/aphrodite.gif" />
<h2>Name: Aphrodite</h2>
...
</center>
</body>
</html>
```

Тепер можна витягти конкретну інформацію зі сторінки. Наприклад, отримати заголовок сторінки за допомогою методу рядка `find()`:

```python
from urllib.request import urlopen

url = "http://olympus.realpython.org/profiles/aphrodite"
page = urlopen(url)
html = page.read().decode('utf-8')

start_tag = "<title>"
end_tag = "</title>"
start_index = html.find(start_tag) + len(start_tag)
end_index = html.find(end_tag)
print(html[start_index:end_index])
```

Результат: `Profile: Aphrodite`.

Однак HTML у реальному світі може бути набагато складнішим і непередбачуваним. Більш надійною альтернативою є використання регулярних виразів.

### Регулярні вирази

Регулярні вирази (regular expressions, або «regex») — це рядки, що можуть використовуватися для визначення того, чи відповідає текст певному шаблону.

Python надає вбудовану підтримку регулярних виразів через модуль `re`.

Функція `re.findall()` знаходить будь-який текст у рядку, що відповідає регулярному виразу:

```python
>>> import re
>>> re.findall("ab*c", "ac")
['ac']
>>> re.findall("ab*c", "abcd")
['abc']
>>> re.findall("ab*c", "abcac")
['abc', 'ac']
>>> re.findall("ab*c", "abdc")
[]
```

Регулярний вираз `ab*c` відповідає будь-якій частині рядка, що починається з `"a"`, закінчується на `"c"` і має нуль або більше `"b"` між ними. `re.findall()` повертає список усіх збігів.

Зіставлення чутливе до регістру. Для порівняння без урахування регістру передайте третій аргумент `re.IGNORECASE`:

```python
>>> re.findall("ab*c", "ABC", re.IGNORECASE)
['ABC']
```

Символ `.` відповідає будь-якому одиночному символу:

```python
>>> re.findall("a.c", "abc")
['abc']
>>> re.findall("a.c", "acc")
['acc']
```

Вираз `.*` всередині регулярного виразу означає будь-який символ, повторений будь-яку кількість разів:

```python
>>> re.findall("a.*c", "abbc")
['abbc']
```

Функція `re.search()` повертає об'єкт `MatchObject`. Метод `.group()` повертає перший і найбільш включний результат:

```python
>>> match_results = re.search("ab*c", "ABC", re.IGNORECASE)
>>> match_results.group()
'ABC'
```

Функція `re.sub()` дозволяє замінити текст, що відповідає регулярному виразу, новим текстом:

```python
>>> string = "Everything is <replaced> if it's in <tags>."
>>> string = re.sub("<.*>", "ELEPHANTS", string)
>>> string
'Everything is ELEPHANTS.'
```

Зверніть увагу: регулярні вирази Python є «жадібними» — вони намагаються знайти найдовший можливий збіг. Щоб знайти найкоротший збіг, використовуйте `*?` (нежадібний квантор):

```python
>>> string = "Everything is <replaced> if it's in <tags>."
>>> string = re.sub("<.*?>", "ELEPHANTS", string)
>>> string
"Everything is ELEPHANTS if it's in ELEPHANTS."
```

Ось приклад використання регулярних виразів для розбору заголовка зі складного HTML:

```python
import re
from urllib.request import urlopen

url = "http://olympus.realpython.org/profiles/dionysus"
page = urlopen(url)
html = page.read().decode("utf-8")

pattern = "<title.*?>.*?</title.*?>"
match_results = re.search(pattern, html, re.IGNORECASE)
title = match_results.group()
title = re.sub("<.*?>", "", title)  # Видалити HTML-теги
print(title)
```

---

**Завдання для повторення**

Розв'язання цих завдань та інші бонусні матеріали можна знайти онлайн на realpython.com/python-basics/resources.

1. Напишіть скрипт, що отримує повний HTML зі сторінки `http://olympus.realpython.org/profiles/dionysus`.
2. За допомогою методу рядка `.find()` відобразіть текст після «Name:» і «Favorite Color:».
3. Повторіть попереднє завдання, використовуючи регулярні вирази.

---

## 16.2 Використання HTML-парсера для скрейпінгу вебсайтів

Хоча регулярні вирази чудово підходять для пошуку збігів із шаблонами, іноді простіше використовувати HTML-парсер. Бібліотека Beautiful Soup — хороший варіант для початку.

### Встановлення Beautiful Soup

```
$ pip3 install beautifulsoup4
```

### Використання Beautiful Soup

```python
from bs4 import BeautifulSoup
from urllib.request import urlopen

url = "http://olympus.realpython.org/profiles/dionysus"
page = urlopen(url)
html = page.read().decode("utf-8")
soup = BeautifulSoup(html, "html.parser")
```

Об'єкт `BeautifulSoup` отримує два аргументи: HTML для розбору та рядок `"html.parser"`, що вказує використовуваний парсер.

Метод `.get_text()` витягує весь текст документа, автоматично видаляючи HTML-теги:

```python
>>> print(soup.get_text())

Profile: Dionysus

Name: Dionysus

Hometown: Mount Olympus
...
```

Метод `find_all()` повертає список усіх екземплярів певного тегу:

```python
>>> soup.find_all("img")
[<img src="/static/dionysus.jpg"/>, <img src="/static/grapes.png"/>]
```

Об'єкти `Tag` надають простий інтерфейс для роботи з інформацією:

```python
>>> image1, image2 = soup.find_all("img")
>>> image1.name
'img'
>>> image1["src"]
'/static/dionysus.jpg'
```

Для доступу до тегу `<title>` можна використати властивість `.title`:

```python
>>> soup.title
<title>Profile: Dionysus</title>
>>> soup.title.string
'Profile: Dionysus'
```

Beautiful Soup дозволяє шукати певні теги з атрибутами, що відповідають конкретним значенням:

```python
>>> soup.find_all("img", src="/static/dionysus.jpg")
[<img src="/static/dionysus.jpg"/>]
```

> **Примітка**
>
> Іноді HTML настільки погано написаний, що навіть такий потужний парсер як Beautiful Soup не може правильно інтерпретувати HTML-теги. У цьому випадку доводиться вдаватися до `.find()` і регулярних виразів.

---

**Завдання для повторення**

Розв'язання цих завдань та інші бонусні матеріали можна знайти онлайн на realpython.com/python-basics/resources.

1. Напишіть скрипт, що отримує повний HTML зі сторінки `http://olympus.realpython.org/profiles`.
2. За допомогою Beautiful Soup розберіть список усіх посилань на сторінці, шукаючи теги `a` і отримуючи значення атрибута `href`.
3. Отримайте HTML із кожної зі сторінок у списку та відобразіть текст (без HTML-тегів) за допомогою методу `.get_text()`.

---

## 16.3 Взаємодія з HTML-формами

Іноді для отримання потрібного вмісту необхідно взаємодіяти з вебсторінкою — наприклад, відправити форму або натиснути кнопку.

Стандартна бібліотека Python не надає вбудованих засобів для інтерактивної роботи з вебсторінками, але є сторонні пакети. Серед них MechanicalSoup є популярним і відносно простим у використанні.

MechanicalSoup встановлює так званий headless browser — вебброузер без графічного інтерфейсу, яким керують програмно через Python-скрипт.

```
$ pip3 install MechanicalSoup
```

Ось простий скрипт для заповнення та відправки форми входу:

```python
import mechanicalsoup

# 1
browser = mechanicalsoup.Browser()
url = "http://olympus.realpython.org/login"
login_page = browser.get(url)
login_html = login_page.soup

# 2
form = login_html.select("form")[0]
form.select("input")[0]["value"] = "zeus"
form.select("input")[1]["value"] = "ThunderDude"

# 3
profiles_page = browser.submit(form, login_page.url)
```

Після збереження та запуску скрипту можна перевірити успішний вхід:

```python
>>> profiles_page.url
'http://olympus.realpython.org/profiles'
```

Розберімо приклад:

1. Створюється екземпляр `Browser` і отримується сторінка входу.
2. Форма заповнюється: ім'я користувача `"zeus"` і пароль `"ThunderDude"`.
3. Форма відправляється за допомогою `browser.submit()`.

> **Примітка**
>
> Автоматизовані скрипти, подібні до щойно розробленого, можуть використовуватися зловмисниками для «брутфорс-атак» — швидкого перебору паролів. Майже всі вебсайти блокують доступ при надмірній кількості невдалих запитів. Не намагайтеся цього робити!

Тепер можна програмно отримати URL-адресу кожного посилання на сторінці `/profiles`:

```python
>>> links = profiles_page.soup.select("a")
>>> base_url = "http://olympus.realpython.org"
>>> for link in links:
...     address = base_url + link["href"]
...     text = link.text
...     print(f"{text}: {address}")
Aphrodite: http://olympus.realpython.org/profiles/aphrodite
Poseidon: http://olympus.realpython.org/profiles/poseidon
Dionysus: http://olympus.realpython.org/profiles/dionysus
```

---

**Завдання для повторення**

1. За допомогою MechanicalSoup введіть правильне ім'я користувача `"zeus"` і пароль `"ThunderDude"` у форму входу на `http://olympus.realpython.org/login`.
2. Відобразіть заголовок поточної сторінки, щоб переконатися, що вас переадресовано на сторінку `/profiles`.
3. Використовуючи MechanicalSoup, поверніться на сторінку входу, перейшовши «назад».
4. Введіть неправильне ім'я користувача і пароль, а потім знайдіть у HTML поверненої сторінки текст `"Wrong username or password!"`.

---

## 16.4 Взаємодія з вебсайтами в режимі реального часу

Іноді потрібно отримувати дані в реальному часі з вебсайту, що пропонує постійно оновлювану інформацію. Для автоматизації цього процесу можна використати метод `.get()` об'єкта `Browser` із MechanicalSoup.

Наступний скрипт отримує результат кидання кубика чотири рази з інтервалом 30 секунд:

```python
import time
import mechanicalsoup

browser = mechanicalsoup.Browser()

for i in range(4):
    page = browser.get("http://olympus.realpython.org/dice")
    tag = page.soup.select("#result")[0]
    result = tag.text
    print(f"The result of your dice roll is: {result}")

    # Чекати 30 секунд, якщо це не останній запит
    if i < 3:
        time.sleep(30)
```

Функція `time.sleep()` з модуля `time` призупиняє виконання програми на вказану кількість секунд.

> **Примітка**
>
> Виконання багаторазових запитів до вебсайту в короткий проміжок часу може сприйматися як підозрілий або навіть зловмисний трафік. Завжди перевіряйте умови використання вебсайту і ставтеся до чужих серверів із повагою!

---

**Завдання для повторення**

1. Повторіть приклад скрейпінгу результату кидання кубика, але також відображайте поточний час, отриманий зі сторінки.

---

## 16.5 Підсумок та додаткові ресурси

Робота з даними з Інтернету може бути складною. Структура вебсайтів суттєво відрізняється від сайту до сайту, і навіть один і той самий вебсайт може часто змінюватися.

У цьому розділі ви дізналися про Beautiful Soup і MechanicalSoup — два інструменти для написання Python-програм, що автоматизують взаємодію з вебсайтами. Beautiful Soup використовується для розбору HTML-даних зі сторінок, MechanicalSoup — для взаємодії з компонентами вебсайтів, як-от натискання посилань і відправка форм.

Прийоми вебскрейпінгу використовуються в багатьох реальних дисциплінах. Наприклад, журналісти-розслідувачі покладаються на дані, зібрані з великої кількості ресурсів.

Пишіть автоматизовані програми вебскрейпінгу з розумом. Завжди перевіряйте умови використання вебсайту і не надсилайте надмірну кількість запитів, щоб не перевантажити сервер.

---

**Інтерактивний тест**

realpython.com/quizzes/python-basics-15

**Додаткові ресурси**

- Practical Introduction to Web Scraping in Python
- API Integration in Python
- Рекомендовані ресурси на realpython.com
