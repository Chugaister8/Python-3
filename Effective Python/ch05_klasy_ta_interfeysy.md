# Розділ 5. Класи та інтерфейси

Python як об'єктно-орієнтована мова підтримує повний спектр можливостей: успадкування, поліморфізм та інкапсуляцію. Виконання завдань у Python часто вимагає написання нових класів і визначення їхньої взаємодії через інтерфейси та ієрархії.

Класи та успадкування в Python спрощують вираження запланованої поведінки програми за допомогою об'єктів. Вони дозволяють покращувати та розширювати функціональність з плином часу. Вони надають гнучкість в умовах мінливих вимог. Знання того, як правильно їх використовувати, дає змогу писати підтримуваний код.

---

## Порада 37. Компонуйте класи замість глибокого вкладення вбудованих типів

Вбудований тип `dict` у Python чудово підходить для підтримки динамічного внутрішнього стану протягом усього існування об'єкта. Під «динамічним» маються на увазі ситуації, коли потрібно вести облік непередбачуваного набору ідентифікаторів. Наприклад, потрібно записувати оцінки учнів, чиї імена заздалегідь невідомі. Можна визначити клас для зберігання імен у словнику, замість використання заздалегідь визначеного атрибута для кожного учня:

```python
class SimpleGradebook:
    def __init__(self):
        self._grades = {}
    def add_student(self, name):
        self._grades[name] = []
    def report_grade(self, name, score):
        self._grades[name].append(score)
    def average_grade(self, name):
        grades = self._grades[name]
        return sum(grades) / len(grades)
```

Використання класу є простим:

```python
book = SimpleGradebook()
book.add_student('Isaac Newton')
book.report_grade('Isaac Newton', 90)
book.report_grade('Isaac Newton', 95)
book.report_grade('Isaac Newton', 85)
print(book.average_grade('Isaac Newton'))
>>>
90.0
```

Словники та пов'язані вбудовані типи настільки прості у використанні, що існує небезпека надмірно їх розширювати й писати крихкий код. Наприклад, потрібно розширити клас `SimpleGradebook` для зберігання списку оцінок за предметами, а не лише загалом. Це можна зробити, змінивши словник `_grades` так, щоб він відображав імена учнів (ключі) на ще один словник (значення). Внутрішній словник відображатиме предмети (ключі) на список оцінок (значення). Тут для внутрішнього словника використовується екземпляр `defaultdict` для обробки відсутніх предметів (см. Порада 17: «Надавайте перевагу defaultdict над setdefault для обробки відсутніх елементів у внутрішньому стані»):

```python
from collections import defaultdict

class BySubjectGradebook:
    def __init__(self):
        self._grades = {}               # Зовнішній словник
    def add_student(self, name):
        self._grades[name] = defaultdict(list)  # Внутрішній словник
    def report_grade(self, name, subject, grade):
        by_subject = self._grades[name]
        grade_list = by_subject[subject]
        grade_list.append(grade)
    def average_grade(self, name):
        by_subject = self._grades[name]
        total, count = 0, 0
        for grades in by_subject.values():
            total += sum(grades)
            count += len(grades)
        return total / count
```

Використання класу залишається простим:

```python
book = BySubjectGradebook()
book.add_student('Albert Einstein')
book.report_grade('Albert Einstein', 'Math', 75)
book.report_grade('Albert Einstein', 'Math', 65)
book.report_grade('Albert Einstein', 'Gym', 90)
book.report_grade('Albert Einstein', 'Gym', 95)
print(book.average_grade('Albert Einstein'))
>>>
81.25
```

Тепер уявімо, що вимоги знову змінилися. Також потрібно відстежувати вагу кожної оцінки щодо підсумкової, щоб іспити були важливішими за контрольні роботи. Один зі способів реалізувати цю функцію — змінити внутрішній словник; замість відображення предметів (ключів) на список оцінок (значень) можна використати кортеж `(score, weight)` у списку значень:

```python
class WeightedGradebook:
    def __init__(self):
        self._grades = {}
    def add_student(self, name):
        self._grades[name] = defaultdict(list)
    def report_grade(self, name, subject, score, weight):
        by_subject = self._grades[name]
        grade_list = by_subject[subject]
        grade_list.append((score, weight))
    def average_grade(self, name):
        by_subject = self._grades[name]
        score_sum, score_count = 0, 0
        for subject, scores in by_subject.items():
            subject_avg, total_weight = 0, 0
            for score, weight in scores:
                subject_avg += score * weight
                total_weight += weight
            score_sum += subject_avg / total_weight
            score_count += 1
        return score_sum / score_count
```

Хоча зміни в `report_grade` здаються простими, метод `average_grade` тепер має цикл усередині циклу і важко читається. Використання класу також ускладнилося; незрозуміло, що означають усі числа в позиційних аргументах:

```python
book = WeightedGradebook()
book.add_student('Albert Einstein')
book.report_grade('Albert Einstein', 'Math', 75, 0.05)
book.report_grade('Albert Einstein', 'Math', 65, 0.15)
book.report_grade('Albert Einstein', 'Math', 70, 0.80)
book.report_grade('Albert Einstein', 'Gym', 100, 0.40)
book.report_grade('Albert Einstein', 'Gym', 85, 0.60)
print(book.average_grade('Albert Einstein'))
>>>
80.25
```

Коли з'являється така складність, настав час перейти від вбудованих типів, як-от словники, кортежі, множини та списки, до ієрархії класів.

У прикладі з оцінками спочатку не було відомо, що знадобиться підтримка зважених оцінок, тому складність створення класів здавалася невиправданою. Вбудовані типи `dict` та `tuple` Python спростили додавання шарів до внутрішнього обліку. Але не слід робити це більш ніж на одному рівні вкладеності; використання словників, що містять словники, ускладнює читання коду іншими програмістами і створює проблеми для підтримки. Щойно обліковий код стає складним, слід переносити все у класи.

