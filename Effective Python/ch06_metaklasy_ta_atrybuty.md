# Розділ 6. Метакласи та атрибути

Метакласи часто згадуються в переліках можливостей Python, але мало хто розуміє, що вони реально забезпечують на практиці. Назва «метаклас» невиразно натякає на концепцію вище і за межами класу. Просто кажучи, метакласи дозволяють перехоплювати оператор `class` у Python і забезпечувати особливу поведінку при кожному визначенні класу.

Не менш загадковими та потужними є вбудовані можливості Python для динамічного налаштування доступу до атрибутів. Разом із об'єктно-орієнтованими конструкціями Python ці засоби надають чудові інструменти для полегшення переходу від простих класів до складних.

Однак із цими можливостями пов'язані численні пастки. Динамічні атрибути дозволяють перевизначати об'єкти та спричиняти несподівані побічні ефекти. Метакласи можуть створювати надзвичайно дивну поведінку, недоступну для розуміння початківців. Важливо дотримуватися принципу найменшої несподіванки і використовувати ці механізми лише для реалізації добре зрозумілих ідіом.

---

## Порада 44. Використовуйте прості атрибути замість методів setter та getter

Програмісти, що переходять на Python з інших мов, можуть природно намагатися реалізовувати явні методи getter та setter у своїх класах:

```python
class OldResistor:
    def __init__(self, ohms):
        self._ohms = ohms
    def get_ohms(self):
        return self._ohms
    def set_ohms(self, ohms):
        self._ohms = ohms
```

Використання таких setter-ів та getter-ів є простим, але не Pythonic:

```python
r0 = OldResistor(50e3)
print('Before:', r0.get_ohms())
r0.set_ohms(10e3)
print('After: ', r0.get_ohms())
>>>
Before: 50000.0
After:  10000.0
```

Такі методи особливо незручні для операцій типу «збільшення на місці»:

```python
r0.set_ohms(r0.get_ohms() - 4e3)
assert r0.get_ohms() == 6e3
```

Ці утилітарні методи, однак, допомагають визначити інтерфейс класу, полегшуючи інкапсуляцію функціональності, перевірку використання та визначення меж. Це важливі цілі при проєктуванні класу, що забезпечують відсутність порушень у викликаючому коді в процесі еволюції класу.

У Python, однак, ніколи не потрібно реалізовувати явні методи setter або getter. Натомість слід завжди починати реалізацію з простих публічних атрибутів:

```python
class Resistor:
    def __init__(self, ohms):
        self.ohms = ohms
        self.voltage = 0
        self.current = 0

r1 = Resistor(50e3)
r1.ohms = 10e3
```

Такі атрибути роблять операції типу «збільшення на місці» природними та зрозумілими:

```python
r1.ohms += 5e3
```

Пізніше, якщо виникає потреба у спеціальній поведінці при встановленні атрибута, можна перейти до декоратора `@property` (см. Порада 26: «Визначайте декоратори функцій за допомогою functools.wraps») та відповідного атрибута setter. Тут визначається новий підклас `Resistor`, що дозволяє змінювати струм шляхом присвоєння властивості voltage. Зверніть увагу: для коректної роботи цього коду імена методів setter і getter повинні збігатися з іменем передбаченої властивості:

```python
class VoltageResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
        self._voltage = 0
    @property
    def voltage(self):
        return self._voltage
    @voltage.setter
    def voltage(self, voltage):
        self._voltage = voltage
        self.current = self._voltage / self.ohms
```

Тепер присвоєння властивості voltage запускатиме метод voltage setter, який у свою чергу оновлюватиме атрибут current об'єкта:

```python
r2 = VoltageResistance(1e3)
print(f'Before: {r2.current:.2f} amps')
r2.voltage = 10
print(f'After: {r2.current:.2f} amps')
>>>
Before: 0.00 amps
After:  0.01 amps
```

Вказівка setter для властивості також дозволяє виконувати перевірку типів та валідацію значень, що передаються класу. Тут визначається клас, що гарантує, що всі значення опору перевищують нуль Ом:

```python
class BoundedResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
    @property
    def ohms(self):
        return self._ohms
    @ohms.setter
    def ohms(self, ohms):
        if ohms <= 0:
            raise ValueError(f'ohms must be > 0; got {ohms}')
        self._ohms = ohms
```

Присвоєння некоректного значення опору атрибуту тепер підніматиме виняток:

```python
r3 = BoundedResistance(1e3)
r3.ohms = 0
>>>
Traceback ...
ValueError: ohms must be > 0; got 0
```

Виняток також підіймається, якщо передати некоректне значення в конструктор:

```python
BoundedResistance(-5)
>>>
Traceback ...
ValueError: ohms must be > 0; got -5
```

Це відбувається тому, що `BoundedResistance.__init__` викликає `Resistor.__init__`, який присвоює `self.ohms = -5`. Це присвоєння спричиняє виклик методу `@ohms.setter` з `BoundedResistance`, і він негайно виконує код валідації ще до завершення конструювання об'єкта.

Можна навіть використовувати `@property` для забезпечення незмінності атрибутів батьківських класів:

```python
class FixedResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
    @property
    def ohms(self):
        return self._ohms
    @ohms.setter
    def ohms(self, ohms):
        if hasattr(self, '_ohms'):
            raise AttributeError("Ohms is immutable")
        self._ohms = ohms
```

Спроба присвоєння властивості після конструювання підніматиме виняток:

```python
r4 = FixedResistance(1e3)
r4.ohms = 2e3
>>>
Traceback ...
AttributeError: Ohms is immutable
```

При використанні методів `@property` для реалізації setter-ів та getter-ів переконайтеся, що реалізована поведінка не є несподіваною. Наприклад, не слід встановлювати інші атрибути в методах getter властивості:

```python
class MysteriousResistor(Resistor):
    @property
    def ohms(self):
        self.voltage = self._ohms * self.current
        return self._ohms
    @ohms.setter
    def ohms(self, ohms):
        self._ohms = ohms
```

Встановлення інших атрибутів у методах getter властивості призводить до надзвичайно дивної поведінки:

```python
r7 = MysteriousResistor(10)
r7.current = 0.01
print(f'Before: {r7.voltage:.2f}')
r7.ohms
print(f'After: {r7.voltage:.2f}')
>>>
Before: 0.00
After:  0.10
```

Найкраща практика — змінювати лише пов'язаний стан об'єкта в методах `@property.setter`. Також слід уникати будь-яких інших побічних ефектів, яких викликаючий код може не очікувати: динамічного імпорту модулів, виконання повільних допоміжних функцій, операцій введення-виведення або дорогих запитів до бази даних. Користувачі класу очікуватимуть, що його атрибути поводитимуться як будь-який інший Python-об'єкт: швидко та просто. Для будь-чого складнішого або повільнішого використовуйте звичайні методи.

Найбільший недолік `@property` — методи для атрибута можуть спільно використовуватися лише підкласами. Непов'язані класи не можуть використовувати одну й ту саму реалізацію. Однак Python також підтримує дескриптори (см. Порада 46: «Використовуйте дескриптори для повторно використовуваних методів @property»), що забезпечують можливість повторного використання логіки властивостей та інші сценарії.

**Що слід запам'ятати**

- Визначайте нові інтерфейси класів за допомогою простих публічних атрибутів і уникайте визначення методів setter та getter.
- Використовуйте `@property` для визначення особливої поведінки при доступі до атрибутів об'єктів, якщо це необхідно.
- Дотримуйтеся принципу найменшої несподіванки та уникайте дивних побічних ефектів у методах `@property`.
- Забезпечте швидкодію методів `@property`; для повільної або складної роботи — особливо пов'язаної з введенням-виведенням або побічними ефектами — використовуйте замість них звичайні методи.

---

## Порада 45. Розгляньте @property замість рефакторингу атрибутів

Вбудований декоратор `@property` спрощує роботу простих звернень до атрибутів екземпляра (см. Порада 44: «Використовуйте прості атрибути замість методів setter та getter»). Одне розширене, але поширене застосування `@property` — перетворення колись простого числового атрибута на обчислення на льоту. Це надзвичайно корисно, оскільки дозволяє мігрувати всі існуючі варіанти використання класу до нової поведінки без необхідності переписувати код у точках виклику. `@property` також надає важливий тимчасовий засіб для покращення інтерфейсів з часом.

Наприклад, потрібно реалізувати квоту «текучого відра» за допомогою звичайних Python-об'єктів. Тут клас `Bucket` представляє обсяг квоти, що залишилася, і тривалість, протягом якої квота буде доступна:

```python
from datetime import datetime, timedelta

class Bucket:
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.quota = 0
    def __repr__(self):
        return f'Bucket(quota={self.quota})'
```

Алгоритм «текучого відра» гарантує, що при заповненні відра обсяг квоти не переноситься з одного часового проміжку на наступний:

```python
def fill(bucket, amount):
    now = datetime.now()
    if (now - bucket.reset_time) > bucket.period_delta:
        bucket.quota = 0
        bucket.reset_time = now
    bucket.quota += amount
```

Кожного разу, коли споживач квоти хоче щось зробити, він спочатку повинен переконатися, що може вирахувати необхідний обсяг квоти:

```python
def deduct(bucket, amount):
    now = datetime.now()
    if (now - bucket.reset_time) > bucket.period_delta:
        return False  # Відро не заповнювалося у цьому проміжку
    if bucket.quota - amount < 0:
        return False  # Відро заповнено, але недостатньо
    bucket.quota -= amount
    return True       # Відро мало достатньо, квота витрачена
```

Проблема цієї реалізації: невідомо, яким був початковий рівень квоти відра. Квота зменшується протягом часового проміжку до нуля. У цей момент `deduct` завжди повертатиме `False`, поки відро не поповниться. Коли це відбувається, корисно знати: чи виклики `deduct` блокуються через те, що квота відра вичерпалася, чи тому, що відро взагалі не мало квоти в цьому проміжку.

Щоб виправити це, можна змінити клас для відстеження `max_quota`, виданої за цей проміжок, і `quota_consumed` за цей проміжок:

```python
class NewBucket:
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.max_quota = 0
        self.quota_consumed = 0
    def __repr__(self):
        return (f'NewBucket(max_quota={self.max_quota}, '
                f'quota_consumed={self.quota_consumed})')
    @property
    def quota(self):
        return self.max_quota - self.quota_consumed
    @quota.setter
    def quota(self, amount):
        delta = self.max_quota - amount
        if amount == 0:
            # Квота скидається для нового проміжку
            self.quota_consumed = 0
            self.max_quota = 0
        elif delta < 0:
            # Квота поповнюється для нового проміжку
            assert self.quota_consumed == 0
            self.max_quota = amount
        else:
            # Квота витрачається протягом проміжку
            assert self.max_quota >= self.quota_consumed
            self.quota_consumed += delta
```

Повторний запуск демонстраційного коду дає ті самі результати:

```python
bucket = NewBucket(60)
print('Initial', bucket)
fill(bucket, 100)
print('Filled', bucket)
if deduct(bucket, 99):
    print('Had 99 quota')
else:
    print('Not enough for 99 quota')
print('Now', bucket)
if deduct(bucket, 3):
    print('Had 3 quota')
else:
    print('Not enough for 3 quota')
print('Still', bucket)
>>>
Initial NewBucket(max_quota=0, quota_consumed=0)
Filled NewBucket(max_quota=100, quota_consumed=0)
Had 99 quota
Now NewBucket(max_quota=100, quota_consumed=99)
Not enough for 3 quota
Still NewBucket(max_quota=100, quota_consumed=99)
```

Найкраще те, що код, який використовує `Bucket.quota`, не повинен змінюватися або знати про зміни в класі. `@property` — це інструмент для вирішення проблем, із якими доводиться стикатися в реальному коді. Не зловживайте ним. Коли виявляєте, що повторно розширюєте методи `@property`, настав час виконати рефакторинг класу.

**Що слід запам'ятати**

- Використовуйте `@property` для надання новою функціональністю існуючих атрибутів екземпляра.
- Поступово вдосконалюйте моделі даних за допомогою `@property`.
- Розгляньте рефакторинг класу та всіх точок виклику, коли виявляєте надмірне використання `@property`.

---

## Порада 46. Використовуйте дескриптори для повторно використовуваних методів @property

Велика проблема з вбудованим `@property` (см. Порада 44 та Порада 45) — повторне використання. Декоровані ним методи не можуть повторно використовуватися для кількох атрибутів одного класу. Вони також не можуть повторно використовуватися непов'язаними класами.

Наприклад, потрібен клас для валідації того, що оцінка за домашнє завдання є відсотком:

```python
class Homework:
    def __init__(self):
        self._grade = 0
    @property
    def grade(self):
        return self._grade
    @grade.setter
    def grade(self, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._grade = value
```

Тепер потрібно також виставляти оцінку студенту за іспит, де іспит складається з кількох предметів, кожен із окремою оцінкою:

```python
class Exam:
    def __init__(self):
        self._writing_grade = 0
        self._math_grade = 0
    @staticmethod
    def _check_grade(value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
    @property
    def writing_grade(self):
        return self._writing_grade
    @writing_grade.setter
    def writing_grade(self, value):
        self._check_grade(value)
        self._writing_grade = value
    @property
    def math_grade(self):
        return self._math_grade
    @math_grade.setter
    def math_grade(self, value):
        self._check_grade(value)
        self._math_grade = value
```

Це швидко стає нудним. Для кожного розділу іспиту потрібно додавати новий `@property` і пов'язану з ним валідацію. Якщо потрібно повторно використовувати цю валідацію відсотків в інших класах, доведеться щоразу писати шаблонний код `@property` та метод `_check_grade`.

Кращим способом у Python є використання дескриптора. Протокол дескриптора визначає, як мова інтерпретує доступ до атрибутів. Клас дескриптора може надавати методи `__get__` та `__set__`, що дозволяють повторно використовувати поведінку валідації оцінок без шаблонного коду. Для цього дескриптори є кращими від mix-in-ів (см. Порада 41), оскільки дозволяють повторно використовувати одну й ту саму логіку для багатьох різних атрибутів одного класу.

Тут визначається новий клас `Exam` з атрибутами класу, що є екземплярами `Grade`. Клас `Grade` реалізує протокол дескриптора:

```python
class Grade:
    def __get__(self, instance, instance_type):
        ...
    def __set__(self, instance, value):
        ...

class Exam:
    # Атрибути класу
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()
```

При присвоєнні властивості:

```python
exam = Exam()
exam.writing_grade = 40
```

це інтерпретується як:

```python
Exam.__dict__['writing_grade'].__set__(exam, 40)
```

При отриманні властивості:

```python
exam.writing_grade
```

це інтерпретується як:

```python
Exam.__dict__['writing_grade'].__get__(exam, Exam)
```

Ця поведінка зумовлена методом `__getattribute__` об'єкта (см. Порада 47). Коли екземпляр `Exam` не має атрибута з іменем `writing_grade`, Python звертається до атрибута класу `Exam`. Якщо цей атрибут класу — об'єкт із методами `__get__` та `__set__`, Python вважає, що потрібно дотримуватися протоколу дескриптора.

Перша спроба реалізації дескриптора `Grade`:

```python
class Grade:
    def __init__(self):
        self._value = 0
    def __get__(self, instance, instance_type):
        return self._value
    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._value = value
```

На жаль, це неправильно і призводить до хибної поведінки. Звернення до кількох атрибутів одного екземпляра `Exam` працює як очікується, але звернення до цих атрибутів для кількох екземплярів `Exam` спричиняє несподівану поведінку:

```python
second_exam = Exam()
second_exam.writing_grade = 75
print(f'Second {second_exam.writing_grade} is right')
print(f'First {first_exam.writing_grade} is wrong; should be 82')
>>>
Second 75 is right
First 75 is wrong; should be 82
```

Проблема в тому, що один екземпляр `Grade` спільно використовується всіма екземплярами `Exam` для атрибута класу `writing_grade`. Для вирішення цього клас `Grade` повинен відстежувати своє значення для кожного унікального екземпляра `Exam`. Це можна зробити, зберігаючи стан кожного екземпляра у словнику:

```python
class Grade:
    def __init__(self):
        self._values = {}
    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return self._values.get(instance, 0)
    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._values[instance] = value
```