### Рефакторинг до класів

Розпочати перехід до класів можна знизу дерева залежностей: з однієї оцінки. Клас видається надто важким для такої простої інформації. Кортеж, однак, здається доречним, оскільки оцінки є незмінними. Тут кортеж `(score, weight)` використовується для відстеження оцінок у списку:

```python
grades = []
grades.append((95, 0.45))
grades.append((85, 0.55))
total = sum(score * weight for score, weight in grades)
total_weight = sum(weight for _, weight in grades)
average_grade = total / total_weight
```

Тут використовується `_` (ім'я змінної-підкреслення — угода Python для невикористовуваних змінних) для захоплення першого елемента кортежу кожної оцінки і його ігнорування при обчисленні `total_weight`.

Проблема цього коду в тому, що екземпляри кортежів є позиційними. Якщо, наприклад, потрібно пов'язати з оцінкою більше інформації, наприклад нотатки вчителя, доведеться переписати кожне використання двоелементного кортежу так, щоб врахувати наявність трьох елементів замість двох, що означає необхідність додаткового використання `_` для ігнорування певних індексів:

```python
grades = []
grades.append((95, 0.45, 'Great job'))
grades.append((85, 0.55, 'Better next time'))
total = sum(score * weight for score, weight, _ in grades)
total_weight = sum(weight for _, weight, _ in grades)
average_grade = total / total_weight
```

Цей шаблон нескінченного подовження кортежів аналогічний поглибленню рівнів словників. Щойно виявляється, що кортеж довший за двоелементний, настав час розглянути інший підхід.

Тип `namedtuple` з вбудованого модуля `collections` робить саме те, що потрібно в цьому випадку: він дозволяє легко визначати мінімальні незмінні класи даних:

```python
from collections import namedtuple

Grade = namedtuple('Grade', ('score', 'weight'))
```

Ці класи можна конструювати за допомогою позиційних або іменованих аргументів. Поля доступні за іменованими атрибутами. Наявність іменованих атрибутів полегшує перехід від `namedtuple` до класу пізніше, якщо вимоги знову зміняться і знадобиться, наприклад, підтримка змінюваності або поведінки в простих контейнерах даних.

**Обмеження namedtuple**

Хоча `namedtuple` корисний у багатьох обставинах, важливо розуміти, коли він може завдати більше шкоди, ніж користі:

- Не можна вказувати значення аргументів за замовчуванням для класів `namedtuple`. Це робить їх незручними, коли дані можуть мати багато необов'язкових властивостей. Якщо виявляєте, що використовуєте більше кількох атрибутів, вбудований модуль `dataclasses` може бути кращим вибором.
- Значення атрибутів екземплярів `namedtuple` все ще доступні за числовими індексами та через ітерацію. Особливо у зовнішніх API це може призводити до ненавмисного використання, що ускладнює подальший перехід до справжнього класу. Якщо немає контролю над усіма варіантами використання екземплярів `namedtuple`, краще явно визначити новий клас.

Далі можна написати клас для представлення одного предмета, що містить набір оцінок:

```python
class Subject:
    def __init__(self):
        self._grades = []
    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))
    def average_grade(self):
        total, total_weight = 0, 0
        for grade in self._grades:
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight
```

Потім — клас для представлення набору предметів, що вивчаються одним учнем:

```python
class Student:
    def __init__(self):
        self._subjects = defaultdict(Subject)
    def get_subject(self, name):
        return self._subjects[name]
    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count
```

Нарешті — контейнер для всіх учнів, індексований динамічно за іменами:

```python
class Gradebook:
    def __init__(self):
        self._students = defaultdict(Student)
    def get_student(self, name):
        return self._students[name]
```

Кількість рядків цих класів майже вдвічі перевищує розмір попередньої реалізації. Але цей код значно легший для читання. Приклад, що керує класами, є також більш зрозумілим і розширюваним:

```python
book = Gradebook()
albert = book.get_student('Albert Einstein')
math = albert.get_subject('Math')
math.report_grade(75, 0.05)
math.report_grade(65, 0.15)
math.report_grade(70, 0.80)
gym = albert.get_subject('Gym')
gym.report_grade(100, 0.40)
gym.report_grade(85, 0.60)
print(albert.average_grade())
>>>
80.25
```

Також можна написати методи зворотної сумісності для полегшення міграції зі старого стилю API до нової ієрархії об'єктів.

**Що слід запам'ятати**

- Уникайте створення словників зі значеннями, що є словниками, довгими кортежами або складними вкладеннями інших вбудованих типів.
- Використовуйте `namedtuple` для легких незмінних контейнерів даних, перш ніж вам знадобиться гнучкість повноцінного класу.
- Переносьте обліковий код до використання кількох класів, коли внутрішні словники стану ускладнюються.

---

## Порада 38. Приймайте функції замість класів для простих інтерфейсів

Багато вбудованих API Python дозволяють налаштовувати поведінку, передаючи функцію. Ці hook-и використовуються API для зворотного виклику вашого коду під час виконання. Наприклад, метод `sort` типу `list` приймає необов'язковий аргумент `key`, що використовується для визначення значення кожного індексу для сортування (см. Порада 14: «Сортуйте за складними критеріями за допомогою параметра key»). Тут список імен сортується за їхньою довжиною шляхом надання вбудованої функції `len` як hook-а для ключа:

```python
names = ['Socrates', 'Archimedes', 'Plato', 'Aristotle']
names.sort(key=len)
print(names)
>>>
['Plato', 'Socrates', 'Aristotle', 'Archimedes']
```

В інших мовах hook-и зазвичай визначаються абстрактним класом. У Python багато hook-ів — це просто функції без стану з добре визначеними аргументами та значеннями, що повертаються. Функції ідеальні для hook-ів, оскільки їх легше описати та простіше визначити, ніж класи. Функції працюють як hook-и, оскільки Python підтримує функції першого класу: функції та методи можна передавати та посилатися на них як на будь-яке інше значення в мові.

Наприклад, потрібно налаштувати поведінку класу `defaultdict` (см. Порада 17: «Надавайте перевагу defaultdict над setdefault для обробки відсутніх елементів у внутрішньому стані»). Ця структура даних дозволяє надати функцію, що буде викликатися без аргументів при кожному доступі до відсутнього ключа. Функція повинна повертати значення за замовчуванням, яке матиме відсутній ключ у словнику. Тут визначається hook, що журналює кожне додавання ключа і повертає `0` як значення за замовчуванням:

```python
def log_missing():
    print('Key added')
    return 0
```

Маючи початковий словник і набір бажаних приростів, можна змусити функцію `log_missing` виконатися та вивести повідомлення двічі (для `'red'` та `'orange'`):

```python
from collections import defaultdict

current = {'green': 12, 'blue': 3}
increments = [
    ('red', 5),
    ('blue', 17),
    ('orange', 9),
]
result = defaultdict(log_missing, current)
print('Before:', dict(result))
for key, amount in increments:
    result[key] += amount
print('After: ', dict(result))
>>>
Before: {'green': 12, 'blue': 3}
Key added
Key added
After:  {'green': 12, 'blue': 20, 'red': 5, 'orange': 9}
```

Надання таких функцій, як `log_missing`, спрощує побудову та тестування API, оскільки розділяє побічні ефекти від детермінованої поведінки. Наприклад, тепер потрібно, щоб hook значення за замовчуванням, переданий у `defaultdict`, підраховував загальну кількість відсутніх ключів. Один зі способів — використання статичного замикання (см. Порада 21: «Знайте, як замикання взаємодіють зі scope змінних»). Тут визначається допоміжна функція, що використовує таке замикання як hook значення за замовчуванням:

```python
def increment_with_report(current, increments):
    added_count = 0
    def missing():
        nonlocal added_count  # Статичне замикання
        added_count += 1
        return 0
    result = defaultdict(missing, current)
    for key, amount in increments:
        result[key] += amount
    return result, added_count
```

Виконання цієї функції дає очікуваний результат (2), хоча `defaultdict` не знає, що hook `missing` підтримує стан:

```python
result, count = increment_with_report(current, increments)
assert count == 2
```

Проблема з визначенням замикання для статичних hook-ів полягає в тому, що воно важче для читання, ніж приклад з функцією без стану. Інший підхід — визначити невеликий клас, що інкапсулює стан, який потрібно відстежувати:

```python
class CountMissing:
    def __init__(self):
        self.added = 0
    def missing(self):
        self.added += 1
        return 0
```

В інших мовах можна очікувати, що тепер `defaultdict` потрібно модифікувати для розміщення інтерфейсу `CountMissing`. Але в Python завдяки функціям першого класу можна посилатися безпосередньо на метод `CountMissing.missing` на об'єкті та передавати його в `defaultdict` як hook значення за замовчуванням:

```python
counter = CountMissing()
result = defaultdict(counter.missing, current)  # Посилання на метод
for key, amount in increments:
    result[key] += amount
assert counter.added == 2
```

Використання такого допоміжного класу для забезпечення поведінки статичного замикання є зрозумілішим, ніж функція `increment_with_report`. Однак окремо взятий клас `CountMissing` не є одразу очевидним. Python дозволяє класам визначати спеціальний метод `__call__`. `__call__` дозволяє викликати об'єкт так само, як функцію. Він також змушує вбудовану функцію `callable` повертати `True` для такого екземпляра, як і для звичайної функції або методу. Всі об'єкти, що можуть виконуватися таким чином, називаються callable-об'єктами:

```python
class BetterCountMissing:
    def __init__(self):
        self.added = 0
    def __call__(self):
        self.added += 1
        return 0

counter = BetterCountMissing()
assert counter() == 0
assert callable(counter)
```

Тут екземпляр `BetterCountMissing` використовується як hook значення за замовчуванням для `defaultdict` для відстеження кількості відсутніх доданих ключів:

```python
counter = BetterCountMissing()
result = defaultdict(counter, current)  # Покладається на __call__
for key, amount in increments:
    result[key] += amount
assert counter.added == 2
```

Це значно зрозуміліше, ніж приклад із `CountMissing.missing`. Метод `__call__` вказує, що екземпляри класу використовуватимуться там, де також підійшов би аргумент-функція (наприклад, API hook-и). Він спрямовує нових читачів коду до точки входу, відповідальної за основну поведінку класу. Він надає чіткий натяк на те, що мета класу — діяти як статичне замикання.

**Що слід запам'ятати**

- Замість визначення та ініціалізації класів часто можна просто використовувати функції для простих інтерфейсів між компонентами в Python.
- Посилання на функції та методи в Python є об'єктами першого класу, тобто їх можна використовувати у виразах (як і будь-який інший тип).
- Спеціальний метод `__call__` дозволяє викликати екземпляри класу як звичайні Python-функції.
- Коли потрібно, щоб функція підтримувала стан, розгляньте визначення класу з методом `__call__` замість визначення статичного замикання.

---

## Порада 39. Використовуйте поліморфізм @classmethod для узагальненого конструювання об'єктів

У Python поліморфізм підтримується не лише для об'єктів, але й для класів. Що це означає і для чого це корисно?

Поліморфізм дозволяє кільком класам в ієрархії реалізовувати власні унікальні версії методу. Це означає, що багато класів можуть виконувати той самий інтерфейс або абстрактний базовий клас, забезпечуючи при цьому різну функціональність (см. Порада 43: «Успадковуйте від collections.abc для власних типів-контейнерів»).

Наприклад, є реалізація MapReduce і потрібен загальний клас для представлення вхідних даних. Тут визначається такий клас з методом `read`, що має бути визначений підкласами:

```python
class InputData:
    def read(self):
        raise NotImplementedError
```

Є також конкретний підклас `InputData`, що читає дані з файлу на диску:

```python
class PathInputData(InputData):
    def __init__(self, path):
        super().__init__()
        self.path = path
    def read(self):
        with open(self.path) as f:
            return f.read()
```

Потрібен аналогічний абстрактний інтерфейс для воркера MapReduce, що споживає вхідні дані стандартним способом:

```python
class Worker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None
    def map(self):
        raise NotImplementedError
    def reduce(self, other):
        raise NotImplementedError
```

Тут визначається конкретний підклас `Worker` для реалізації конкретної функції MapReduce — простого підрахунку рядків:

```python
class LineCountWorker(Worker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')
    def reduce(self, other):
        self.result += other.result
```

Найпростіший підхід — вручну побудувати та з'єднати об'єкти за допомогою допоміжних функцій. Тут перелічується вміст директорії і для кожного файлу створюється екземпляр `PathInputData`:

```python
import os

def generate_inputs(data_dir):
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))
```

Далі створюються екземпляри `LineCountWorker` за допомогою екземплярів `InputData`:

```python
def create_workers(input_list):
    workers = []
    for input_data in input_list:
        workers.append(LineCountWorker(input_data))
    return workers
```

Ці екземпляри `Worker` виконуються шляхом розподілу кроку `map` по кількох потоках (см. Порада 53: «Використовуйте потоки для блокуючого введення-виведення, уникайте для паралелізму»). Потім `reduce` викликається повторно для об'єднання результатів в одне фінальне значення:

```python
from threading import Thread

def execute(workers):
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads: thread.start()
    for thread in threads: thread.join()
    first, *rest = workers
    for worker in rest:
        first.reduce(worker)
    return first.result
```

Нарешті, всі частини з'єднуються у функції для запуску кожного кроку:

```python
def mapreduce(data_dir):
    inputs = generate_inputs(data_dir)
    workers = create_workers(inputs)
    return execute(workers)
```

Велика проблема в тому, що функція `mapreduce` зовсім не є узагальненою. Якщо потрібно написати інший підклас `InputData` або `Worker`, також доведеться переписати функції `generate_inputs`, `create_workers` та `mapreduce` для відповідності.

Ця проблема зводиться до необхідності узагальненого способу конструювання об'єктів. В інших мовах це вирішується за допомогою поліморфізму конструктора, вимагаючи, щоб кожен підклас `InputData` мав спеціальний конструктор, що може використовуватися узагальнено допоміжними методами. Але Python допускає лише один метод конструктора `__init__`. Вимагати від кожного підкласу `InputData` сумісного конструктора є нерозумним.

Найкращий спосіб вирішення — поліморфізм методів класу. Це аналогічно поліморфізму методів екземпляра для `InputData.read`, але для цілих класів, а не для їхніх сконструйованих об'єктів.

Тут ця ідея застосовується до класів MapReduce. Клас `InputData` розширюється узагальненим `@classmethod`, відповідальним за створення нових екземплярів `InputData` через спільний інтерфейс:

```python
class GenericInputData:
    def read(self):
        raise NotImplementedError
    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError
```

`generate_inputs` приймає словник із набором параметрів конфігурації, які конкретний підклас `GenericInputData` повинен інтерпретувати. Тут `config` використовується для знаходження директорії зі вхідними файлами:

```python
class PathInputData(GenericInputData):
    ...
    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))
```

Аналогічно допоміжна функція `create_workers` стає частиною класу `GenericWorker`. Тут параметр `input_class`, що має бути підкласом `GenericInputData`, використовується для генерування необхідних вхідних даних. Екземпляри конкретного підкласу `GenericWorker` конструюються за допомогою `cls()` як узагальненого конструктора:

```python
class GenericWorker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None
    def map(self):
        raise NotImplementedError
    def reduce(self, other):
        raise NotImplementedError
    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        for input_data in input_class.generate_inputs(config):
            workers.append(cls(input_data))
        return workers
```

Виклик `input_class.generate_inputs` вище — це поліморфізм класу. Також видно, як виклик `cls()` у `create_workers` надає альтернативний спосіб конструювання об'єктів `GenericWorker` на додаток до безпосереднього використання методу `__init__`.

Ефект на конкретний підклас `GenericWorker` полягає лише у зміні батьківського класу:

```python
class LineCountWorker(GenericWorker):
    ...
```

Нарешті, функцію `mapreduce` можна переписати так, щоб вона була повністю узагальненою, шляхом виклику `create_workers`:

```python
def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)
```

Запуск нового воркера на наборі тестових файлів дає той самий результат, що й стара реалізація. Різниця в тому, що функція `mapreduce` вимагає більше параметрів для узагальненої роботи:

```python
config = {'data_dir': tmpdir}
result = mapreduce(LineCountWorker, PathInputData, config)
print(f'There are {result} lines')
>>>
There are 4360 lines
```

**Що слід запам'ятати**

- Python підтримує лише один конструктор для класу: метод `__init__`.
- Використовуйте `@classmethod` для визначення альтернативних конструкторів для своїх класів.
- Використовуйте поліморфізм методів класу для надання узагальнених способів побудови та з'єднання багатьох конкретних підкласів.

---

## Порада 40. Ініціалізуйте батьківські класи за допомогою super

Старий, простий спосіб ініціалізації батьківського класу з дочірнього — безпосередній виклик методу `__init__` батьківського класу з дочірнім екземпляром:

```python
class MyBaseClass:
    def __init__(self, value):
        self.value = value

class MyChildClass(MyBaseClass):
    def __init__(self):
        MyBaseClass.__init__(self, 5)
```

Цей підхід добре працює для базових ієрархій класів, але ламається в багатьох випадках.

Якщо клас зазнає множинного успадкування (якого загалом слід уникати; см. Порада 41: «Розгляньте компонування функціональності за допомогою mix-in-класів»), безпосередній виклик методів `__init__` суперкласів може призводити до непередбачуваної поведінки.

Одна проблема — порядок виклику `__init__` не визначений для всіх підкласів. Наприклад, тут визначаються два батьківські класи, що оперують полем `value` екземпляра:

```python
class TimesTwo:
    def __init__(self):
        self.value *= 2

class PlusFive:
    def __init__(self):
        self.value += 5
```

Цей клас визначає батьківські класи в одному порядку:

```python
class OneWay(MyBaseClass, TimesTwo, PlusFive):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        TimesTwo.__init__(self)
        PlusFive.__init__(self)

foo = OneWay(5)
print('First ordering value is (5 * 2) + 5 =', foo.value)
>>>
First ordering value is (5 * 2) + 5 = 15
```

А тут інший клас визначає ті самі батьківські класи в іншому порядку, але виклики конструкторів батьківських класів залишилися в попередньому порядку. Результат не відповідає порядку батьківських класів у визначенні:

```python
class AnotherWay(MyBaseClass, PlusFive, TimesTwo):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        TimesTwo.__init__(self)
        PlusFive.__init__(self)

bar = AnotherWay(5)
print('Second ordering value is', bar.value)
>>>
Second ordering value is 15
```

Інша проблема виникає при ромбовидному успадкуванні. Ромбовидне успадкування відбувається, коли підклас успадковує від двох окремих класів, що мають спільний суперклас десь в ієрархії. Воно спричиняє багаторазовий виклик методу `__init__` спільного суперкласу. Наприклад:

```python
class TimesSeven(MyBaseClass):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        self.value *= 7

class PlusNine(MyBaseClass):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        self.value += 9

class ThisWay(TimesSeven, PlusNine):
    def __init__(self, value):
        TimesSeven.__init__(self, value)
        PlusNine.__init__(self, value)

foo = ThisWay(5)
print('Should be (5 * 7) + 9 = 44 but is', foo.value)
>>>
Should be (5 * 7) + 9 = 44 but is 14
```

Виклик конструктора другого батьківського класу, `PlusNine.__init__`, скидає `self.value` назад до 5 при повторному виклику `MyBaseClass.__init__`. Це призводить до обчислення `self.value` як `5 + 9 = 14`, повністю ігноруючи ефект конструктора `TimesSeven.__init__`.

Для вирішення цих проблем Python має вбудовану функцію `super` та стандартний порядок вирішення методів (MRO). `super` гарантує, що спільні суперкласи в ромбових ієрархіях виконуються лише один раз. MRO визначає порядок ініціалізації суперкласів за алгоритмом C3-лінеаризації.

Тут знову створюється ромбоподібна ієрархія класів, але цього разу з використанням `super` для ініціалізації батьківського класу:

```python
class TimesSevenCorrect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value *= 7

class PlusNineCorrect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value += 9

class GoodWay(TimesSevenCorrect, PlusNineCorrect):
    def __init__(self, value):
        super().__init__(value)

foo = GoodWay(5)
print('Should be 7 * (5 + 9) = 98 and is', foo.value)
>>>
Should be 7 * (5 + 9) = 98 and is 98
```

Цей порядок може здатися зворотнім на перший погляд. Чи не повинен `TimesSevenCorrect.__init__` виконуватися першим? Відповідь — ні. Цей порядок відповідає тому, що визначає MRO для цього класу. Порядок MRO доступний через метод класу `mro`:

```python
mro_str = '\n'.join(repr(cls) for cls in GoodWay.mro())
print(mro_str)
>>>
<class '__main__.GoodWay'>
<class '__main__.TimesSevenCorrect'>
<class '__main__.PlusNineCorrect'>
<class '__main__.MyBaseClass'>
<class 'object'>
```

При виклику `GoodWay(5)` він у свою чергу викликає `TimesSevenCorrect.__init__`, який викликає `PlusNineCorrect.__init__`, який викликає `MyBaseClass.__init__`. Досягнувши вершини ромба, всі методи ініціалізації фактично виконують свою роботу у зворотному порядку відносно того, як були викликані їхні функції `__init__`. `MyBaseClass.__init__` присвоює значення `5`. `PlusNineCorrect.__init__` додає `9`, отримуючи `14`. `TimesSevenCorrect.__init__` множить на `7`, отримуючи `98`.

Окрім забезпечення надійності множинного успадкування, виклик `super().__init__` також є значно зручнішим для підтримки, ніж безпосередній виклик `MyBaseClass.__init__` з підкласів.

Функцію `super` також можна викликати з двома параметрами: спочатку тип класу, чий батьківський вигляд MRO потрібно отримати, а потім екземпляр для доступу до цього вигляду. Однак ці параметри не є обов'язковими для ініціалізації екземпляра об'єкта. Компілятор Python автоматично надає коректні параметри (`__class__` та `self`), коли `super` викликається без аргументів у визначенні класу. Це означає, що всі три наступні використання є еквівалентними:

```python
class ExplicitTrisect(MyBaseClass):
    def __init__(self, value):
        super(ExplicitTrisect, self).__init__(value)
        self.value /= 3

class AutomaticTrisect(MyBaseClass):
    def __init__(self, value):
        super(__class__, self).__init__(value)
        self.value /= 3

class ImplicitTrisect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value /= 3

assert ExplicitTrisect(9).value == 3
assert AutomaticTrisect(9).value == 3
assert ImplicitTrisect(9).value == 3
```

Параметри `super` слід надавати лише тоді, коли потрібно отримати доступ до конкретної функціональності реалізації суперкласу з дочірнього класу (наприклад, для загортання або повторного використання функціональності).

**Що слід запам'ятати**

- Стандартний порядок вирішення методів Python (MRO) вирішує проблеми порядку ініціалізації суперкласів та ромбового успадкування.
- Використовуйте вбудовану функцію `super` без аргументів для ініціалізації батьківських класів.

---

## Порада 41. Розгляньте компонування функціональності за допомогою mix-in-класів

Python — об'єктно-орієнтована мова з вбудованими засобами для керованого множинного успадкування (см. Порада 40: «Ініціалізуйте батьківські класи за допомогою super»). Однак краще повністю уникати множинного успадкування.

Якщо потрібна зручність та інкапсуляція, що забезпечуються множинним успадкуванням, але хочеться уникнути потенційних проблем, варто написати mix-in. Mix-in — це клас, що визначає лише невеликий набір додаткових методів для своїх дочірніх класів. Mix-in-класи не визначають власних атрибутів екземпляра і не вимагають виклику свого конструктора `__init__`.

Написання mix-in-ів є простим, оскільки Python дозволяє легко перевіряти поточний стан будь-якого об'єкта незалежно від його типу. Це означає, що можна написати узагальнену функціональність лише один раз, у mix-in, і потім застосовувати її до багатьох інших класів.

Наприклад, потрібна можливість перетворювати Python-об'єкт із його представлення в пам'яті на словник, готовий до серіалізації. Тут визначається приклад mix-in, що реалізує це за допомогою нового публічного методу, що додається до будь-якого класу, що успадковується від нього:

```python
class ToDictMixin:
    def to_dict(self):
        return self._traverse_dict(self.__dict__)
    def _traverse_dict(self, instance_dict):
        output = {}
        for key, value in instance_dict.items():
            output[key] = self._traverse(key, value)
        return output
    def _traverse(self, key, value):
        if isinstance(value, ToDictMixin):
            return value.to_dict()
        elif isinstance(value, dict):
            return self._traverse_dict(value)
        elif isinstance(value, list):
            return [self._traverse(key, i) for i in value]
        elif hasattr(value, '__dict__'):
            return self._traverse_dict(value.__dict__)
        else:
            return value
```

Тут визначається приклад класу, що використовує mix-in для створення словникового представлення бінарного дерева:

```python
class BinaryTree(ToDictMixin):
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right
```

Перетворення великої кількості пов'язаних Python-об'єктів на словник стає простим:

```python
tree = BinaryTree(10,
    left=BinaryTree(7, right=BinaryTree(9)),
    right=BinaryTree(13, left=BinaryTree(11)))
print(tree.to_dict())
>>>
{'value': 10,
 'left': {'value': 7, 'left': None,
          'right': {'value': 9, 'left': None, 'right': None}},
 'right': {'value': 13,
           'left': {'value': 11, 'left': None, 'right': None},
           'right': None}}
```

Найкраще в mix-in-ах — можна зробити їхню узагальнену функціональність підключаємою, щоб поведінку можна було перевизначати при необхідності. Наприклад, визначається підклас `BinaryTree`, що містить посилання на свого батька. Це кругове посилання змусило б реалізацію `ToDictMixin.to_dict` зациклитися назавжди:

```python
class BinaryTreeWithParent(BinaryTree):
    def __init__(self, value, left=None,
                 right=None, parent=None):
        super().__init__(value, left=left, right=right)
        self.parent = parent
```

Рішення — перевизначити метод `BinaryTreeWithParent._traverse` для обробки лише важливих значень і запобігання циклам. Тут перевизначення `_traverse` вставляє числове значення батька, а в іншому випадку делегує реалізації mix-in за замовчуванням через `super`:

```python
    def _traverse(self, key, value):
        if (isinstance(value, BinaryTreeWithParent) and
                key == 'parent'):
            return value.value  # Запобігання циклам
        else:
            return super()._traverse(key, value)
```

Mix-in-и також можна компонувати разом. Наприклад, потрібен mix-in, що надає узагальнену JSON-серіалізацію для будь-якого класу:

```python
import json

class JsonMixin:
    @classmethod
    def from_json(cls, data):
        kwargs = json.loads(data)
        return cls(**kwargs)
    def to_json(self):
        return json.dumps(self.to_dict())
```

Зверніть увагу, що клас `JsonMixin` визначає і методи екземпляра, і методи класу. Mix-in-и дозволяють додавати обидва види поведінки до підкласів. Єдині вимоги до підкласу `JsonMixin` — надати метод `to_dict` і приймати іменовані аргументи для методу `__init__`.

Цей mix-in спрощує створення ієрархій утилітарних класів, що можуть серіалізуватися та десеріалізуватися в JSON з мінімальним шаблонним кодом:

```python
class DatacenterRack(ToDictMixin, JsonMixin):
    def __init__(self, switch=None, machines=None):
        self.switch = Switch(**switch)
        self.machines = [Machine(**kwargs) for kwargs in machines]

class Switch(ToDictMixin, JsonMixin):
    def __init__(self, ports=None, speed=None):
        self.ports = ports
        self.speed = speed

class Machine(ToDictMixin, JsonMixin):
    def __init__(self, cores=None, ram=None, disk=None):
        self.cores = cores
        self.ram = ram
        self.disk = disk
```

Серіалізація цих класів у JSON та назад є простою:

```python
serialized = """{
    "switch": {"ports": 5, "speed": 1e9},
    "machines": [
        {"cores": 8, "ram": 32e9, "disk": 5e12},
        {"cores": 4, "ram": 16e9, "disk": 1e12},
        {"cores": 2, "ram": 4e9,  "disk": 500e9}
    ]
}"""
deserialized = DatacenterRack.from_json(serialized)
roundtrip = deserialized.to_json()
assert json.loads(serialized) == json.loads(roundtrip)
```

**Що слід запам'ятати**

- Уникайте використання множинного успадкування з атрибутами екземпляра та `__init__`, якщо mix-in-класи можуть досягти того самого результату.
- Використовуйте підключаєму поведінку на рівні екземпляра для надання налаштувань для кожного класу, коли mix-in-класи можуть цього вимагати.
- Mix-in-и можуть включати методи екземпляра або методи класу залежно від ваших потреб.
- Компонуйте mix-in-и для створення складної функціональності з простої поведінки.

---

## Порада 42. Надавайте перевагу публічним атрибутам над приватними

У Python є лише два типи видимості атрибутів класу: публічні та приватні:

```python
class MyObject:
    def __init__(self):
        self.public_field = 5
        self.__private_field = 10
    def get_private_field(self):
        return self.__private_field
```

Публічні атрибути доступні будь-кому за допомогою оператора крапки для об'єкта:

```python
foo = MyObject()
assert foo.public_field == 5
```

Приватні поля вказуються шляхом додавання подвійного підкреслення перед іменем атрибута. До них можна безпосередньо звертатися з методів класу-контейнера:

```python
assert foo.get_private_field() == 10
```

Однак безпосередній доступ до приватних полів ззовні класу підніматиме виняток:

```python
foo.__private_field
>>>
Traceback ...
AttributeError: 'MyObject' object has no attribute '__private_field'
```

Методи класу також мають доступ до приватних атрибутів, оскільки вони оголошені всередині блоку оточуючого класу:

```python
class MyOtherObject:
    def __init__(self):
        self.__private_field = 71
    @classmethod
    def get_private_field_of_instance(cls, instance):
        return instance.__private_field

bar = MyOtherObject()
assert MyOtherObject.get_private_field_of_instance(bar) == 71
```

Як і очікується від приватних полів, підклас не може звертатися до приватних полів батьківського класу:

```python
class MyParentObject:
    def __init__(self):
        self.__private_field = 71

class MyChildObject(MyParentObject):
    def get_private_field(self):
        return self.__private_field

baz = MyChildObject()
baz.get_private_field()
>>>
Traceback ...
AttributeError: 'MyChildObject' object has no attribute
'_MyChildObject__private_field'
```

Поведінка приватного атрибута реалізована за допомогою простої трансформації імені атрибута. Коли компілятор Python бачить доступ до приватного атрибута в методах, таких як `MyChildObject.get_private_field`, він транслює доступ до `__private_field` на ім'я `_MyChildObject__private_field`. У прикладі вище `__private_field` визначено лише у `MyParentObject.__init__`, тому справжнє ім'я приватного атрибута — `_MyParentObject__private_field`. Доступ до приватного атрибута батька з дочірнього класу не вдається просто тому, що трансформоване ім'я атрибута не існує.

Знаючи цю схему, можна легко отримати доступ до приватних атрибутів будь-якого класу — з підкласу або ззовні — без дозволу:

```python
assert baz._MyParentObject__private_field == 71
```

Чому синтаксис приватних атрибутів не забезпечує суворої видимості? Найпростіша відповідь — це часто цитоване гасло Python: «Ми всі тут дорослі». Це означає, що мова не потребує захисту від наших власних дій. Python-програмісти вважають, що переваги відкритості — дозвіл на незаплановане розширення класів за замовчуванням — переважують недоліки.

Щоб мінімізувати шкоду від ненавмисного доступу до внутрішніх елементів, Python-програмісти дотримуються угоди іменування, визначеної в настанові зі стилю (см. Порада 2: «Дотримуйтеся настанов стилю PEP 8»). Поля з префіксом одинарного підкреслення (наприклад, `_protected_field`) є захищеними за угодою, тобто зовнішні користувачі класу повинні діяти обережно.

Однак багато програмістів-початківців використовують приватні поля для позначення внутрішнього API, до якого не повинні звертатися підкласи або зовнішній код. Це неправильний підхід. Рано чи пізно хтось — можливо, навіть ви самі — захоче успадкуватися від вашого класу для додавання нової поведінки. Вибравши приватні атрибути, ви лише ускладнюєте перевизначення та розширення підкласів.

Загалом краще схилятися до дозволу підкласам робити більше за допомогою захищених атрибутів. Документуйте кожне захищене поле і пояснюйте, які поля є внутрішніми API, доступними підкласам, а які слід залишити без змін.

Єдиний випадок, коли варто серйозно розглянути використання приватних атрибутів — це коли є занепокоєння щодо конфлікту імен з підкласами. Ця проблема виникає, коли дочірній клас ненавмисно визначає атрибут, вже визначений батьківським класом:

```python
class ApiClass:
    def __init__(self):
        self._value = 5
    def get(self):
        return self._value

class Child(ApiClass):
    def __init__(self):
        super().__init__()
        self._value = 'hello'  # Конфлікт

a = Child()
print(f'{a.get()} and {a._value} should be different')
>>>
hello and hello should be different
```

Це є проблемою насамперед для класів, що є частиною публічного API; підкласи знаходяться поза вашим контролем, тому не можна виконати рефакторинг для виправлення проблеми. Щоб зменшити ризик такого конфлікту, можна використовувати приватний атрибут у батьківському класі, щоб гарантувати відсутність перекриття імен атрибутів із дочірніми класами:

```python
class ApiClass:
    def __init__(self):
        self.__value = 5      # Подвійне підкреслення
    def get(self):
        return self.__value   # Подвійне підкреслення

class Child(ApiClass):
    def __init__(self):
        super().__init__()
        self._value = 'hello'  # ОК!

a = Child()
print(f'{a.get()} and {a._value} are different')
>>>
5 and hello are different
```

**Що слід запам'ятати**

- Приватні атрибути не є суворо забезпеченими компілятором Python.
- Плануйте з самого початку дозволяти підкласам робити більше з вашими внутрішніми API та атрибутами, замість того щоб блокувати до них доступ.
- Використовуйте документацію захищених полів для керування підкласами замість спроб контролювати доступ за допомогою приватних атрибутів.
- Розгляньте використання приватних атрибутів лише для уникнення конфліктів імен із підкласами, що знаходяться поза вашим контролем.

---

## Порада 43. Успадковуйте від collections.abc для власних типів-контейнерів

Велика частина програмування на Python — це визначення класів, що містять дані, і опис того, як такі об'єкти пов'язані між собою. Кожен Python-клас є своєрідним контейнером, що інкапсулює атрибути та функціональність. Python також надає вбудовані типи-контейнери для управління даними: списки, кортежі, множини та словники.

При проєктуванні класів для простих випадків використання, наприклад послідовностей, природно хотіти безпосередньо успадковуватися від вбудованого типу `list`. Наприклад, потрібно створити власний тип списку з додатковими методами для підрахунку частоти елементів:

```python
class FrequencyList(list):
    def __init__(self, members):
        super().__init__(members)
    def frequency(self):
        counts = {}
        for item in self:
            counts[item] = counts.get(item, 0) + 1
        return counts
```

Успадковуючись від `list`, отримуємо всю стандартну функціональність `list` і зберігаємо семантику, знайому всім Python-програмістам:

```python
foo = FrequencyList(['a', 'b', 'a', 'c', 'b', 'a', 'd'])
print('Length is', len(foo))
foo.pop()
print('After pop:', repr(foo))
print('Frequency:', foo.frequency())
>>>
Length is 7
After pop: ['a', 'b', 'a', 'c', 'b', 'a']
Frequency: {'a': 3, 'b': 2, 'c': 1}
```

Тепер уявімо, що потрібен об'єкт, який поводиться як список і дозволяє індексування, але не є підкласом `list`. Наприклад, потрібна семантика послідовності (як у `list` або `tuple`) для класу бінарного дерева:

```python
class BinaryNode:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right
```

Як зробити цей клас таким, що поводиться як тип-послідовність? Python реалізує поведінку контейнера з методами екземпляра зі спеціальними іменами. Коли звертаємося до елемента послідовності за індексом:

```python
bar = [1, 2, 3]
bar[0]
```

це інтерпретується як:

```python
bar.__getitem__(0)
```

Щоб клас `BinaryNode` поводився як послідовність, можна надати власну реалізацію `__getitem__`, що обходить дерево об'єктів у глибину:

```python
class IndexableNode(BinaryNode):
    def _traverse(self):
        if self.left is not None:
            yield from self.left._traverse()
        yield self
        if self.right is not None:
            yield from self.right._traverse()
    def __getitem__(self, index):
        for i, item in enumerate(self._traverse()):
            if i == index:
                return item.value
        raise IndexError(f'Index {index} is out of range')
```

Бінарне дерево можна конструювати як зазвичай, але також звертатися до нього як до списку на додаток до обходу за атрибутами `left` та `right`:

```python
tree = IndexableNode(
    10,
    left=IndexableNode(5,
        left=IndexableNode(2),
        right=IndexableNode(6, right=IndexableNode(7))),
    right=IndexableNode(15, left=IndexableNode(11)))
print('LRR is', tree.left.right.right.value)
print('Index 0 is', tree[0])
print('Index 1 is', tree[1])
print('11 in the tree?', 11 in tree)
print('17 in the tree?', 17 in tree)
print('Tree is', list(tree))
>>>
LRR is 7
Index 0 is 2
Index 1 is 5
11 in the tree? True
17 in the tree? False
Tree is [2, 5, 6, 7, 10, 11, 15]
```

Проблема в тому, що реалізація `__getitem__` не достатня для надання всієї семантики послідовності, яку можна очікувати від екземпляра `list`:

```python
len(tree)
>>>
Traceback ...
TypeError: object of type 'IndexableNode' has no len()
```

Вбудована функція `len` вимагає іншого спеціального методу — `__len__`, що повинен мати реалізацію для власного типу-послідовності:

```python
class SequenceNode(IndexableNode):
    def __len__(self):
        for count, _ in enumerate(self._traverse(), 1):
            pass
        return count
```

На жаль, цього все ще недостатньо для того, щоб клас повністю був валідною послідовністю. Також відсутні методи `count` та `index`, які Python-програміст очікував би побачити у послідовності на кшталт `list` або `tuple`.

Щоб уникнути цієї складності в усій екосистемі Python, вбудований модуль `collections.abc` визначає набір абстрактних базових класів, що надають усі типові методи для кожного типу-контейнера. При успадкуванні від цих абстрактних базових класів та пропуску реалізації необхідних методів, модуль повідомляє про помилку:

```python
from collections.abc import Sequence

class BadType(Sequence):
    pass

foo = BadType()
>>>
Traceback ...
TypeError: Can't instantiate abstract class BadType with
abstract methods __getitem__, __len__
```

При реалізації всіх методів, необхідних абстрактному базовому класу з `collections.abc`, як це зроблено вище з `SequenceNode`, він безкоштовно надає всі додаткові методи, наприклад `index` та `count`:

```python
class BetterNode(SequenceNode, Sequence):
    pass

tree = BetterNode(
    10,
    left=BetterNode(5,
        left=BetterNode(2),
        right=BetterNode(6, right=BetterNode(7))),
    right=BetterNode(15, left=BetterNode(11)))
print('Index of 7 is', tree.index(7))
print('Count of 10 is', tree.count(10))
>>>
Index of 7 is 3
Count of 10 is 1
```

Перевага використання цих абстрактних базових класів ще більша для складніших типів-контейнерів, таких як `Set` та `MutableMapping`, що мають велику кількість спеціальних методів, які потрібно реалізувати для відповідності угодам Python.

**Що слід запам'ятати**

- Успадковуйтеся безпосередньо від вбудованих типів-контейнерів Python (наприклад, `list` або `dict`) для простих випадків використання.
- Будьте уважні до великої кількості методів, необхідних для коректної реалізації власних типів-контейнерів.
- Успадковуйте власні типи-контейнери від інтерфейсів, визначених у `collections.abc`, щоб гарантувати відповідність класів необхідним інтерфейсам та поведінці.