Ця реалізація є простою і добре працює, але є одна пастка: витік пам'яті. Словник `_values` зберігає посилання на кожен екземпляр `Exam`, коли-небудь переданий у `__set__` протягом існування програми. Це призводить до того, що лічильник посилань екземплярів ніколи не досягає нуля, запобігаючи їх очищенню збирачем сміття (см. Порада 81: «Використовуйте tracemalloc для розуміння використання пам'яті та витоків»).

Щоб виправити це, можна використати вбудований модуль Python `weakref`. Цей модуль надає спеціальний клас `WeakKeyDictionary`, що може замінити простий словник для `_values`. Особлива поведінка `WeakKeyDictionary` полягає в тому, що він видаляє екземпляри `Exam` зі свого набору елементів, коли Python runtime знає, що зберігає останнє посилання на екземпляр у програмі:

```python
from weakref import WeakKeyDictionary

class Grade:
    def __init__(self):
        self._values = WeakKeyDictionary()
    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return self._values.get(instance, 0)
    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._values[instance] = value
```

Використовуючи цю реалізацію дескриптора `Grade`, все працює як очікується:

```python
class Exam:
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

first_exam = Exam()
first_exam.writing_grade = 82
second_exam = Exam()
second_exam.writing_grade = 75
print(f'First {first_exam.writing_grade} is right')
print(f'Second {second_exam.writing_grade} is right')
>>>
First 82 is right
Second 75 is right
```

**Що слід запам'ятати**

- Повторно використовуйте поведінку та валідацію методів `@property`, визначаючи власні класи дескрипторів.
- Використовуйте `WeakKeyDictionary`, щоб гарантувати відсутність витоків пам'яті у класах дескрипторів.
- Не заглиблюйтеся надмірно в розуміння того, як саме `__getattribute__` використовує протокол дескриптора для отримання та встановлення атрибутів.

---

## Порада 47. Використовуйте \_\_getattr\_\_, \_\_getattribute\_\_ та \_\_setattr\_\_ для «ледачих» атрибутів

Hook-и об'єктів Python спрощують написання узагальненого коду для з'єднання систем. Наприклад, потрібно представити записи в базі даних як Python-об'єкти. База даних вже має встановлену схему. Код, що використовує об'єкти, що відповідають цим записам, повинен також знати, як виглядає база даних. Однак у Python код, що з'єднує Python-об'єкти з базою даних, не потребує явного визначення схеми записів — він може бути узагальненим.

Як це можливо? Звичайні атрибути екземпляра, методи `@property` та дескриптори не можуть цього зробити, оскільки всі вони повинні бути визначені заздалегідь. Python робить цю динамічну поведінку можливою за допомогою спеціального методу `__getattr__`. Якщо клас визначає `__getattr__`, цей метод викликається щоразу, коли атрибут не може бути знайдений у словнику екземпляра об'єкта:

```python
class LazyRecord:
    def __init__(self):
        self.exists = 5
    def __getattr__(self, name):
        value = f'Value for {name}'
        setattr(self, name, value)
        return value
```

Тут звертаємося до відсутньої властивості `foo`. Це змушує Python викликати метод `__getattr__` вище, який змінює словник екземпляра `__dict__`:

```python
data = LazyRecord()
print('Before:', data.__dict__)
print('foo:   ', data.foo)
print('After: ', data.__dict__)
>>>
Before: {'exists': 5}
foo:    Value for foo
After:  {'exists': 5, 'foo': 'Value for foo'}
```

Тут до `LazyRecord` додається журналювання для демонстрації того, коли насправді викликається `__getattr__`. Зверніть увагу, як викликається `super().__getattr__()` для використання реалізації `__getattr__` суперкласу для отримання реального значення властивості та уникнення нескінченної рекурсії (см. Порада 40: «Ініціалізуйте батьківські класи за допомогою super»):

```python
class LoggingLazyRecord(LazyRecord):
    def __getattr__(self, name):
        print(f'* Called __getattr__({name!r}), '
              f'populating instance dictionary')
        result = super().__getattr__(name)
        print(f'* Returning {result!r}')
        return result

data = LoggingLazyRecord()
print('exists:    ', data.exists)
print('First foo: ', data.foo)
print('Second foo:', data.foo)
>>>
exists:     5
* Called __getattr__('foo'), populating instance dictionary
* Returning 'Value for foo'
First foo:  Value for foo
Second foo: Value for foo
```

Атрибут `exists` присутній у словнику екземпляра, тому `__getattr__` для нього ніколи не викликається. Атрибут `foo` спочатку відсутній у словнику екземпляра, тому `__getattr__` викликається вперше. Але виклик `__getattr__` для `foo` також виконує `setattr`, що поповнює `foo` у словнику екземпляра. Ось чому другого разу при доступі до `foo` виклик `__getattr__` не журналюється.

Така поведінка особливо корисна для сценаріїв типу «ледачого» доступу до безсхемних даних. `__getattr__` виконується один раз для складної роботи з завантаження властивості; всі наступні звернення отримують вже наявний результат.

Припустимо, також потрібні транзакції в цій системі баз даних: при наступному зверненні користувача до властивості потрібно знати, чи відповідний запис у базі даних ще є дійсним і чи транзакція ще відкрита. Hook `__getattr__` не дозволить надійно це зробити, оскільки він використовуватиме словник екземпляра об'єкта як швидкий шлях для наявних атрибутів.

Для цього більш розширеного сценарію Python має інший hook об'єкта — `__getattribute__`. Цей спеціальний метод викликається кожного разу при зверненні до будь-якого атрибута об'єкта, навіть якщо він існує в словнику атрибутів. Тут визначається `ValidatingRecord` для журналювання кожного виклику `__getattribute__`:

```python
class ValidatingRecord:
    def __init__(self):
        self.exists = 5
    def __getattribute__(self, name):
        print(f'* Called __getattribute__({name!r})')
        try:
            value = super().__getattribute__(name)
            print(f'* Found {name!r}, returning {value!r}')
            return value
        except AttributeError:
            value = f'Value for {name}'
            print(f'* Setting {name!r} to {value!r}')
            setattr(self, name, value)
            return value
```

У разі, якщо динамічно доступна властивість не повинна існувати, можна підняти `AttributeError`, щоб спричинити стандартну поведінку відсутності властивості Python як для `__getattr__`, так і для `__getattribute__`:

```python
class MissingPropertyRecord:
    def __getattr__(self, name):
        if name == 'bad_name':
            raise AttributeError(f'{name} is missing')
        ...
```

Код Python, що реалізує узагальнену функціональність, часто покладається на вбудовану функцію `hasattr` для визначення наявності властивостей та вбудовану функцію `getattr` для отримання їхніх значень. Ці функції також шукають ім'я атрибута у словнику екземпляра, перш ніж викликати `__getattr__`. У прикладі вище `__getattr__` викликається лише один раз. Навпаки, класи, що реалізують `__getattribute__`, мають цей метод, що викликається кожного разу при використанні `hasattr` або `getattr` з екземпляром.

Тепер потрібно «ледаче» зберігати дані назад у базу даних при присвоєнні значень Python-об'єкту. Це можна зробити за допомогою `__setattr__` — аналогічного hook-а об'єкта, що дозволяє перехоплювати довільні присвоєння атрибутів. На відміну від отримання атрибута через `__getattr__` та `__getattribute__`, тут не потрібні два окремі методи. Метод `__setattr__` завжди викликається при кожному присвоєнні атрибута екземпляра (безпосередньо або через вбудовану функцію `setattr`):

```python
class SavingRecord:
    def __setattr__(self, name, value):
        # Збереження деяких даних для запису
        ...
        super().__setattr__(name, value)
```

Тут визначається журналюючий підклас `SavingRecord`. Його метод `__setattr__` завжди викликається при кожному присвоєнні атрибута:

```python
class LoggingSavingRecord(SavingRecord):
    def __setattr__(self, name, value):
        print(f'* Called __setattr__({name!r}, {value!r})')
        super().__setattr__(name, value)

data = LoggingSavingRecord()
print('Before: ', data.__dict__)
data.foo = 5
print('After:  ', data.__dict__)
data.foo = 7
print('Finally:', data.__dict__)
>>>
Before:  {}
* Called __setattr__('foo', 5)
After:   {'foo': 5}
* Called __setattr__('foo', 7)
Finally: {'foo': 7}
```

Проблема з `__getattribute__` та `__setattr__` полягає в тому, що вони викликаються при кожному зверненні до атрибута об'єкта, навіть коли це не потрібно. Наприклад, якщо потрібно, щоб звернення до атрибутів об'єкта фактично шукали ключі у пов'язаному словнику — при спробі реалізувати це напряму виникне нескінченна рекурсія через ланцюг `__getattribute__` → `self._data` → `__getattribute__` → ...

Рішення — використовувати метод `super().__getattribute__` для отримання значень зі словника атрибутів екземпляра. Це дозволяє уникнути рекурсії:

```python
class DictionaryRecord:
    def __init__(self, data):
        self._data = data
    def __getattribute__(self, name):
        print(f'* Called __getattribute__({name!r})')
        data_dict = super().__getattribute__('_data')
        return data_dict[name]

data = DictionaryRecord({'foo': 3})
print('foo: ', data.foo)
>>>
* Called __getattribute__('foo')
foo:  3
```

Методи `__setattr__`, що модифікують атрибути об'єкта, також повинні відповідно використовувати `super().__setattr__`.

**Що слід запам'ятати**

- Використовуйте `__getattr__` та `__setattr__` для «ледачого» завантаження та збереження атрибутів об'єкта.
- Розумійте, що `__getattr__` викликається лише при зверненні до відсутнього атрибута, тоді як `__getattribute__` викликається при кожному зверненні до будь-якого атрибута.
- Уникайте нескінченної рекурсії в `__getattribute__` та `__setattr__`, використовуючи методи `super()` (тобто клас `object`) для доступу до атрибутів екземпляра.

---

## Порада 48. Перевіряйте підкласи за допомогою \_\_init\_subclass\_\_

Одне з найпростіших застосувань метакласів — перевірка коректності визначення класу. При побудові складної ієрархії класів може знадобитися забезпечення дотримання стилю, вимога перевизначення методів або наявність суворих відносин між атрибутами класу. Метакласи забезпечують ці сценарії, надаючи надійний спосіб виконувати код валідації при кожному визначенні нового підкласу.

Часто код валідації класу виконується в методі `__init__`, коли об'єкт типу класу конструюється під час виконання (см. Порада 44). Використання метакласів для валідації дозволяє підіймати помилки значно раніше — наприклад, коли модуль із класом вперше імпортується при запуску програми.

Перед описом того, як визначити метаклас для валідації підкласів, важливо зрозуміти дію метакласу для стандартних об'єктів. Метаклас визначається успадкуванням від `type`. У типовому випадку метаклас отримує вміст пов'язаних операторів `class` у своєму методі `__new__`. Тут можна перевіряти та модифікувати інформацію про клас перед тим, як тип буде фактично сконструйований:

```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        print(f'* Running {meta}.__new__ for {name}')
        print('Bases:', bases)
        print(class_dict)
        return type.__new__(meta, name, bases, class_dict)

class MyClass(metaclass=Meta):
    stuff = 123
    def foo(self):
        pass

class MySubclass(MyClass):
    other = 567
    def bar(self):
        pass
```

Метаклас має доступ до імені класу, батьківських класів (bases), від яких він успадковується, та всіх атрибутів класу, визначених у тілі класу. Усі класи успадковуються від `object`, тому він явно не вказаний у кортежі базових класів.

Наприклад, потрібно представити будь-який тип багатокутника. Можна визначити спеціальний перевіряючий метаклас і використати його в базовому класі ієрархії класів многокутників:

```python
class ValidatePolygon(type):
    def __new__(meta, name, bases, class_dict):
        # Перевіряти лише підкласи Polygon
        if bases:
            if class_dict['sides'] < 3:
                raise ValueError('Polygons need 3+ sides')
        return type.__new__(meta, name, bases, class_dict)

class Polygon(metaclass=ValidatePolygon):
    sides = None  # Повинно бути вказано підкласами
    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class Triangle(Polygon):
    sides = 3

class Rectangle(Polygon):
    sides = 4

class Nonagon(Polygon):
    sides = 9

assert Triangle.interior_angles() == 180
assert Rectangle.interior_angles() == 360
assert Nonagon.interior_angles() == 1260
```

Це виглядає як доволі складний механізм для такого базового завдання. На щастя, Python 3.6 ввів спрощений синтаксис — спеціальний метод класу `__init_subclass__` — для досягнення тієї самої поведінки без метакласів. Тут цей механізм використовується для забезпечення того самого рівня валідації:

```python
class BetterPolygon:
    sides = None  # Повинно бути вказано підкласами
    def __init_subclass__(cls):
        super().__init_subclass__()
        if cls.sides < 3:
            raise ValueError('Polygons need 3+ sides')
    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class Hexagon(BetterPolygon):
    sides = 6

assert Hexagon.interior_angles() == 720
```

Код значно коротший, і метаклас `ValidatePolygon` повністю відсутній. Також простіше стежити за кодом, оскільки тепер можна безпосередньо звертатися до атрибута `sides` на екземплярі `cls` у `__init_subclass__`, замість того щоб занурюватися у словник класу через `class_dict['sides']`.

Ще одна проблема стандартного механізму метакласів Python — можна вказати лише один метаклас для визначення класу. Якщо потрібно поєднати метакласи для валідації кількох аспектів, виникає конфлікт метакласів:

```python
class RedPentagon(Filled, Polygon):
    color = 'red'
    sides = 5
>>>
Traceback ...
TypeError: metaclass conflict: ...
```

Метод `__init_subclass__` також може вирішити цю проблему. Він може бути визначений кількома рівнями ієрархії класів за умови використання вбудованої функції `super` для виклику будь-яких батьківських або сусідніх визначень `__init_subclass__`. Він навіть сумісний із множинним успадкуванням. Тут визначається клас для представлення кольору заливки регіону, що може компонуватися з класом `BetterPolygon`:

```python
class Filled:
    color = None  # Повинно бути вказано підкласами
    def __init_subclass__(cls):
        super().__init_subclass__()
        if cls.color not in ('red', 'green', 'blue'):
            raise ValueError('Fills need a valid color')
```

Тепер можна успадкуватися від обох класів для визначення нового класу. Обидва класи викликають `super().__init_subclass__()`, що змушує відповідну логіку валідації виконуватися при створенні підкласу:

```python
class RedTriangle(Filled, BetterPolygon):
    color = 'red'
    sides = 3

ruddy = RedTriangle()
assert isinstance(ruddy, Filled)
assert isinstance(ruddy, BetterPolygon)
```

Можна навіть використовувати `__init_subclass__` у складних випадках, як-от ромбове успадкування:

```python
class Top:
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f'Top for {cls}')

class Left(Top):
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f'Left for {cls}')

class Right(Top):
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f'Right for {cls}')

class Bottom(Left, Right):
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f'Bottom for {cls}')
>>>
Top for <class '__main__.Left'>
Top for <class '__main__.Right'>
Top for <class '__main__.Bottom'>
Right for <class '__main__.Bottom'>
Left for <class '__main__.Bottom'>
```

Як і очікується, `Top.__init_subclass__` викликається лише один раз для кожного класу, навіть якщо для класу `Bottom` існують два шляхи до нього через батьківські класи `Left` і `Right`.

**Що слід запам'ятати**

- Метод `__new__` метакласів виконується після обробки всього тіла оператора `class`.
- Метакласи можна використовувати для перевірки або модифікації класу після його визначення, але до його створення; однак вони часто є складнішими, ніж потрібно.
- Використовуйте `__init_subclass__` для забезпечення коректності підкласів під час їхнього визначення, ще до конструювання об'єктів їхнього типу.
- Обов'язково викликайте `super().__init_subclass__` у визначенні `__init_subclass__` вашого класу для забезпечення валідації в кількох рівнях класів і множинному успадкуванні.

---

## Порада 49. Реєструйте існування класу за допомогою \_\_init\_subclass\_\_

Ще одне поширене використання метакласів — автоматична реєстрація типів у програмі. Реєстрація корисна для зворотного пошуку: коли потрібно зіставити простий ідентифікатор із відповідним класом.

Наприклад, потрібно реалізувати власне серіалізоване представлення Python-об'єкта за допомогою JSON. Тут визначається базовий клас, що записує параметри конструктора і перетворює їх на JSON-словник:

```python
import json

class BetterSerializable:
    def __init__(self, *args):
        self.args = args
    def serialize(self):
        return json.dumps({
            'class': self.__class__.__name__,
            'args': self.args,
        })
    def __repr__(self):
        name = self.__class__.__name__
        args_str = ', '.join(str(x) for x in self.args)
        return f'{name}({args_str})'
```

Потім підтримується відображення імен класів назад на конструктори цих об'єктів. Узагальнена функція десеріалізації працює для будь-яких класів, переданих у `register_class`:

```python
registry = {}

def register_class(target_class):
    registry[target_class.__name__] = target_class

def deserialize(data):
    params = json.loads(data)
    name = params['class']
    target_class = registry[name]
    return target_class(*params['args'])
```

Щоб `deserialize` завжди працювала коректно, потрібно викликати `register_class` для кожного класу, який може знадобитися десеріалізувати в майбутньому. Проблема в тому, що можна забути викликати `register_class`:

```python
class Point3D(BetterSerializable):
    def __init__(self, x, y, z):
        super().__init__(x, y, z)
        self.x = x
        self.y = y
        self.z = z
# Забули викликати register_class! Ой!
```

Це спричиняє помилку під час виконання:

```python
point = Point3D(5, 9, -4)
data = point.serialize()
deserialize(data)
>>>
Traceback ...
KeyError: 'Point3D'
```

Кращий підхід — використати `__init_subclass__`, що викликається автоматично і усуває можливість забути реєстрацію:

```python
class BetterRegisteredSerializable(BetterSerializable):
    def __init_subclass__(cls):
        super().__init_subclass__()
        register_class(cls)

class Vector1D(BetterRegisteredSerializable):
    def __init__(self, magnitude):
        super().__init__(magnitude)
        self.magnitude = magnitude

before = Vector1D(6)
print('Before:    ', before)
data = before.serialize()
print('Serialized:', data)
print('After:     ', deserialize(data))
>>>
Before:     Vector1D(6)
Serialized: {"class": "Vector1D", "args": [6]}
After:      Vector1D(6)
```

Використовуючи `__init_subclass__` (або метакласи) для реєстрації класів, можна гарантувати, що клас ніколи не буде пропущений за умови правильної ієрархії успадкування.

**Що слід запам'ятати**

- Реєстрація класів — корисний шаблон для побудови модульних Python-програм.
- Метакласи дозволяють автоматично виконувати код реєстрації при кожному успадкуванні від базового класу в програмі.
- Використання метакласів для реєстрації класів допомагає уникати помилок, гарантуючи, що виклик реєстрації ніколи не буде пропущений.
- Надавайте перевагу `__init_subclass__` над стандартним механізмом метакласів, оскільки він є зрозумілішим і простішим для розуміння початківцями.

---

## Порада 50. Анотуйте атрибути класу за допомогою \_\_set\_name\_\_

Ще одна корисна можливість, що забезпечується метакласами, — здатність модифікувати або анотувати властивості після визначення класу, але до його фактичного використання. Цей підхід зазвичай використовується з дескрипторами (см. Порада 46) для надання їм більшої інтроспекції щодо того, як вони використовуються у своєму класі-контейнері.

Наприклад, потрібно визначити новий клас для представлення рядка в базі даних клієнтів із відповідною властивістю для кожного стовпця таблиці. Тут визначається клас дескриптора для з'єднання атрибутів з іменами стовпців:

```python
class Field:
    def __init__(self, name):
        self.name = name
        self.internal_name = '_' + self.name
    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')
    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```

Визначення класу для рядка бази даних вимагає надання імені стовпця для кожного атрибута класу:

```python
class Customer:
    # Атрибути класу
    first_name = Field('first_name')
    last_name  = Field('last_name')
    prefix     = Field('prefix')
    suffix     = Field('suffix')
```

Але визначення класу виглядає надлишковим. Ім'я поля вже оголошено ліворуч (`first_name =`). Чому також потрібно передавати рядок із тією самою інформацією в конструктор `Field` праворуч?

```python
class Customer:
    # Ліворуч надлишково відносно правого
    first_name = Field('first_name')
```

Щоб усунути цю надлишковість, можна використати метаклас для автоматичного присвоєння `Field.name` та `Field.internal_name` дескриптору:

```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        for key, value in class_dict.items():
            if isinstance(value, Field):
                value.name = key
                value.internal_name = '_' + key
        cls = type.__new__(meta, name, bases, class_dict)
        return cls

class DatabaseRow(metaclass=Meta):
    pass

class Field:
    def __init__(self):
        # Будуть присвоєні метакласом
        self.name = None
        self.internal_name = None
    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')
    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)

class BetterCustomer(DatabaseRow):
    first_name = Field()
    last_name  = Field()
    prefix     = Field()
    suffix     = Field()
```

Проблема цього підходу в тому, що не можна використовувати клас `Field` для властивостей, якщо не успадковуватися від `DatabaseRow`.

Рішенням є використання спеціального методу `__set_name__` для дескрипторів. Цей метод, введений у Python 3.6, викликається для кожного екземпляра дескриптора при визначенні класу-контейнера. Він отримує як параметри клас-власник, що містить екземпляр дескриптора, та ім'я атрибута, якому був присвоєний екземпляр дескриптора:

```python
class Field:
    def __init__(self):
        self.name = None
        self.internal_name = None
    def __set_name__(self, owner, name):
        # Викликається при створенні класу для кожного дескриптора
        self.name = name
        self.internal_name = '_' + name
    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')
    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```

Тепер можна отримати переваги дескриптора `Field` без необхідності успадковуватися від конкретного батьківського класу або використовувати метаклас:

```python
class FixedCustomer:
    first_name = Field()
    last_name  = Field()
    prefix     = Field()
    suffix     = Field()

cust = FixedCustomer()
print(f'Before: {cust.first_name!r} {cust.__dict__}')
cust.first_name = 'Mersenne'
print(f'After:  {cust.first_name!r} {cust.__dict__}')
>>>
Before: '' {}
After:  'Mersenne' {'_first_name': 'Mersenne'}
```

**Що слід запам'ятати**

- Метакласи дозволяють модифікувати атрибути класу до його повного визначення.
- Дескриптори та метакласи утворюють потужну комбінацію для декларативної поведінки та інтроспекції під час виконання.
- Визначайте `__set_name__` у класах дескрипторів, щоб вони могли враховувати клас-контейнер та його імена властивостей.
- Уникайте витоків пам'яті та вбудованого модуля `weakref`, зберігаючи дані, якими маніпулюють дескриптори, безпосередньо в словнику екземплярів класу.

---

## Порада 51. Надавайте перевагу декораторам класів над метакласами для компонованих розширень класів

Хоча метакласи дозволяють налаштовувати створення класу різними способами (см. Порада 48 та Порада 49), вони все ж не справляються з кожною ситуацією, що може виникнути.

Наприклад, потрібно прикрасити всі методи класу допоміжною функцією, що виводить аргументи, значення, що повертаються, та підняті винятки. Тут визначається налагоджувальний декоратор (см. Порада 26: «Визначайте декоратори функцій за допомогою functools.wraps»):

```python
from functools import wraps

def trace_func(func):
    if hasattr(func, 'tracing'):
        return func  # Декорувати лише один раз
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = None
        try:
            result = func(*args, **kwargs)
            return result
        except Exception as e:
            result = e
            raise
        finally:
            print(f'{func.__name__}({args!r}, {kwargs!r}) -> '
                  f'{result!r}')
    wrapper.tracing = True
    return wrapper
```

Можна застосувати цей декоратор до різних спеціальних методів нового підкласу `dict` (см. Порада 43):

```python
class TraceDict(dict):
    @trace_func
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
    @trace_func
    def __setitem__(self, *args, **kwargs):
        return super().__setitem__(*args, **kwargs)
    @trace_func
    def __getitem__(self, *args, **kwargs):
        return super().__getitem__(*args, **kwargs)
    ...
```

Проблема в тому, що довелося перевизначити всі методи, які потрібно було прикрасити. Це надлишковий шаблонний код, важкий для читання та схильний до помилок. Крім того, якщо пізніше буде додано новий метод до суперкласу `dict`, він не буде прикрашений.

Один зі способів вирішення — використати метаклас для автоматичного прикрашання всіх методів класу:

```python
import types

trace_types = (
    types.MethodType,
    types.FunctionType,
    types.BuiltinFunctionType,
    types.BuiltinMethodType,
    types.MethodDescriptorType,
    types.ClassMethodDescriptorType)

class TraceMeta(type):
    def __new__(meta, name, bases, class_dict):
        klass = super().__new__(meta, name, bases, class_dict)
        for key in dir(klass):
            value = getattr(klass, key)
            if isinstance(value, trace_types):
                wrapped = trace_func(value)
                setattr(klass, key, wrapped)
        return klass
```

Але цей підхід виходить з ладу, якщо суперклас вже має вказаний метаклас:

```python
class TraceDict(SimpleDict, metaclass=TraceMeta):
    pass
>>>
Traceback ...
TypeError: metaclass conflict: ...
```

Щоб вирішити цю проблему, Python підтримує декоратори класів. Декоратори класів працюють так само, як декоратори функцій: вони застосовуються за допомогою символу `@` перед оголошенням класу. Функція повинна модифікувати або відтворити клас і потім повернути його:

```python
def my_class_decorator(klass):
    klass.extra_param = 'hello'
    return klass

@my_class_decorator
class MyClass:
    pass

print(MyClass.extra_param)
>>>
hello
```

Можна реалізувати декоратор класу для застосування `trace_func` до всіх методів та функцій класу, перемістивши ядро методу `TraceMeta.__new__` у standalone-функцію:

```python
def trace(klass):
    for key in dir(klass):
        value = getattr(klass, key)
        if isinstance(value, trace_types):
            wrapped = trace_func(value)
            setattr(klass, key, wrapped)
    return klass
```

Цей декоратор можна застосувати до підкласу `dict` для отримання тієї самої поведінки:

```python
@trace
class TraceDict(dict):
    pass

trace_dict = TraceDict([('hi', 1)])
trace_dict['there'] = 2
trace_dict['hi']
try:
    trace_dict['does not exist']
except KeyError:
    pass  # Очікується
>>>
__new__((<class '__main__.TraceDict'>, [('hi', 1)]), {}) -> {}
__getitem__(({'hi': 1, 'there': 2}, 'hi'), {}) -> 1
__getitem__(({'hi': 1, 'there': 2}, 'does not exist'),
{}) -> KeyError('does not exist')
```

Декоратори класів також працюють, коли клас, що декорується, вже має метаклас:

```python
class OtherMeta(type):
    pass

@trace
class TraceDict(dict, metaclass=OtherMeta):
    pass
```

При пошуку компонованих способів розширення класів декоратори класів є найкращим інструментом для цього завдання.

**Що слід запам'ятати**

- Декоратор класу — це проста функція, що отримує екземпляр класу як параметр та повертає або новий клас, або модифіковану версію оригінального.
- Декоратори класів корисні, коли потрібно модифікувати кожен метод або атрибут класу з мінімальним шаблонним кодом.
- Метакласи не можна легко компонувати разом, тоді як кілька декораторів класів можна використовувати для розширення одного класу без конфліктів.
