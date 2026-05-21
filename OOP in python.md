
https://chat.deepseek.com/share/49jxy6l9xbfuab9alj


**1. Введение в объектно-ориентированное программирование в Python**

Объектно-ориентированное программирование (ООП) в Python — это не просто набор синтаксических конструкций, а парадигма, при которой данные и поведение объединяются в сущности — **объекты**. В отличие от процедурного подхода (данные отдельно, функции отдельно), ООП диктует: состояние (атрибуты) и методы (функции) живут внутри одного контейнера — **класса**.

**Ключевое различие: класс vs объект**
- **Класс** — чертёж, описание структуры. Он не существует в памяти как нечто работающее, пока вы его не используете.
- **Объект (экземпляр)** — конкретная реализация чертежа, выделенная в памяти. Один класс может породить множество независимых объектов.

**Минимальная реализация на Python:**
```python
class Car:
    pass          # пустой класс — редкость, но синтаксис допустим
```

**Почему ООП в Python, а не просто словари с функциями?**
- Словарь хранит состояние, но не может гарантировать, что вы случайно не удалите ключ.
- Функция, работающая со словарём, не привязана жёстко к структуре данных — легко ошибиться.
- Класс связывает данные и методы на уровне интерпретатора: `car.start()` явно говорит, что `start` относится именно к `car`, а не к любому другому объекту.

**Базовые принципы, которые мы дальше разберём детально:**
1. **Инкапсуляция** — сокрытие внутреннего состояния от прямого доступа.
2. **Наследование** — создание нового класса на основе существующего.
3. **Полиморфизм** — единый интерфейс для разных типов данных.

Первый шаг в ООП — научиться видеть в задаче не последовательность действий («сначала посчитать, потом записать»), а набор взаимодействующих объектов («машина получает команду ехать»). Python реализует эту парадигму нестрого (всё открыто по умолчанию), но при грамотном подходе вы получаете читаемый, модульный и расширяемый код.



**2. Классы и объекты: определение и создание экземпляров**

**Определение класса**

Класс создаётся инструкцией `class`, за которой следует имя и тело с атрибутами. По соглашению PEP 8 — имена классов в `CapWords` (CamelCase).

```python
class Account:
    """Банковский счёт — пример класса"""
    currency = "RUB"          # атрибут класса (общий для всех экземпляров)

    def __init__(self, owner: str, balance: float):
        self.owner = owner    # атрибут экземпляра
        self.balance = balance
```

**Метод `__init__` — конструктор (инициализатор)**
- Вызывается автоматически после создания объекта.
- Не возвращает новое значение (возвращает `None`).
- `self` — ссылка на создаваемый экземпляр. Первый параметр всех методов экземпляра.

**Создание экземпляра (instantiation)**

```python
acc1 = Account("Иван", 1000.0)
acc2 = Account("Мария", 500.0)
```

Синтаксис: имя_класса + круглые скобки с аргументами для `__init__`.

**Что происходит под капотом:**
1. `Account.__new__()` создаёт новый пустой объект в памяти.
2. Вызывается `Account.__init__(obj, "Иван", 1000.0)`, где `self` — только что созданный объект.
3. Объект возвращается и присваивается переменной `acc1`.

**Различие атрибутов класса и экземпляра — критично**

```python
print(acc1.currency)          # "RUB" — берётся из класса
print(acc2.currency)          # "RUB" — тот же самый

Account.currency = "USD"      # меняем атрибут класса
print(acc1.currency)          # "USD" — отразилось на всех

acc1.currency = "EUR"         # создаём атрибут экземпляра
print(acc1.currency)          # "EUR" — теперь свой
print(acc2.currency)          # "USD" — остался от класса
```

**Правило доступа:** Python сначала ищет атрибут в экземпляре (`__dict__` экземпляра), затем — в классе и его родителях.

**Типичная ошибка — мутация атрибута класса через экземпляр:**
```python
class User:
    tags = []                 # атрибут класса — список

u1 = User()
u2 = User()
u1.tags.append("admin")       # ошибка! изменили атрибут класса
print(u2.tags)                # ["admin"] — неожиданно для новичка
```
Правильно: инициализировать изменяемые типы в `__init__`.

**Проверка принадлежности классу:**
```python
isinstance(acc1, Account)     # True
type(acc1) is Account         # True (но не учитывает наследование)
```

**Кратко о `self`:** не магия, а явный параметр. Вы можете назвать его иначе (стандарт — `self`), но первый аргумент метода всегда получает сам объект. Когда пишете `acc1.deposit(100)`, Python вызывает `Account.deposit(acc1, 100)`.

**Практическое правило:** атрибуты класса — для констант и счётчиков, общих для всех экземпляров. Всё, что может отличаться у разных объектов, инициализируйте через `self` в `__init__`.



**3. Атрибуты и методы экземпляра**

**Атрибуты экземпляра**

Атрибуты экземпляра — данные, уникальные для каждого объекта. Они создаются внутри методов с префиксом `self.` (чаще всего — внутри `__init__`).

```python
class Book:
    def __init__(self, title: str, pages: int):
        self.title = title      # атрибут экземпляра
        self.pages = pages      # атрибут экземпляра
        self.current_page = 0   # со значением по умолчанию
```

**Динамическое добавление атрибутов**

Python позволяет создавать атрибуты экземпляра вне класса — прямо в коде:

```python
book = Book("Python Cookbook", 800)
book.author = "David Beazley"   # динамически добавленный атрибут
```

Это гибкость, но и источник ошибок. Типичная проблема:

```python
book1 = Book("Python Cookbook", 800)
book2 = Book("Fluent Python", 700)

book1.author = "David Beazley"
print(book2.author)  # AttributeError — у book2 нет author
```

**Контроль через `__slots__`** (редко, но нужно знать):
```python
class Point:
    __slots__ = ('x', 'y')   # запрещает создавать другие атрибуты
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
p.z = 3  # AttributeError
```

**Методы экземпляра**

Метод экземпляра — функция, определённая внутри класса, первым параметром принимающая `self`. Вызывается либо через экземпляр, либо через класс с явной передачей экземпляра.

```python
class Counter:
    def __init__(self):
        self.count = 0
    
    def increment(self, step: int = 1):
        self.count += step
    
    def get_value(self) -> int:
        return self.count

c = Counter()
c.increment(5)           # эквивалентно Counter.increment(c, 5)
print(c.get_value())     # 5
```

**Три правила методов экземпляра:**

1. **Первый параметр всегда `self`** — ссылка на конкретный объект. Имя `self` — соглашение, но не синтаксическое требование (можно `this`, но не делайте так).

2. **Доступ к атрибутам только через `self`** — внутри метода `self.count`, а не просто `count` (иначе это локальная переменная).

3. **Метод может вызывать другой метод того же экземпляра** — через `self.other_method()`.

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def scale(self, factor):
        self.width *= factor
        self.height *= factor
        return self.area()      # вызов другого метода
```

**Пространство имён: поиск атрибутов при вызове**

Когда пишете `obj.method()`, Python:
1. Ищет `method` в `obj.__dict__` (атрибуты экземпляра).
2. Не находит — идёт в `obj.__class__.__dict__`.
3. Находит функцию — возвращает объект **bound method** (привязанный метод), где `self` уже подставлен.

```python
r = Rectangle(10, 5)
print(r.area)           # <bound method Rectangle.area of <Rectangle object...>>
print(Rectangle.area)   # <function Rectangle.area at 0x...> — непривязанная функция
```

**Частая ошибка: забыли `self` в определении метода**

```python
class Wrong:
    def method():       # нет self
        print("hello")

w = Wrong()
w.method()              # TypeError: method() takes 0 positional arguments but 1 was given
```
Python автоматически передаёт экземпляр первым аргументом, но метод его не принимает.

**Метод vs функция внутри класса**

```python
class Example:
    def instance_method(self):
        return "instance"
    
    @staticmethod
    def static_method():
        return "static"
    
    @classmethod
    def class_method(cls):
        return "class"
```

Разбор:
- **Метод экземпляра** — получает `self`. Работает с состоянием объекта.
- **Статический метод** — не получает ни `self`, ни `cls`. Как обычная функция, но сгруппирована в класс для удобства.
- **Метод класса** — получает `cls` (класс). Используется для фабрик и работы с атрибутами класса.

**Итоговое правило:** если метод читает или изменяет атрибуты `self` — это метод экземпляра. Если нет — подумайте, не нужен ли `@staticmethod` или `@classmethod`.



**4. Конструктор `__init__` и инициализация экземпляров**

**`__init__` — не конструктор, а инициализатор**

В Python настоящий конструктор — `__new__`. Он создаёт пустой объект и возвращает его. `__init__` лишь заполняет уже существующий объект атрибутами. Различие важно для неизменяемых типов и паттерна "одиночка", но в 99% случаев вы работаете с `__init__`.

```python
class Point:
    def __new__(cls, x, y):
        print("Создаю объект")
        instance = super().__new__(cls)  # выделение памяти
        return instance                   # если вернуть None — __init__ не вызовется
    
    def __init__(self, x, y):
        print("Заполняю объект")
        self.x = x
        self.y = y
```

**Сигнатура `__init__`**

Первый параметр — `self` (уже существующий объект), далее любые аргументы, которые вы передаёте при создании экземпляра.

```python
class User:
    def __init__(self, name, age=18, email=None):
        self.name = name
        self.age = age
        self.email = email or f"{name.lower()}@example.com"

u = User("Alice", 25)                    # name=Alice, age=25, email=alice@example.com
```

**`__init__` всегда возвращает `None`**

Попытка вернуть что-либо, кроме `None`, вызывает `TypeError`:

```python
class Broken:
    def __init__(self):
        return 42          # TypeError: __init__() should return None

b = Broken()
```

**Порядок инициализации при наследовании**

Родительский класс должен быть инициализирован до дочернего, если дочернему нужны его атрибуты. Вызов `super().__init__(...)` обязателен, если у родителя есть свои параметры.

```python
class Vehicle:
    def __init__(self, brand):
        self.brand = brand

class Car(Vehicle):
    def __init__(self, brand, model):
        super().__init__(brand)      # сначала родитель
        self.model = model           # потом своё

c = Car("Toyota", "Camry")
```

**Распространённая ошибка:** забыли вызвать `super().__init__()` — атрибуты родителя не появятся.

**Перегрузка `__init__` в Python отсутствует**

В отличие от Java или C++, в Python нельзя написать несколько `__init__` с разными сигнатурами. Решения:

1. **Значения по умолчанию:**
```python
class Rectangle:
    def __init__(self, width=0, height=0):
        self.width = width
        self.height = height
```

2. **Аргументы-звёздочки:**
```python
class DataPoint:
    def __init__(self, *values, label=None):
        self.values = values
        self.label = label

d = DataPoint(1, 2, 3, label="A")
```

3. **`@classmethod` фабрики (паттерн):**
```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
    
    @classmethod
    def from_string(cls, date_str):
        year, month, day = map(int, date_str.split('-'))
        return cls(year, month, day)

d = Date.from_string("2024-03-15")
```

4. **`@dataclass` (Python 3.7+) — автоматический `__init__`:**
```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int = 0
    email: str = None

p = Person("Bob", 30)   # __init__ сгенерирован автоматически
```

**Антипаттерны `__init__`**

❌ **Сложная логика и вызовы внешних API:**
```python
class Bad:
    def __init__(self, user_id):
        self.data = fetch_from_db(user_id)  # не делайте так
        self.validate()                      # и так тоже
```
Инициализатор должен быть быстрым и предсказуемым. Тяжёлую логику — в методы.

❌ **Изменение класса (`cls.__name__` и прочее):**
```python
class Weird:
    def __init__(self):
        type(self).counter = 5   # допустимо, но сбивает с толку
```

**Типичный паттерн: защита от мутации аргументов**

```python
class Team:
    def __init__(self, members):
        self.members = list(members)   # копируем, а не сохраняем ссылку

original = ["Alice", "Bob"]
team = Team(original)
original.append("Eve")                 # не влияет на team.members
```

**Итоговое правило:** `__init__` отвечает только за присвоение начальных атрибутов. Всё остальное — проверки, преобразования, логи — минимизируйте или выносите в отдельные методы. Помните: `__init__` не обязан принимать все будущие атрибуты — динамическое добавление возможно, но плохо читается.



**5. Атрибуты класса и методы класса (`@classmethod`)**

**Атрибуты класса**

Атрибуты класса принадлежат самому классу, а не его экземплярам. Они существуют в единственном экземпляре и разделяются всеми объектами этого класса.

```python
class Product:
    tax_rate = 0.20          # атрибут класса
    total_products = 0       # счётчик всех созданных продуктов
    
    def __init__(self, name, price):
        self.name = name
        self.price = price
        Product.total_products += 1
```

**Доступ к атрибутам класса**

```python
# Через класс (предпочтительно)
print(Product.tax_rate)        # 0.2

# Через экземпляр (возможно, но с нюансами)
p = Product("Laptop", 1000)
print(p.tax_rate)              # 0.2 — поиск идёт в классе

# Присваивание через экземпляр создаёт атрибут экземпляра
p.tax_rate = 0.25              # создали свой, не тронули класс
print(p.tax_rate)              # 0.25
print(Product.tax_rate)        # 0.2 — не изменился
```

**Когда использовать атрибуты класса:**
- Константы, общие для всех экземпляров
- Счётчики и аккумуляторы (количество созданных объектов)
- Значения по умолчанию для атрибутов экземпляров
- Кэши и реестры на уровне класса

**Методы класса (`@classmethod`)**

Декоратор `@classmethod` превращает метод в метод класса. Первый параметр — `cls` (ссылка на класс, а не на экземпляр).

```python
class Person:
    population = 0
    
    def __init__(self, name):
        self.name = name
        Person.population += 1
    
    @classmethod
    def get_population(cls):
        return cls.population
    
    @classmethod
    def create_anonymous(cls):
        return cls("Anonymous")

# Вызов
print(Person.get_population())           # через класс
p = Person.create_anonymous()            # фабричный метод
```

**Ключевые отличия от методов экземпляра:**

| Характеристика | Метод экземпляра | Метод класса |
|----------------|------------------|---------------|
| Первый параметр | `self` (экземпляр) | `cls` (класс) |
| Доступ к атрибутам экземпляра | Да | Нет |
| Доступ к атрибутам класса | Да (через `self.__class__`) | Да |
| Может быть вызван через экземпляр | Да | Да |
| Может быть вызван через класс | Да (с явной передачей экземпляра) | Да |

**Типичные применения `@classmethod`**

**1. Альтернативные конструкторы (фабричные методы):**

```python
from datetime import date

class Person:
    def __init__(self, name, birth_year):
        self.name = name
        self.birth_year = birth_year
    
    @classmethod
    def from_birth_date(cls, name, birth_date):
        birth_year = birth_date.year
        return cls(name, birth_year)
    
    @classmethod
    def from_age(cls, name, age):
        current_year = date.today().year
        birth_year = current_year - age
        return cls(name, birth_year)

p1 = Person("Alice", 1990)
p2 = Person.from_birth_date("Bob", date(1995, 5, 15))
p3 = Person.from_age("Charlie", 30)
```

**2. Наследование и полиморфизм классов:**

```python
class Animal:
    sound = "?"
    
    @classmethod
    def make_sound(cls):
        print(cls.sound)
    
    @classmethod
    def create_from_config(cls, config):
        return cls()

class Dog(Animal):
    sound = "Woof"

class Cat(Animal):
    sound = "Meow"

Dog.make_sound()          # Woof
Cat.make_sound()          # Meow
# При наследовании cls указывает на конкретный дочерний класс
```

**3. Управление реестрами классов:**

```python
class Plugin:
    _registry = {}
    
    def __init__(self, name):
        self.name = name
    
    @classmethod
    def register(cls, name, plugin_class):
        cls._registry[name] = plugin_class
    
    @classmethod
    def get_plugin(cls, name):
        return cls._registry.get(name)

class TextPlugin(Plugin):
    pass

Plugin.register("text", TextPlugin)
```

**Частая ошибка: забыли `@classmethod` и передали `cls` как обычный параметр**

```python
class Wrong:
    def factory(cls, arg):     # нет декоратора — обычный метод экземпляра
        return cls(arg)

# TypeError: factory() takes 2 positional arguments but 3 were given
# Потому что Python подставил self, а вы передали ещё и cls
```

**`@classmethod` vs `@staticmethod`**

```python
class Demo:
    @classmethod
    def class_method(cls):
        return f"Class: {cls.__name__}"
    
    @staticmethod
    def static_method():
        return "No class or instance info"
```

| | `@classmethod` | `@staticmethod` |
|--|----------------|------------------|
| Получает класс | Да (`cls`) | Нет |
| Может вызывать другие classmethod'ы | Да | Только через явное указание класса |
| Поддерживает наследование (полиморфизм) | Да | Нет |
| Может быть переопределён в дочернем классе | Да | Да (но не получит `cls`) |

**Правило выбора:**
- Нужен доступ к атрибутам/методам класса или нужен полиморфизм при наследовании → `@classmethod`
- Нужна просто функция, логически сгруппированная в классе → `@staticmethod`
- Нужен доступ к `self` → метод экземпляра

**Итог:** `@classmethod` — мощный инструмент для работы на уровне класса. Главное преимущество — полиморфизм при наследовании, чего лишены статические методы. Используйте их для альтернативных конструкторов и операций над всеми экземплярами класса.



**6. Статические методы (`@staticmethod`)**

**Определение и синтаксис**

Статический метод — метод, не получающий при вызове ни `self` (экземпляр), ни `cls` (класс). Это обычная функция, но логически принадлежащая классу и вызванная через его пространство имён.

```python
class MathUtils:
    @staticmethod
    def is_prime(n: int) -> bool:
        if n < 2:
            return False
        for i in range(2, int(n ** 0.5) + 1):
            if n % i == 0:
                return False
        return True
    
    @staticmethod
    def factorial(n: int) -> int:
        result = 1
        for i in range(2, n + 1):
            result *= i
        return result

# Вызов через класс
print(MathUtils.is_prime(17))    # True

# Вызов через экземпляр (работает, но не имеет смысла)
mu = MathUtils()
print(mu.factorial(5))           # 120
```

**Почему статический метод, а не функция вне класса?**

| Критерий | Функция вне класса | `@staticmethod` внутри класса |
|----------|-------------------|-------------------------------|
| Логическая группировка | Нет, в глобальном пространстве | Да, в контексте класса |
| Наследование и переопределение | Нет | Да (дочерний класс может переопределить) |
| Импорт | Нужен отдельный импорт функции | Импортируется класс, методы доступны через точку |
| Доступ к атрибутам класса | Нет | Нет (если только не через явное указание класса) |

**Типичные сценарии использования**

**1. Вспомогательные функции, тесно связанные с классом**

```python
class FileValidator:
    @staticmethod
    def is_image(filename: str) -> bool:
        return filename.lower().endswith(('.png', '.jpg', '.jpeg', '.gif'))
    
    @staticmethod
    def is_archive(filename: str) -> bool:
        return filename.lower().endswith(('.zip', '.tar', '.gz'))
    
    @staticmethod
    def sanitize_filename(filename: str) -> str:
        return ''.join(c for c in filename if c.isalnum() or c in '._-')

# Использование
if FileValidator.is_image("photo.jpg"):
    name = FileValidator.sanitize_filename("user:photo.jpg")
```

**2. Проверки и валидация данных перед созданием объекта**

```python
class User:
    def __init__(self, username, email):
        self.username = username
        self.email = email
    
    @staticmethod
    def is_valid_username(username: str) -> bool:
        return 3 <= len(username) <= 20 and username.isalnum()
    
    @staticmethod
    def is_valid_email(email: str) -> bool:
        return '@' in email and '.' in email.split('@')[-1]
    
    @classmethod
    def create_safe(cls, username, email):
        if not cls.is_valid_username(username):
            raise ValueError("Invalid username")
        if not cls.is_valid_email(email):
            raise ValueError("Invalid email")
        return cls(username, email)
```

**3. Форматирование и преобразование данных**

```python
class Currency:
    def __init__(self, amount: float, code: str):
        self.amount = amount
        self.code = code
    
    @staticmethod
    def format_amount(amount: float, decimals: int = 2) -> str:
        return f"{amount:.{decimals}f}"
    
    @staticmethod
    def from_cents(cents: int, code: str = "USD"):
        return Currency(cents / 100, code)
    
    def __str__(self):
        return f"{self.format_amount(self.amount)} {self.code}"

c = Currency.from_cents(12345, "EUR")
print(c)  # 123.45 EUR
```

**Статический метод vs метод класса: ключевое различие**

```python
class Parent:
    @staticmethod
    def static():
        return "Parent static"
    
    @classmethod
    def class_method(cls):
        return f"Parent class: {cls.__name__}"

class Child(Parent):
    @staticmethod
    def static():
        return "Child static"
    
    @classmethod
    def class_method(cls):
        return f"Child class: {cls.__name__}"

# Вызов через дочерний класс
print(Child.static())           # "Child static" — переопределили
print(Child.class_method())     # "Child class: Child" — получил Child как cls
```

Но статический метод **не знает**, через какой класс вызван, и не может получить доступ к атрибутам класса:

```python
class Config:
    DEFAULT_LEVEL = 1
    
    @staticmethod
    def get_level():
        # Нет доступа к Config.DEFAULT_LEVEL, если не указать явно
        return Config.DEFAULT_LEVEL
    
    @classmethod
    def get_level_polymorphic(cls):
        return cls.DEFAULT_LEVEL

class CustomConfig(Config):
    DEFAULT_LEVEL = 2

print(CustomConfig.get_level())              # 1 — взял из Config, а не из CustomConfig
print(CustomConfig.get_level_polymorphic())  # 2 — полиморфное поведение
```

**Когда `@staticmethod` — антипаттерн**

❌ **Статический метод, активно использующий другие статические методы того же класса** — возможно, лучше сделать функцией вне класса.

❌ **Статический метод, который постоянно обращается к атрибутам через явное имя класса** — используйте `@classmethod`.

```python
# Плохо
class Bad:
    TAX = 0.2
    
    @staticmethod
    def calculate_with_tax(price):
        return price * (1 + Bad.TAX)   # жёсткая привязка к Bad
    
# Хорошо
class Good:
    TAX = 0.2
    
    @classmethod
    def calculate_with_tax(cls, price):
        return price * (1 + cls.TAX)   # полиморфизм при наследовании
```

**Практическое правило выбора:**

| Нужно | Решение |
|-------|---------|
| Функция не использует `self` и не использует атрибуты класса | Внешняя функция или `@staticmethod` |
| Функция логически принадлежит классу и не требует доступа к экземпляру/классу | `@staticmethod` |
| Функция требует доступа к атрибутам класса или полиморфизма при наследовании | `@classmethod` |
| Функция требует доступа к экземпляру (`self`) | Метод экземпляра |

**Итог:** статические методы — это обычные функции, помещённые в класс для организации кода. Они не видят ни класс, ни экземпляр. Используйте их для утилит, проверок, преобразований, тесно связанных с классом, но не нуждающихся в его состоянии. Для всего остального, что работает на уровне класса, предпочтительнее `@classmethod`.



**7. Инкапсуляция: публичные, защищённые и приватные члены**

**Концепция инкапсуляции в Python**

Инкапсуляция — механизм, ограничивающий прямой доступ к внутренним данным объекта. В отличие от Java или C++, Python не имеет строгих модификаторов доступа. Вместо этого используются соглашения об именовании и механизм "name mangling" (искажение имён).

**Три уровня доступа**

| Уровень | Соглашение | Пример | Доступ |
|---------|------------|--------|--------|
| Публичный (public) | Обычное имя | `self.name` | Открыт везде |
| Защищённый (protected) | Одно подчёркивание | `self._name` | "Не трогайте извне" |
| Приватный (private) | Два подчёркивания | `self.__name` | Искажение имени |

**Публичные члены (public)**

По умолчанию все атрибуты и методы в Python публичны. Это осознанное решение языка — "мы все взрослые люди".

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner      # публичный атрибут
        self.balance = balance  # публичный атрибут
    
    def deposit(self, amount):  # публичный метод
        self.balance += amount

acc = BankAccount("Ivan", 1000)
acc.balance = 999999            # напрямую изменяем — ничего не阻止
acc.deposit(500)
```

**Защищённые члены (protected): одно подчёркивание `_`**

Соглашение: "это внутренняя деталь реализации, не используйте снаружи". Интерпретатор не запрещает доступ — это договорённость между разработчиками.

```python
class DatabaseConnection:
    def __init__(self, host):
        self.host = host
        self._connection = None      # защищённый атрибут
        self._is_connected = False   # защищённый атрибут
    
    def connect(self):
        self._open_connection()
    
    def _open_connection(self):      # защищённый метод
        if not self._is_connected:
            self._connection = socket.socket()
            self._is_connected = True

db = DatabaseConnection("localhost")
db.connect()
db._open_connection()                # Синтаксически разрешено, но нарушает контракт
print(db._is_connected)              # Можем, но не должны
```

**Правило:** обращаться к защищённым членам можно только внутри класса, в его методах и в классах-наследниках. Снаружи — не принято.

**Приватные члены (private): два подчёркивания `__`**

Механизм **name mangling**: Python переименовывает атрибут в `_ClassName__attribute`. Это не настоящая приватность, а способ избежать случайных конфликтов имён при наследовании.

```python
class User:
    def __init__(self, name):
        self.name = name
        self.__password = "secret"   # приватный атрибут
    
    def __hash_password(self):       # приватный метод
        return f"hashed_{self.__password}"

u = User("Alice")
print(u.__password)                   # AttributeError: 'User' object has no attribute '__password'
print(u._User__password)              # "secret" — но так делать не нужно
```

**Как работает name mangling:**

```python
class Parent:
    def __init__(self):
        self.__value = 42        # становится _Parent__value
    
    def __method(self):          # становится _Parent__method
        pass

class Child(Parent):
    def __init__(self):
        super().__init__()
        self.__value = 100       # становится _Child__value — другой атрибут!

c = Child()
print(c._Parent__value)          # 42
print(c._Child__value)           # 100
```

**Зачем нужен name mangling?**

1. **Предотвращение конфликтов имён в глубокой иерархии наследования:**
```python
class Logger:
    def log(self, message):
        self.__write(message)      # вызывает свой __write
    
    def __write(self, msg):        # _Logger__write
        print(f"LOG: {msg}")

class FileLogger(Logger):
    def __write(self, msg):        # _FileLogger__write — другой метод!
        with open("log.txt", "a") as f:
            f.write(msg)

# Logger.log всегда вызывает _Logger__write, не _FileLogger__write
```

2. **Сигнал "это абсолютно внутренняя деталь, не переопределяй".**

**Правильный способ доступа к приватным членам (геттеры и сеттеры):**

```python
class Temperature:
    def __init__(self, celsius):
        self.__celsius = celsius
    
    def get_celsius(self):
        return self.__celsius
    
    def set_celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero")
        self.__celsius = value
    
    def get_fahrenheit(self):
        return self.__celsius * 9/5 + 32

t = Temperature(25)
print(t.get_celsius())        # 25
t.set_celsius(30)
```

**Современный подход: свойства (property)**

Свойства позволяют контролировать доступ без изменения интерфейса:

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius   # защищённый, не приватный
    
    @property
    def celsius(self):
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero")
        self._celsius = value
    
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32

t = Temperature(25)
print(t.celsius)           # 25 — выглядит как атрибут
t.celsius = 30             # через сеттер
print(t.fahrenheit)        # 86.0 — вычисляемое свойство
```

**Сравнение подходов:**

| Подход | Контроль доступа | Изменение интерфейса | Сложность |
|--------|------------------|----------------------|-----------|
| Прямой атрибут | Нет | Нет | Минимальная |
| Геттер/сеттер | Да | Да (было свойство, стал метод) | Средняя |
| `@property` | Да | Нет | Средняя |

**Практические рекомендации:**

1. **Начинайте с публичных атрибутов.** Не усложняйте без необходимости.

2. **Используйте `_` для внутренних деталей.** Сигнал другим разработчикам (и себе в будущем).

3. **Приватные `__` используйте редко.** Только когда точно нужен name mangling (мало кейсов).

4. **Свойства — для контроля.** Когда нужно добавить логику при доступе или сделать атрибут только для чтения.

```python
class ReadOnlyExample:
    def __init__(self, value):
        self._value = value
    
    @property
    def value(self):
        return self._value
        # нет сеттера — атрибут только для чтения
```

**Мифы о приватности в Python:**

- "`__` защищает от доступа извне" — нет, защита только на уровне переименования.
- "В Python нет инкапсуляции" — нет, инкапсуляция есть, но она основана на соглашениях, не на синтаксических барьерах.

**Итог:** Python поощряет дисциплину разработчика, а не запреты. Используйте `_` для защиты, `__` только для избежания конфликтов имён при наследовании, а для контроля доступа — `@property`. Главная инкапсуляция в Python — в голове программиста, а не в интерпретаторе.



**8. Свойства (`@property`) и контролируемый доступ к атрибутам**

**Проблема, которую решают свойства**

Прямой доступ к атрибутам прост и читаем, но не позволяет добавить валидацию или логику позже без изменения интерфейса. Свойства дают возможность заменить атрибут методом, сохранив синтаксис прямого обращения.

```python
# Плохо: прямой доступ без контроля
class User:
    def __init__(self, age):
        self.age = age  # можно установить age = -100

# Ещё хуже: геттеры и сеттеры ломают интерфейс
class User:
    def __init__(self, age):
        self._age = age
    def get_age(self):
        return self._age
    def set_age(self, value):
        if value < 0:
            raise ValueError
user.set_age(-5)  # Громоздко, не похоже на атрибут

# Хорошо: свойства
class User:
    def __init__(self, age):
        self._age = age
    
    @property
    def age(self):
        return self._age
    
    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("Age cannot be negative")
        self._age = value

user = User(25)
user.age = 30      # Синтаксис как у атрибута, но с валидацией
print(user.age)    # 30
```

**Синтаксис свойств**

Свойство создаётся декоратором `@property`. Метод становится геттером. Сеттер и делитер добавляются декораторами `@имя.setter` и `@имя.deleter`.

```python
class Rectangle:
    def __init__(self, width, height):
        self._width = width
        self._height = height
    
    # Геттер
    @property
    def width(self):
        """Ширина прямоугольника"""
        return self._width
    
    # Сеттер
    @width.setter
    def width(self, value):
        if value <= 0:
            raise ValueError("Width must be positive")
        self._width = value
    
    # Делитер
    @width.deleter
    def width(self):
        print("Deleting width")
        del self._width
    
    # Вычисляемое свойство (только геттер)
    @property
    def area(self):
        return self._width * self._height

r = Rectangle(10, 5)
print(r.area)      # 50 — вычисляется на лету
r.width = 15
del r.width        # вызывает deleter
```

**Вычисляемые свойства (read-only)**

Самый частый вариант — свойство без сеттера. Используется для деривации данных.

```python
from datetime import date

class Person:
    def __init__(self, birth_date: date):
        self._birth_date = birth_date
    
    @property
    def birth_date(self):
        return self._birth_date
    
    @property
    def age(self):
        today = date.today()
        return today.year - self._birth_date.year - (
            (today.month, today.day) < (self._birth_date.month, self._birth_date.day)
        )
    
    @property
    def is_adult(self):
        return self.age >= 18

p = Person(date(1995, 5, 15))
print(p.age)        # вычисляется при каждом обращении
print(p.is_adult)   # вычисляется на основе age
```

**Свойства с кэшированием (ленивая инициализация)**

```python
class DataProcessor:
    def __init__(self, data):
        self.data = data
        self._processed_cache = None
    
    @property
    def processed(self):
        if self._processed_cache is None:
            print("Processing data...")  # Тяжёлая операция
            self._processed_cache = [x * 2 for x in self.data]
        return self._processed_cache

dp = DataProcessor([1, 2, 3, 4, 5])
print(dp.processed)  # Processing data... [2, 4, 6, 8, 10]
print(dp.processed)  # [2, 4, 6, 8, 10] — из кэша
```

**Классический vs современный синтаксис свойств**

Старый способ (функция `property`):

```python
class OldWay:
    def __init__(self, value):
        self._value = value
    
    def get_value(self):
        return self._value
    
    def set_value(self, value):
        self._value = value
    
    value = property(get_value, set_value)

# Современный способ (декораторы) — предпочтительнее
class NewWay:
    def __init__(self, value):
        self._value = value
    
    @property
    def value(self):
        return self._value
    
    @value.setter
    def value(self, value):
        self._value = value
```

**Свойства и наследование**

Свойства наследуются и могут быть переопределены в дочерних классах.

```python
class Animal:
    def __init__(self, name):
        self._name = name
    
    @property
    def name(self):
        return self._name
    
    @property
    def sound(self):
        return "???"

class Dog(Animal):
    @property
    def sound(self):
        return "Woof"
    
    @property
    def name(self):
        return f"Dog: {self._name}"

d = Dog("Rex")
print(d.name)   # Dog: Rex — переопределили геттер
print(d.sound)  # Woof
```

**Тонкий момент: свойства не переопределяют атрибуты экземпляра**

Если в экземпляре есть атрибут с тем же именем, что и свойство класса, атрибут экземпляра «побеждает» при чтении, но сеттер работать не будет.

```python
class Demo:
    @property
    def x(self):
        return 42
    
    @x.setter
    def x(self, value):
        print(f"Setting x to {value}")

d = Demo()
print(d.x)        # 42 — вызывает геттер
d.x = 100         # "Setting x to 100" — вызывает сеттер

d.__dict__['x'] = 999   # прямое добавление атрибута
print(d.x)        # 999 — теперь атрибут экземпляра, свойство игнорируется
d.x = 200         # всё ещё "Setting x to 200" — сеттер работает, но меняет _x?
```

**Когда использовать свойства:**

✅ **Валидация при установке:**
```python
@temperature.setter
def temperature(self, value):
    if value < -273.15:
        raise ValueError
    self._temperature = value
```

✅ **Вычисляемые атрибуты:**
```python
@property
def full_name(self):
    return f"{self.first_name} {self.last_name}"
```

✅ **Ленивая загрузка ресурсов:**
```python
@property
def db_connection(self):
    if self._connection is None:
        self._connection = create_connection()
    return self._connection
```

✅ **Обратная совместимость — замена публичного атрибута на метод без изменения кода клиентов.**

❌ **Когда НЕ использовать свойства:**

- **Дорогие вычисления, вызываемые часто** — оставьте методом, чтобы было очевидно, что операция затратна.
- **Операции с побочными эффектами** — свойство не должно менять состояние системы.
- **Медленные операции (сеть, диск)** — свойство должно быть быстрым (O(1) или легковесным).

```python
# Плохо: свойство делает HTTP-запрос
class Bad:
    @property
    def user_data(self):
        return requests.get('/api/user').json()  # не делайте так

# Хорошо: обычный метод
class Good:
    def fetch_user_data(self):
        return requests.get('/api/user').json()
```

**Свойства vs атрибуты класса**

```python
class Example:
    class_attr = 100           # атрибут класса
    
    def __init__(self):
        self._instance_attr = 200
    
    @property
    def instance_attr(self):
        return self._instance_attr

# Свойство принадлежит экземпляру, но определено на уровне класса
print(Example.class_attr)      # 100
print(Example.instance_attr)   # <property object at ...> — не вызывается
```

**Итоговые правила:**

1. Начинайте с публичного атрибута. Когда понадобится контроль — превращайте его в свойство. API клиентов не ломается.

2. Свойство должно быть быстрым и не иметь побочных эффектов.

3. Для сложной логики используйте методы. Свойства — для простых операций чтения/записи с валидацией.

4. `@property.setter` и `@property.deleter` используйте только когда реально нужны. Read-only свойства — отличное решение.

5. Не злоупотребляйте свойствами. Прямой атрибут `self.value` чаще всего — лучшее решение.



**9. Наследование: одиночное и множественное наследование**

**Определение и базовый синтаксис**

Наследование — механизм, позволяющий создать новый класс на основе существующего. Дочерний класс (подкласс) получает все атрибуты и методы родителя (суперкласса) и может расширять или переопределять их.

```python
class Animal:                     # Родительский класс (базовый)
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return "Some sound"

class Dog(Animal):                # Дочерний класс — Animal в скобках
    def speak(self):              # Переопределение метода
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

d = Dog("Rex")
c = Cat("Whiskers")
print(d.name)        # Rex — унаследовал атрибут
print(d.speak())     # Woof!
print(c.speak())     # Meow!
```

**Одиночное наследование (single inheritance)**

Класс наследуется от одного родителя. Это наиболее распространённый и понятный случай.

```python
class Vehicle:
    def __init__(self, brand, year):
        self.brand = brand
        self.year = year
    
    def info(self):
        return f"{self.brand} ({self.year})"

class Car(Vehicle):
    def __init__(self, brand, year, doors):
        super().__init__(brand, year)    # Вызов родительского __init__
        self.doors = doors
    
    def info(self):                       # Расширение, а не замена
        return f"Car: {super().info()}, Doors: {self.doors}"

class Motorcycle(Vehicle):
    def __init__(self, brand, year, engine_cc):
        super().__init__(brand, year)
        self.engine_cc = engine_cc

c = Car("Toyota", 2020, 4)
print(c.info())      # Car: Toyota (2020), Doors: 4
```

**Функция `super()`**

`super()` возвращает прокси-объект, делегирующий вызовы методам родительского класса. Это критически важно при множественном наследовании и правильном построении MRO (Method Resolution Order).

```python
class Parent:
    def __init__(self):
        print("Parent init")

class Child(Parent):
    def __init__(self):
        super().__init__()    # Вызов родительского __init__
        print("Child init")

# Без super():
class BadChild(Parent):
    def __init__(self):
        Parent.__init__(self)   # Работает, но ломает множественное наследование
```

**Переопределение методов и атрибутов**

Дочерний класс может полностью переопределить метод родителя или расширить его.

```python
class Logger:
    def log(self, message):
        print(f"LOG: {message}")
    
    def format_message(self, message):
        return message.upper()

class FileLogger(Logger):
    def log(self, message):                     # Полное переопределение
        with open("log.txt", "a") as f:
            f.write(f"{message}\n")
    
    def format_message(self, message):          # Расширение
        formatted = super().format_message(message)
        return f"[{timestamp()}] {formatted}"
```

**Множественное наследование (multiple inheritance)**

Класс может наследоваться от нескольких родителей. Python поддерживает его полностью, но требует осторожности.

```python
class Flyer:
    def move(self):
        return "Flying"
    
    def action(self):
        return "Spreading wings"

class Swimmer:
    def move(self):
        return "Swimming"
    
    def action(self):
        return "Diving"

class Duck(Flyer, Swimmer):      # Порядок важен — влияет на MRO
    def perform(self):
        print(self.move())       # "Flying" — от первого родителя
        print(self.action())     # "Spreading wings"

d = Duck()
d.perform()
print(Duck.__mro__)  # (<class 'Duck'>, <class 'Flyer'>, <class 'Swimmer'>, <class 'object'>)
```

**Проблема ромба (Diamond problem) и MRO**

В классическом ромбовидном наследовании один класс наследуется от двух, которые наследуются от общего предка. Python решает это через C3-линеаризацию (MRO — Method Resolution Order).

```python
class A:
    def method(self):
        print("A")
    
    def common(self):
        print("A common")

class B(A):
    def method(self):
        print("B")
        super().method()

class C(A):
    def method(self):
        print("C")
        super().method()

class D(B, C):
    def method(self):
        print("D")
        super().method()

d = D()
d.method()          # D → B → C → A
print(D.__mro__)    # (D, B, C, A, object)

# common метод наследуется только один раз
d.common()          # "A common" — не дублируется
```

**Правила MRO (C3-линеаризация):**

1. Дочерние классы идут раньше родительских.
2. Порядок родителей сохраняется (как в скобках).
3. Если класс встречается несколько раз, остаётся только первое вхождение.

**Практические паттерны множественного наследования**

**1. Миксины (Mixins)** — небольшие классы, добавляющие конкретную функциональность.

```python
class JSONMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__)

class XMLMixin:
    def to_xml(self):
        return f"<object>{self.__dict__}</object>"

class User(JSONMixin, XMLMixin):
    def __init__(self, name, age):
        self.name = name
        self.age = age

u = User("Alice", 30)
print(u.to_json())   # {"name": "Alice", "age": 30}
print(u.to_xml())    # <object>{'name': 'Alice', 'age': 30}</object>
```

**2. Абстрактные базовые классы (ABC)** — определение интерфейсов.

```python
from abc import ABC, abstractmethod

class Drawable(ABC):
    @abstractmethod
    def draw(self):
        pass

class Resizable(ABC):
    @abstractmethod
    def resize(self, factor):
        pass

class Rectangle(Drawable, Resizable):
    def draw(self):
        print("Drawing rectangle")
    
    def resize(self, factor):
        print(f"Resizing by {factor}")
```

**Опасности множественного наследования**

❌ **Конфликт имён методов с одинаковой сигнатурой:**

```python
class One:
    def process(self):
        return 1

class Two:
    def process(self):
        return 2

class Three(One, Two):
    pass

t = Three()
print(t.process())   # 1 — от первого родителя, но это не всегда очевидно
```

❌ **Сложный MRO с запутанной логикой:**

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass
class E(C, B): pass   # TypeError: Cannot create a consistent method resolution — impossible
```

❌ **Инициализация с разными сигнатурами родителей:**

```python
class Parent1:
    def __init__(self, a):
        self.a = a

class Parent2:
    def __init__(self, b):
        self.b = b

class Child(Parent1, Parent2):
    def __init__(self, a, b):
        Parent1.__init__(self, a)   # Явный вызов, super() не поможет
        Parent2.__init__(self, b)
```

**Проверка принадлежности и типовая иерархия**

```python
class Animal: pass
class Mammal(Animal): pass
class Dog(Mammal): pass

d = Dog()
print(isinstance(d, Dog))       # True
print(isinstance(d, Mammal))    # True
print(isinstance(d, Animal))    # True
print(isinstance(d, object))    # True

print(issubclass(Dog, Animal))  # True
print(issubclass(Dog, object))  # True
```

**Когда использовать наследование:**

✅ **Отношение «is-a»** — «собака — это животное»: `class Dog(Animal)`

✅ **Повторное использование кода** — общая логика в родителе

✅ **Полиморфизм** — единый интерфейс для разных типов

❌ **Не используйте наследование для «has-a»** (используйте композицию)

```python
# Плохо: наследование для переиспользования метода
class Engine:
    def start(self): pass

class Car(Engine):      # Машина НЕ является двигателем
    pass

# Хорошо: композиция
class Car:
    def __init__(self):
        self.engine = Engine()   # Машина имеет двигатель
```

**Итоговые рекомендации:**

1. **Одиночное наследование** — предпочтительно. Просто, понятно, безопасно.

2. **Миксины** — допустимое использование множественного наследования. Называйте их с суффиксом `Mixin`.

3. **Глубина наследования** — не более 2–3 уровней. Глубокая иерархия — признак плохого дизайна.

4. **Всегда вызывайте `super().__init__()`** в конструкторе дочернего класса, если инициализируете родителя.

5. **Проверяйте MRO** через `ClassName.__mro__`, если сомневаетесь в порядке вызовов.

6. **Избегайте множественного наследования** от классов с состоянием (атрибутами). Миксины должны быть stateless (только методы).



**10. Порядок разрешения методов (MRO)**

**Определение MRO**

MRO (Method Resolution Order) — последовательность, в которой Python ищет методы и атрибуты в иерархии наследования. При вызове `obj.method()` интерпретатор обходит классы в строго определённом порядке, возвращая первый найденный метод.

```python
class A:
    def who(self):
        print("A")

class B(A):
    def who(self):
        print("B")

class C(A):
    def who(self):
        print("C")

class D(B, C):
    pass

d = D()
d.who()              # "B" — первым в MRO идёт B
print(D.__mro__)     # (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

**Алгоритм C3-линеаризации**

Python использует C3-линеаризацию (известную как C3 superclass linearization), которая гарантирует:

1. **Монотонность** — если класс X следует перед Y в MRO одного класса, то X всегда будет перед Y во всех подклассах.
2. **Локальный порядок предков** — классы в `(Parent1, Parent2)` сохраняют порядок.
3. **Наследование** — подкласс идёт перед суперклассом.

**Ручной расчёт MRO (упрощённо):**

```python
class O: pass
class A(O): pass
class B(O): pass
class C(O): pass
class D(A, B): pass
class E(C, B): pass
class F(D, E): pass

# MRO для F:
# 1. Начинаем с F
# 2. Добавляем MRO(D) = [D, A, B, O]
# 3. Добавляем MRO(E) = [E, C, B, O]
# 4. Склеиваем с сохранением порядка и удалением дубликатов
# Результат: F, D, A, E, C, B, O
```

**Просмотр MRO**

```python
# Способ 1: атрибут __mro__
print(D.__mro__)

# Способ 2: метод mro()
print(D.mro())

# Способ 3: help()
help(D)
```

**Как MRO работает с `super()`**

`super()` не вызывает родительский класс напрямую. Он использует MRO текущего класса и вызывает **следующий класс в цепочке MRO**.

```python
class A:
    def work(self):
        print("A")
        super().work()    # вызывает object.work — ничего не делает

class B(A):
    def work(self):
        print("B")
        super().work()

class C(A):
    def work(self):
        print("C")
        super().work()

class D(B, C):
    def work(self):
        print("D")
        super().work()

D().work()
# D → B → C → A
print(D.__mro__)  # (D, B, C, A, object)
```

**Визуализация цепочки вызовов `super()`:**

```
Вызов D().work()
    ↓
D.work(): print("D") → super() ищет следующий в MRO после D → B
    ↓
B.work(): print("B") → super() ищет следующий в MRO после B → C
    ↓
C.work(): print("C") → super() ищет следующий в MRO после C → A
    ↓
A.work(): print("A") → super() ищет следующий в MRO после A → object
```

**MRO при ромбовидном наследовании (классический пример)**

```python
class Grandparent:
    def method(self):
        print("Grandparent")
        super().method()

class Parent1(Grandparent):
    def method(self):
        print("Parent1")
        super().method()

class Parent2(Grandparent):
    def method(self):
        print("Parent2")
        super().method()

class Child(Parent1, Parent2):
    def method(self):
        print("Child")
        super().method()

Child().method()
print(Child.__mro__)
# (<class 'Child'>, <class 'Parent1'>, <class 'Parent2'>, <class 'Grandparent'>, <class 'object'>)
# Вывод: Child → Parent1 → Parent2 → Grandparent
```

**Почему Grandparent вызывается один раз, а не дважды?**

Именно благодаря MRO и алгоритму C3. `Grandparent` входит в MRO только один раз, в позиции после `Parent2`. Цепочка `super()` идёт линейно, не возвращаясь к уже посещённым классам.

**Типичные ошибки и их причины**

**1. Несовместимый MRO (TypeError)**

```python
class X: pass
class Y: pass
class Z(X, Y): pass
class W(Y, X): pass
class V(Z, W): pass
# TypeError: Cannot create a consistent method resolution order (MRO)
```

Причина: Python не может выстроить линейный порядок, удовлетворяющий всем требованиям (Z требует X до Y, W требует Y до X).

**2. `super()` без аргументов в Python 3**

В Python 3 `super()` автоматически определяет текущий класс и экземпляр:

```python
class Parent:
    def __init__(self):
        print("Parent")

class Child(Parent):
    def __init__(self):
        super().__init__()      # эквивалентно super(Child, self).__init__()
```

**3. Пропуск `super().__init__()` в одном из родителей при множественном наследовании**

```python
class A:
    def __init__(self):
        self.a = 1

class B:
    def __init__(self):
        self.b = 2

class C(A, B):
    def __init__(self):
        super().__init__()      # вызывает только A.__init__, B пропущен!

c = C()
print(hasattr(c, 'b'))          # False — B не инициализирован

# Решение: согласованный вызов super() во всех классах
class A:
    def __init__(self):
        super().__init__()
        self.a = 1

class B:
    def __init__(self):
        super().__init__()
        self.b = 2

class C(A, B):
    def __init__(self):
        super().__init__()      # вызывает A, который вызывает B (через object)
```

**Практические паттерны с MRO**

**1. Кооперативное множественное наследование**

Все классы в иерархии должны вызывать `super()` в своих методах, даже если у них нет непосредственного родителя.

```python
class Base:
    def save(self):
        print("Base save")

class LoggerMixin:
    def save(self):
        print("Logging")
        super().save()

class ValidatorMixin:
    def save(self):
        print("Validating")
        super().save()

class Model(LoggerMixin, ValidatorMixin, Base):
    def save(self):
        print("Model save prep")
        super().save()

m = Model()
m.save()
# Model save prep → Logging → Validating → Base save
print(Model.__mro__)  # Model, LoggerMixin, ValidatorMixin, Base, object
```

**2. Изменение MRO (не рекомендуется, но возможно)**

```python
class A:
    def method(self):
        print("A")

class B(A):
    def method(self):
        print("B")
        super().method()

class C(A):
    def method(self):
        print("C")
        super().method()

class D(B, C):
    def method(self):
        print("D")
        super().method()

# Явный вызов конкретного родителя (обходит MRO)
class E(D):
    def method(self):
        C.method(self)    # прямой вызов, минуя MRO
        super().method()

e = E()
e.method()  # C → D → B → C? (может вызвать C дважды)
```

**MRO и `isinstance` / `issubclass`**

Эти функции используют MRO для определения отношений:

```python
class A: pass
class B(A): pass
class C(B): pass

print(issubclass(C, A))           # True — A есть в MRO(C)
print(issubclass(C, (A, B, int))) # True — хотя бы один есть в MRO
```

**Сравнение MRO в Python 2.2+ (C3) и старых версиях (depth-first)**

В Python 2.1 и ранее использовался depth-first (глубинный обход) с обходом дубликатов. Это приводило к проблемам:

```python
# Старый алгоритм (глубинный) мог вызвать A дважды
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass
# Старый MRO: D, B, A, C, A → проблема

# Новый C3: D, B, C, A → корректно
```

**Итоговые правила работы с MRO**

1. **Просматривайте MRO явно** — `ClassName.__mro__` при любом сомнении.

2. **Всегда используйте `super()`** в иерархиях с множественным наследованием. Прямые вызовы `Parent.method(self)` ломают кооперативность.

3. **Миксины не должны иметь `__init__`** или должны вызывать `super().__init__()`.

4. **Порядок родителей в определении класса важен** — он влияет на MRO.

5. **Избегайте глубины наследования более 3-4 уровней** — MRO становится трудно отслеживать.

6. **Тестируйте MRO** при сложных иерархиях:

```python
def test_mro(cls, expected_chain):
    actual = [c.__name__ for c in cls.__mro__ if c is not object]
    assert actual == expected_chain, f"MRO broken: {actual}"
```

**Ключевое понимание:** MRO — не магия, а предсказуемый линейный порядок, построенный по чётким правилам. `super()` следует этому порядку, гарантируя, что каждый класс в иерархии будет вызван ровно один раз.



**11. Функция `super()` и доступ к родительским классам**

**Что такое `super()` на самом деле**

`super()` возвращает объект-прокси, который делегирует вызовы методов следующему классу в цепочке MRO (Method Resolution Order), а не непосредственно родительскому классу.

```python
class Parent:
    def method(self):
        return "Parent"

class Child(Parent):
    def method(self):
        return f"Child + {super().method()}"

c = Child()
print(c.method())          # "Child + Parent"
```

**Ключевое заблуждение:** `super()` ≠ родительский класс. `super()` вызывает **следующий класс в MRO**, который может быть не прямым родителем при множественном наследовании.

**Полная форма `super()`**

```python
# Полная форма
super(Child, self).method()

# Краткая форма (Python 3+)
super().method()

# Они эквивалентны в контексте метода
class Child(Parent):
    def method(self):
        super().method()              # super(Child, self).method()
```

**Параметры `super(type, obj)`**

| Форма | Что делает |
|-------|------------|
| `super()` | Автоматически определяет текущий класс и `self` (только внутри метода) |
| `super(Child, self)` | Ищет следующий класс в MRO после `Child`, начиная с `self` |
| `super(Child, Child)` | Ищет следующий класс в MRO после `Child`, возвращает непривязанный метод (нужен явный `self`) |
| `super(Child, cls)` | Для классметодов — следующий класс после `Child` в MRO класса `cls` |

**`super()` в методах класса (`@classmethod`)**

```python
class Base:
    @classmethod
    def factory(cls):
        return cls()

class Derived(Base):
    @classmethod
    def factory(cls):
        instance = super().factory()    # super(Derived, cls).factory()
        print(f"Created {instance}")
        return instance

d = Derived.factory()
```

**`super()` без экземпляра (непривязанный вызов)**

```python
class A:
    def hello(self):
        return "A"

class B(A):
    def hello(self):
        return "B"

# Непривязанный super
unbound_super = super(B, B)
print(unbound_super.hello)    # <function A.hello at 0x...> — не привязан к self

# Требует явной передачи экземпляра
b = B()
print(unbound_super.hello(b)) # "A"
```

**Цепочка вызовов `super()` при множественном наследовании**

```python
class A:
    def process(self):
        print("A")
        return 1

class B(A):
    def process(self):
        print("B")
        result = super().process()    # вызывает следующий после B в MRO
        return result + 1

class C(A):
    def process(self):
        print("C")
        result = super().process()    # вызывает следующий после C в MRO
        return result + 10

class D(B, C):
    def process(self):
        print("D")
        result = super().process()    # вызывает следующий после D в MRO
        return result + 100

print(D.__mro__)  # (D, B, C, A, object)
print(D().process())  # D → B → C → A → 1+10+1+100 = 112
```

**Визуализация потока:**

```
D().process()
  ↓ печатает "D"
  super().process() → ищет после D → B
    ↓ печатает "B"
    super().process() → ищет после B → C
      ↓ печатает "C"
      super().process() → ищет после C → A
        ↓ печатает "A"
        возвращает 1
      возвращает 1 + 10 = 11
    возвращает 11 + 1 = 12
  возвращает 12 + 100 = 112
```

**Типичные сценарии использования**

**1. Расширение, а не замена родительского метода**

```python
class Logger:
    def log(self, message, level="INFO"):
        print(f"[{level}] {message}")

class TimestampLogger(Logger):
    def log(self, message, level="INFO"):
        from datetime import datetime
        timestamp = datetime.now().isoformat()
        super().log(f"{timestamp} - {message}", level)

t = TimestampLogger()
t.log("System started")  # [INFO] 2024-03-15T10:30:00 - System started
```

**2. Инициализация в иерархии наследования**

```python
class Person:
    def __init__(self, name):
        self.name = name

class Employee(Person):
    def __init__(self, name, employee_id):
        super().__init__(name)
        self.employee_id = employee_id

class Manager(Employee):
    def __init__(self, name, employee_id, team_size):
        super().__init__(name, employee_id)
        self.team_size = team_size

m = Manager("Alice", "M001", 5)
print(m.name, m.employee_id, m.team_size)  # Alice M001 5
```

**3. Делегирование работы родителям (паттерн Template Method)**

```python
class DataProcessor:
    def process(self, data):
        data = self.validate(data)
        data = self.transform(data)
        data = self.save(data)
        return data
    
    def validate(self, data):
        return data
    
    def transform(self, data):
        return data
    
    def save(self, data):
        return data

class JSONProcessor(DataProcessor):
    def validate(self, data):
        if not isinstance(data, dict):
            raise ValueError("Expected dict")
        return super().validate(data)
    
    def transform(self, data):
        import json
        return json.dumps(data)

processor = JSONProcessor()
result = processor.process({"key": "value"})  # сохраняет JSON-строку
```

**Ошибки и антипаттерны с `super()`**

**1. Прямой вызов родительского класса вместо `super()` (ломает множественное наследование)**

```python
# Плохо
class BadB(A):
    def process(self):
        A.process(self)    # жёсткая привязка к A

# Хорошо
class GoodB(A):
    def process(self):
        super().process()  # работает с любым следующим классом
```

**2. Пропуск `super().__init__()` в одном из классов иерархии**

```python
class Base:
    def __init__(self):
        self.base_attr = True

class Mixin:
    def __init__(self):
        self.mixin_attr = True

class Combined(Base, Mixin):
    def __init__(self):
        super().__init__()   # вызывает только Base.__init__
        # Mixin.__init__ не вызывается!

c = Combined()
print(hasattr(c, 'mixin_attr'))  # False

# Исправление: все классы должны вызывать super().__init__()
class FixedMixin:
    def __init__(self):
        super().__init__()
        self.mixin_attr = True
```

**3. Разные сигнатуры `__init__` в родителях**

```python
class A:
    def __init__(self, a):
        self.a = a

class B:
    def __init__(self, b):
        self.b = b

class C(A, B):
    def __init__(self, a, b):
        super().__init__(a)   # вызывает A.__init__, B пропущен
        # B.__init__ не получит b

# Решение: явное согласование
class C(A, B):
    def __init__(self, a, b):
        A.__init__(self, a)   # прямой вызов неизбежен
        B.__init__(self, b)
```

**`super()` и дескрипторы (property, classmethod, staticmethod)**

`super()` правильно обрабатывает дескрипторы:

```python
class A:
    @property
    def value(self):
        return "A"

class B(A):
    @property
    def value(self):
        return super().value + " → B"

b = B()
print(b.value)  # "A → B"
```

**`super()` в функциях вне класса (продвинутое использование)**

```python
def delegated_method(instance):
    """Вызывает метод родителя для любого экземпляра"""
    cls = type(instance)
    super(cls, instance).method()

class Parent:
    def method(self):
        return "parent"

class Child(Parent):
    def method(self):
        return "child"

c = Child()
print(delegated_method(c))  # "parent" (вызвал Parent.method)
```

**Практические правила использования `super()`**

| Сценарий | Использование |
|----------|---------------|
| Внутри метода экземпляра | `super().method()` |
| Внутри `@classmethod` | `super().method()` |
| Внутри `@staticmethod` | `super(CurrentClass, cls).method()` (редко) |
| Вне класса (продвинутый) | `super(Child, obj).method()` |
| Для вызова метода родителя, игнорируя MRO | `Parent.method(self)` (только если уверены) |

**Итоговая суть `super()`:**

`super()` — не магия, а предсказуемый механизм линейного обхода MRO. Он гарантирует, что каждый класс в иерархии будет вызван ровно один раз при кооперативном множественном наследовании. Всегда используйте `super()` вместо прямых вызовов родителей, если только вы не пишете код, заведомо не предназначенный для множественного наследования. Даже в одиночном наследовании `super()` делает код более гибким к будущим изменениям.



**12. Переопределение и расширение поведения родительских методов**

**Определение переопределения (method overriding)**

Переопределение — создание в дочернем классе метода с тем же именем, что и в родительском. При вызове через экземпляр дочернего класса исполняется его версия, а не родительская.

```python
class Animal:
    def speak(self):
        return "Some generic sound"

class Dog(Animal):
    def speak(self):           # Переопределение
        return "Woof!"

class Cat(Animal):
    def speak(self):           # Переопределение
        return "Meow!"

animals = [Dog(), Cat(), Animal()]
for a in animals:
    print(a.speak())           # Woof! Meow! Some generic sound
```

**Полное переопределение (замена поведения)**

Дочерний класс полностью заменяет реализацию родителя, игнорируя её.

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

class Square(Rectangle):
    def __init__(self, side):
        self.side = side
    
    def area(self):                     # Полное переопределение
        return self.side ** 2

s = Square(5)
print(s.area())                         # 25
```

**Расширение поведения через `super()`**

Дочерний класс вызывает родительскую реализацию и дополняет её.

```python
class Logger:
    def log(self, message):
        print(f"LOG: {message}")

class TimestampLogger(Logger):
    def log(self, message):
        from datetime import datetime
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        super().log(f"[{timestamp}] {message}")   # Расширение

class FileLogger(TimestampLogger):
    def log(self, message):
        super().log(message)                       # Вызов родителя
        with open("log.txt", "a") as f:
            f.write(f"{message}\n")                # Дополнительное поведение

fl = FileLogger()
fl.log("System started")
# LOG: [2024-03-15 10:30:00] System started
# И запись в файл
```

**Паттерны расширения методов**

**1. Pre-processing (до родительского вызова)**

```python
class Validator:
    def validate(self, data):
        return data

class EmailValidator(Validator):
    def validate(self, data):
        if '@' not in data:
            raise ValueError("Invalid email")
        return super().validate(data)      # Родитель после проверки
```

**2. Post-processing (после родительского вызова)**

```python
class DataProcessor:
    def process(self, value):
        return value * 2

class RoundingProcessor(DataProcessor):
    def process(self, value):
        result = super().process(value)     # Родитель первым
        return round(result, 2)             # Пост-обработка
```

**3. Полная замена с условным вызовом родителя**

```python
class PaymentGateway:
    def process_payment(self, amount, currency):
        return {"status": "success", "amount": amount}

class SecureGateway(PaymentGateway):
    def process_payment(self, amount, currency):
        if amount > 10000:
            self.require_approval(amount)
        if self.is_suspicious(amount, currency):
            return {"status": "blocked", "reason": "suspicious"}
        return super().process_payment(amount, currency)
    
    def require_approval(self, amount):
        print(f"Approval needed for ${amount}")
    
    def is_suspicious(self, amount, currency):
        return amount > 50000 and currency == "USD"
```

**Переопределение `__init__`**

Наиболее частый случай переопределения — конструктор.

```python
class Vehicle:
    def __init__(self, brand, year):
        self.brand = brand
        self.year = year

class Car(Vehicle):
    def __init__(self, brand, year, doors):
        super().__init__(brand, year)      # Обязательный вызов
        self.doors = doors

class Truck(Vehicle):
    def __init__(self, brand, year, capacity):
        super().__init__(brand, year)
        self.capacity = capacity

t = Truck("Volvo", 2020, 2000)
print(t.brand, t.year, t.capacity)        # Volvo 2020 2000
```

**Переопределение специальных методов (`__str__`, `__repr__` и др.)**

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __str__(self):
        return f"{self.name}, {self.age} years old"

class Employee(Person):
    def __init__(self, name, age, employee_id):
        super().__init__(name, age)
        self.employee_id = employee_id
    
    def __str__(self):
        return f"{super().__str__()} (ID: {self.employee_id})"

e = Employee("Alice", 30, "E123")
print(str(e))        # Alice, 30 years old (ID: E123)
print(repr(e))       # <__main__.Employee object at 0x...> — не переопределён
```

**Переопределение и сигнатуры методов**

Python не проверяет соответствие сигнатур при переопределении. Это может привести к ошибкам.

```python
class Parent:
    def method(self, x, y):
        return x + y

class Child(Parent):
    def method(self, x):                # Изменена сигнатура
        return x * 2

c = Child()
print(c.method(5))      # 10 — работает
print(c.method(5, 10))  # TypeError: method() takes 2 positional arguments but 3 were given
```

**Правило:** при расширении через `super()` сигнатура должна совпадать. При полной замене — может отличаться, но это усложняет полиморфное использование.

**Переопределение classmethod и staticmethod**

```python
class Base:
    @classmethod
    def factory(cls):
        return cls()
    
    @staticmethod
    def helper():
        return "base"

class Derived(Base):
    @classmethod
    def factory(cls):
        instance = super().factory()
        print(f"Created {instance}")
        return instance
    
    @staticmethod
    def helper():
        return "derived"

d = Derived.factory()    # Created <__main__.Derived object>
print(Derived.helper())  # derived
```

**Запрет на переопределение (финализация методов)**

Python не имеет встроенного механизма `final`. Но можно использовать соглашения или метаклассы.

```python
# Способ 1: соглашение с документированием
class Base:
    def critical_method(self):
        """DO NOT OVERRIDE — core logic"""
        return "critical"

# Способ 2: метакласс
class FinalMeta(type):
    def __new__(cls, name, bases, namespace):
        for base in bases:
            for attr_name, attr_value in base.__dict__.items():
                if attr_name in namespace and callable(attr_value):
                    raise TypeError(f"Cannot override {attr_name}")
        return super().__new__(cls, name, bases, namespace)

class BaseFinal(metaclass=FinalMeta):
    def forbidden(self):
        pass

# class Bad(BaseFinal):
#     def forbidden(self):  # TypeError!
#         pass
```

**Абстрактные методы — обязательное переопределение**

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass                    # Должен быть переопределён

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):              # Обязательное переопределение
        return 3.14 * self.radius ** 2

# s = Shape()                   # TypeError: Can't instantiate abstract class
c = Circle(5)
print(c.area())                 # 78.5
```

**Полиморфизм через переопределение**

```python
class FileHandler:
    def open(self):
        raise NotImplementedError
    
    def read(self):
        raise NotImplementedError
    
    def close(self):
        raise NotImplementedError

class TextFileHandler(FileHandler):
    def open(self):
        print("Opening text file")
    
    def read(self):
        return "Text content"
    
    def close(self):
        print("Closing text file")

class BinaryFileHandler(FileHandler):
    def open(self):
        print("Opening binary file")
    
    def read(self):
        return b"Binary content"
    
    def close(self):
        print("Closing binary file")

def process_file(handler: FileHandler):
    handler.open()
    data = handler.read()
    print(f"Data: {data}")
    handler.close()

process_file(TextFileHandler())
process_file(BinaryFileHandler())
```

**Антипаттерны переопределения**

❌ **Вызов `super()` в неподходящем месте**

```python
class Bad:
    def save(self):
        self.before_save()
        super().save()        # Если у object нет save — AttributeError
        self.after_save()
```

❌ **Забытый вызов `super().__init__()`**

```python
class Parent:
    def __init__(self):
        self.initialized = True

class Child(Parent):
    def __init__(self):
        pass                    # super().__init__() пропущен

c = Child()
print(c.initialized)            # AttributeError
```

❌ **Переопределение без необходимости**

```python
class Child(Parent):
    def method(self):
        return super().method()   # Бессмысленное переопределение — не нужно
```

❌ **Переопределение приватных методов (`__method`)**

```python
class Parent:
    def __private(self):
        return "parent"

class Child(Parent):
    def __private(self):      # Это _Child__private — новый метод
        return "child"

c = Child()
print(c._Parent__private())   # "parent" — родительский не переопределён
```

**Правила переопределения:**

| Сценарий | Подход |
|----------|--------|
| Нужна новая реализация | Полное переопределение без `super()` |
| Нужно добавить логику до/после | `super()` в начале или конце |
| Нужно полностью изменить поведение, но сохранить интерфейс | Переопределение с новой логикой |
| Нужно запретить переопределение | Соглашение или метакласс |
| Нужно обязать переопределить | `@abstractmethod` |

**Итог:** переопределение — основа полиморфизма в Python. Расширение через `super()` сохраняет цепочку вызовов и поддерживает кооперативное наследование. Полная замена даёт свободу, но ломает связь с родителем. Выбирайте паттерн в зависимости от того, является ли дочерний класс **специализацией** (расширение) или **новой сущностью** (замена).



**13. Полиморфизм и утиная типизация в Python**

**Определение полиморфизма**

Полиморфизм — способность объектов разных классов отвечать на один и тот же интерфейс (метод) своим собственным поведением. В Python полиморфизм встроен в язык на уровне синтаксиса и не требует явного наследования.

```python
class Dog:
    def sound(self):
        return "Woof!"

class Cat:
    def sound(self):
        return "Meow!"

class Car:
    def sound(self):
        return "Beep!"

def make_sound(animal):
    print(animal.sound())

make_sound(Dog())   # Woof!
make_sound(Cat())   # Meow!
make_sound(Car())   # Beep! — полиморфизм без наследования
```

**Утиная типизация (Duck Typing)**

Главный принцип: "Если это выглядит как утка, плавает как утка и крякает как утка — это утка". Python не проверяет тип объекта, а проверяет наличие необходимых методов и атрибутов.

```python
class Duck:
    def quack(self):
        print("Quack!")
    def fly(self):
        print("Flying")

class Person:
    def quack(self):
        print("I'm imitating a duck!")
    def fly(self):
        print("I'm flying in a plane")

def act_like_duck(obj):
    obj.quack()
    obj.fly()

act_like_duck(Duck())    # Quack! Flying
act_like_duck(Person())  # I'm imitating a duck! I'm flying in a plane
```

**Полиморфизм через наследование (классический)**

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        return 3.14159 * self.radius ** 2

shapes = [Rectangle(10, 5), Circle(7)]
for shape in shapes:
    print(shape.area())   # 50, 153.938 — полиморфный вызов
```

**Полиморфизм без наследования (утиная типизация в действии)**

```python
class JSONSerializer:
    def serialize(self, data):
        import json
        return json.dumps(data)

class XMLSerializer:
    def serialize(self, data):
        from dict2xml import dict2xml
        return dict2xml(data)

class YAMLSerializer:
    def serialize(self, data):
        import yaml
        return yaml.dump(data)

def save_data(data, serializer):
    serialized = serializer.serialize(data)
    with open("data.txt", "w") as f:
        f.write(serialized)

save_data({"name": "Alice"}, JSONSerializer())
save_data({"name": "Bob"}, XMLSerializer())    # Не наследуют общий класс
save_data({"name": "Charlie"}, YAMLSerializer())
```

**Проверка утиной типизации с `hasattr()`**

```python
def process_duck(obj):
    if hasattr(obj, 'quack') and callable(obj.quack):
        obj.quack()
    if hasattr(obj, 'walk') and callable(obj.walk):
        obj.walk()
    else:
        print("This is not a duck")

class RobotDuck:
    def quack(self):
        print("Beep-quack")
    def walk(self):
        print("Clank-clank")

process_duck(RobotDuck())   # Beep-quack Clank-clank
```

**EAFP (Easier to Ask for Forgiveness than Permission) vs LBYL (Look Before You Leap)**

Утиная типизация поощряет подход EAFP — пробовать, а потом обрабатывать ошибку.

```python
# LBYL (не Pythonic)
def safe_quack_lbyl(obj):
    if hasattr(obj, 'quack') and callable(obj.quack):
        obj.quack()
    else:
        print("Can't quack")

# EAFP (Pythonic)
def safe_quack_eafp(obj):
    try:
        obj.quack()
    except AttributeError:
        print("Can't quack")
```

**Полиморфизм со встроенными типами**

Встроенные функции Python полиморфны по своей природе.

```python
# len() полиморфна
print(len([1, 2, 3]))     # 3 — список
print(len("hello"))       # 5 — строка
print(len({"a": 1}))      # 1 — словарь

# Оператор + полиморфен
print(1 + 2)              # 3 — числа
print("a" + "b")          # "ab" — строки
print([1] + [2])          # [1, 2] — списки

# Цикл for полиморфен
def iterate(collection):
    for item in collection:   # Любой итерируемый объект
        print(item)

iterate([1, 2, 3])
iterate("abc")
iterate({1, 2, 3})
```

**Создание полиморфных интерфейсов с протоколами (Python 3.8+)**

`Protocol` из модуля `typing` позволяет определить структурный тип (утиную типизацию с подсказками).

```python
from typing import Protocol

class Quackable(Protocol):
    def quack(self) -> str:
        ...

class Duck:
    def quack(self) -> str:
        return "Quack!"

class ToyDuck:
    def quack(self) -> str:
        return "Squeak!"

class Dog:
    def bark(self) -> str:
        return "Woof!"

def make_quack(obj: Quackable) -> None:
    print(obj.quack())

make_quack(Duck())       # Quack!
make_quack(ToyDuck())    # Squeak!
# make_quack(Dog())      # Ошибка типа у mypy, но выполнится с AttributeError
```

**Полиморфные функции и операторы**

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):          # Полиморфизм через перегрузку операторов
        return Vector(self.x + other.x, self.y + other.y)
    
    def __str__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)   # Vector(4, 6)
```

**Гибкость vs безопасность: аргументы за и против утиной типизации**

| Преимущества | Недостатки |
|--------------|------------|
| Минимум кода | Ошибки времени выполнения |
| Высокая гибкость | Сложно отследить, что требует функция |
| Лёгкое тестирование (моки) | IDE хуже подсказывает |
| Не требует наследования | Риск случайных совпадений имён |

**Стратегии защиты от ошибок утиной типизации**

```python
# 1. Проверка протокола через hasattr
def process(obj):
    required = ['read', 'write', 'close']
    if all(hasattr(obj, attr) for attr in required):
        obj.read()
    else:
        raise TypeError("Object doesn't implement required protocol")

# 2. Абстрактные базовые классы (регистрация)
from collections.abc import Iterable

def process_iterable(obj: Iterable):
    for item in obj:      # mypy проверит тип
        print(item)

process_iterable([1, 2, 3])     # OK
process_iterable("abc")         # OK
# process_iterable(42)          # Ошибка типа

# 3. Собственные ABC с регистрацией
from abc import ABC, abstractmethod

class Drawable(ABC):
    @abstractmethod
    def draw(self):
        pass

Drawable.register(list)   # Список теперь считается Drawable

def draw_it(obj: Drawable):
    print("Drawing:", obj)
```

**Полиморфизм в стандартной библиотеке**

```python
from io import StringIO, BytesIO

# Разные объекты с одинаковым интерфейсом
def read_data(source):
    return source.read()

string_io = StringIO("Hello, world!")
bytes_io = BytesIO(b"Binary data")

print(read_data(string_io))   # Hello, world!
print(read_data(bytes_io))    # b'Binary data'
```

**Практический пример: полиморфная обработка платежей**

```python
class CreditCardPayment:
    def pay(self, amount):
        print(f"Processing credit card payment of ${amount}")
        return {"status": "success", "method": "credit"}

class PayPalPayment:
    def pay(self, amount):
        print(f"Redirecting to PayPal for ${amount}")
        return {"status": "success", "method": "paypal"}

class CryptoPayment:
    def pay(self, amount):
        print(f"Checking blockchain for ${amount}")
        return {"status": "pending", "method": "crypto"}

def checkout(cart_total, payment_method):
    result = payment_method.pay(cart_total)
    print(f"Payment result: {result}")

checkout(100, CreditCardPayment())
checkout(50, PayPalPayment())
checkout(200, CryptoPayment())
```

**Итоговые правила полиморфизма в Python:**

1. **Полиморфизм не требует наследования** — утиная типизация работает по наличию методов.

2. **EAFP над LBYL** — пробуйте вызвать метод, перехватывайте `AttributeError` вместо проверок `hasattr`.

3. **Используйте протоколы для документации** — `Protocol` из `typing` делает утиную типизацию явной.

4. **Абстрактные базовые классы** — когда нужен строгий контракт и проверка на этапе разработки.

5. **Специальные методы (`__len__`, `__getitem__`, `__iter__`)** — основа полиморфизма для встроенных функций.

**Ключевая мысль:** в Python полиморфизм — не особенность отдельных классов, а фундаментальное свойство языка. Любая функция, которая не проверяет типы явно, уже полиморфна. Утиная типизация даёт максимальную гибкость, но требует дисциплины и хорошего тестирования.



**14. Абстрактные базовые классы (ABC) и `@abstractmethod`**

**Определение ABC**

Абстрактный базовый класс — класс, который не может быть инстанциирован (нельзя создать его экземпляр). Он служит шаблоном (контрактом), определяющим методы, которые должны реализовать дочерние классы.

```python
from abc import ABC, abstractmethod

class Shape(ABC):                    # Наследуем от ABC
    @abstractmethod
    def area(self):
        """Должен быть реализован в подклассе"""
        pass
    
    @abstractmethod
    def perimeter(self):
        pass

# shape = Shape()                    # TypeError: Can't instantiate abstract class Shape

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):                  # Обязательная реализация
        return self.width * self.height
    
    def perimeter(self):             # Обязательная реализация
        return 2 * (self.width + self.height)

r = Rectangle(10, 5)                 # OK
```

**Синтаксис и правила**

**1. Импорт ABC и abstractmethod:**
```python
from abc import ABC, abstractmethod

class MyAbstract(ABC):
    @abstractmethod
    def required_method(self):
        pass
```

**2. Абстрактный метод может иметь реализацию:**
```python
class Logger(ABC):
    @abstractmethod
    def log(self, message):
        """Базовая реализация, которую можно вызвать через super()"""
        print(f"LOG: {message}")

class FileLogger(Logger):
    def log(self, message):
        super().log(message)                # Вызов абстрактной реализации
        with open("log.txt", "a") as f:
            f.write(message + "\n")

fl = FileLogger()
fl.log("Hello")          # LOG: Hello и запись в файл
```

**3. Абстрактные свойства:**
```python
class Config(ABC):
    @property
    @abstractmethod
    def timeout(self):
        pass
    
    @timeout.setter
    @abstractmethod
    def timeout(self, value):
        pass

class AppConfig(Config):
    def __init__(self):
        self._timeout = 30
    
    @property
    def timeout(self):
        return self._timeout
    
    @timeout.setter
    def timeout(self, value):
        if value <= 0:
            raise ValueError
        self._timeout = value
```

**4. Абстрактные классметоды и статические методы:**
```python
class Factory(ABC):
    @classmethod
    @abstractmethod
    def create(cls):
        pass
    
    @staticmethod
    @abstractmethod
    def validate(data):
        pass

class UserFactory(Factory):
    @classmethod
    def create(cls):
        return cls()
    
    @staticmethod
    def validate(data):
        return isinstance(data, dict)
```

**Абстрактные классы с частичной реализацией**

ABC может содержать как абстрактные, так и конкретные методы.

```python
class DataProcessor(ABC):
    def process(self, data):                # Конкретный метод
        data = self.validate(data)
        data = self.transform(data)
        self.save(data)
        return data
    
    @abstractmethod
    def validate(self, data):
        pass
    
    @abstractmethod
    def transform(self, data):
        pass
    
    def save(self, data):                   # Конкретный метод с реализацией по умолчанию
        with open("output.txt", "w") as f:
            f.write(str(data))

class JSONProcessor(DataProcessor):
    def validate(self, data):
        if not isinstance(data, dict):
            raise ValueError
        return data
    
    def transform(self, data):
        import json
        return json.dumps(data)

p = JSONProcessor()
p.process({"key": "value"})                 # validate → transform → save
```

**Регистрация виртуальных подклассов (`register`)**

Класс можно зарегистрировать как подкласс ABC без явного наследования.

```python
class MyIterable(ABC):
    @abstractmethod
    def __iter__(self):
        pass

# Регистрация встроенного типа
MyIterable.register(list)

print(issubclass(list, MyIterable))    # True
print(isinstance([1, 2, 3], MyIterable))  # True

# Регистрация своего класса
class MyCollection:
    def __iter__(self):
        return iter([1, 2, 3])

MyIterable.register(MyCollection)

c = MyCollection()
print(isinstance(c, MyIterable))       # True — хотя не наследует
```

**Проверка абстрактности**

```python
from abc import ABCMeta

class Base(ABC):
    @abstractmethod
    def method(self):
        pass

class Concrete(Base):
    def method(self):
        return "done"

print(Base.__abstractmethods__)        # {'method'}
print(Concrete.__abstractmethods__)    # set()

# Проверка, является ли класс абстрактным
print(ABCMeta.__subclasshook__)        # Можно переопределить
```

**Абстрактные классы в стандартной библиотеке**

Многие модули используют ABC для определения протоколов.

```python
from collections.abc import Iterable, Sequence, MutableMapping

# Проверка на итерируемость
print(isinstance([1, 2, 3], Iterable))      # True
print(isinstance("abc", Iterable))          # True
print(isinstance(123, Iterable))            # False

# Создание своего итерируемого
class MyRange:
    def __init__(self, n):
        self.n = n
    
    def __iter__(self):
        for i in range(self.n):
            yield i

print(isinstance(MyRange(5), Iterable))     # True (есть __iter__)

from io import IOBase
print(issubclass(open('test.txt', 'w'), IOBase))  # True
```

**Абстрактные классы vs протоколы (Protocol)**

| Характеристика | ABC | Protocol (typing) |
|----------------|-----|-------------------|
| Проверка во время выполнения | Да (`isinstance`) | Нет (только статическая) |
| Проверка статическим типизатором | Да (mypy) | Да (структурная) |
| Явное наследование | Обычно да | Нет (структурный) |
| Может содержать реализацию | Да | Нет (только сигнатуры) |
| Регистрация виртуальных подклассов | Да | Не нужно |

```python
# ABC — проверка в рантайме
class DrawableABC(ABC):
    @abstractmethod
    def draw(self): pass

# Protocol — только для mypy
from typing import Protocol
class DrawableProtocol(Protocol):
    def draw(self) -> None: ...

# Выбор: ABC если нужны isintance и реализация по умолчанию
# Protocol если нужна только статическая типизация
```

**Абстрактные классы с декораторами `@abstractmethod` и `@property`**

```python
class AbstractContainer(ABC):
    @property
    @abstractmethod
    def size(self):
        pass
    
    @abstractmethod
    def add(self, item):
        pass
    
    @abstractmethod
    def remove(self, item):
        pass

class SetContainer(AbstractContainer):
    def __init__(self):
        self._items = set()
    
    @property
    def size(self):
        return len(self._items)
    
    def add(self, item):
        self._items.add(item)
    
    def remove(self, item):
        self._items.discard(item)
```

**Антипаттерны ABC**

❌ **Слишком детализированная абстракция:**
```python
class TooAbstract(ABC):
    @abstractmethod
    def get_x(self): pass
    @abstractmethod
    def set_x(self, value): pass
    @abstractmethod
    def get_y(self): pass
    # ... 20 абстрактных методов
```
Лучше: разбить на несколько маленьких ABC или использовать композицию.

❌ **Абстрактные классы без абстрактных методов:**
```python
class EmptyABC(ABC):
    pass

# Бессмысленно — можно использовать обычный класс
```

❌ **Наследование от ABC без реализации:**
```python
class Incomplete(Shape):
    def area(self):          # Не реализован perimeter
        return 0

# Incomplete() - TypeError, но в большом коде легко пропустить
```

**Практический паттерн: шаблонный метод с ABC**

```python
class ReportGenerator(ABC):
    def generate(self, data):
        """Шаблонный метод"""
        self._prepare_data(data)
        content = self._format_content()
        self._output(content)
    
    def _prepare_data(self, data):
        """Общая реализация"""
        self.data = data
    
    @abstractmethod
    def _format_content(self):
        pass
    
    @abstractmethod
    def _output(self, content):
        pass

class HTMLReport(ReportGenerator):
    def _format_content(self):
        return f"<html><body>{self.data}</body></html>"
    
    def _output(self, content):
        with open("report.html", "w") as f:
            f.write(content)

class JSONReport(ReportGenerator):
    def _format_content(self):
        import json
        return json.dumps(self.data)
    
    def _output(self, content):
        print(content)

HTMLReport().generate({"key": "value"})
```

**Наследование от нескольких ABC**

```python
class Readable(ABC):
    @abstractmethod
    def read(self): pass

class Writable(ABC):
    @abstractmethod
    def write(self, data): pass

class ReadWrite(Readable, Writable):
    def read(self):
        return "data"
    
    def write(self, data):
        print(f"Writing {data}")

rw = ReadWrite()  # OK — все методы реализованы
```

**Итоговые правила использования ABC:**

| Сценарий | Решение |
|----------|---------|
| Нужен контракт для иерархии классов | ABC с `@abstractmethod` |
| Нужна реализация по умолчанию | Конкретный метод в ABC |
| Нужно запретить создание экземпляров базового класса | ABC без реализации всех методов |
| Нужна проверка через `isinstance` для чужих классов | `register()` или наследование от ABC |
| Только статическая типизация (mypy) | `Protocol` вместо ABC |
| Множественное наследование интерфейсов | Несколько ABC |

**Ключевое понимание:** ABC в Python — это компромисс между динамической природой языка и потребностью в контрактах. Они дают проверку на этапе создания экземпляра, могут содержать реализацию, но не добавляют строгости времени компиляции. Используйте ABC когда строите публичное API или сложную иерархию с обязательными методами. Для простых случаев достаточно утиной типизации.



**15. Композиция против наследования**

**Определение и ключевое различие**

**Наследование** — отношение "is-a" (является). Класс-потомок расширяет класс-родитель. **Композиция** — отношение "has-a" (имеет). Класс содержит другие классы как свои части.

```python
# Наследование (is-a) — "Автомобиль является транспортным средством"
class Vehicle:
    def move(self):
        return "Moving"

class Car(Vehicle):
    def drive(self):
        return "Driving"

# Композиция (has-a) — "Автомобиль имеет двигатель"
class Engine:
    def start(self):
        return "Engine started"

class Car:
    def __init__(self):
        self.engine = Engine()    # Композиция
    
    def start(self):
        return self.engine.start()
```

**Когда наследование — правильный выбор**

✅ **Явное отношение "is-a":**
```python
class Animal: pass
class Dog(Animal): pass      # Собака — это животное
class Cat(Animal): pass      # Кошка — это животное
```

✅ **Полиморфное поведение через переопределение:**
```python
class PaymentMethod:
    def pay(self, amount): raise NotImplementedError

class CreditCard(PaymentMethod):
    def pay(self, amount): return "Processing credit card"

class PayPal(PaymentMethod):
    def pay(self, amount): return "Processing PayPal"
```

✅ **Повторное использование интерфейса (абстрактные классы):**
```python
from abc import ABC, abstractmethod

class Plugin(ABC):
    @abstractmethod
    def execute(self): pass

class LoggingPlugin(Plugin):
    def execute(self): print("Logging")
```

✅ **Небольшая глубина иерархии (2-3 уровня):**
```python
class FileHandler: pass
class TextFileHandler(FileHandler): pass
class CSVHandler(TextFileHandler): pass
```

**Когда композиция предпочтительнее**

✅ **Отношение "has-a":**
```python
class CPU: pass
class RAM: pass
class Computer:
    def __init__(self):
        self.cpu = CPU()
        self.ram = RAM()      # Компьютер имеет CPU и RAM
```

✅ **Изменяемое поведение во время выполнения:**
```python
class Attack:
    def execute(self): pass

class SwordAttack(Attack):
    def execute(self): return "Slashing with sword"

class MagicAttack(Attack):
    def execute(self): return "Casting fireball"

class Character:
    def __init__(self, attack: Attack):
        self.attack = attack
    
    def fight(self):
        return self.attack.execute()

# Поведение можно менять динамически
warrior = Character(SwordAttack())
mage = Character(MagicAttack())
```

✅ **Избежание "хрупкого базового класса" (fragile base class):**
```python
# Проблема наследования
class Logger:
    def log(self, message):
        self._write(message)
    
    def _write(self, message):
        print(message)

class FileLogger(Logger):
    def _write(self, message):
        with open("log.txt", "a") as f:
            f.write(message)    # Переопределили внутренний метод

# Решение через композицию
class FileWriter:
    def write(self, message):
        with open("log.txt", "a") as f:
            f.write(message)

class Logger:
    def __init__(self, writer):
        self.writer = writer
    
    def log(self, message):
        self.writer.write(message)
```

✅ **Ограничение глубины наследования (God object):**
```python
# Плохо: глубокое наследование
class Animal: pass
class Mammal(Animal): pass
class Primate(Mammal): pass
class Human(Primate): pass
class Employee(Human): pass
class Manager(Employee): pass
class DepartmentManager(Manager): pass   # 7 уровней!

# Хорошо: композиция
class Person:
    def __init__(self, name):
        self.name = name

class Role:
    def __init__(self, title):
        self.title = title

class Employee:
    def __init__(self, person: Person, role: Role):
        self.person = person
        self.role = role
```

**Проблемы наследования**

**1. Проблема "ромба" и сложный MRO:**
```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass    # MRO требует понимания
```

**2. Нарушение инкапсуляции при переопределении:**
```python
class Counter:
    def __init__(self):
        self.count = 0
    
    def increment(self):
        self.count += 1
        self.on_increment()
    
    def on_increment(self):
        pass

class DoubleCounter(Counter):
    def increment(self):
        self.count += 2    # Изменили поведение
        # Не вызвали on_increment — сломали ожидание родителя
```

**3. Негибкость: нельзя изменить родителя во время выполнения:**
```python
class Car:
    def drive(self): pass

class SportsCar(Car): pass

# Нельзя заменить поведение drive после создания
```

**Паттерны композиции**

**1. Простая композиция (владение жизненным циклом):**
```python
class House:
    def __init__(self):
        self.doors = [Door() for _ in range(4)]
        self.windows = [Window() for _ in range(6)]
    # Компоненты уничтожаются вместе с House
```

**2. Агрегация (объекты живут независимо):**
```python
class University:
    def __init__(self, professors):
        self.professors = professors    # Ссылка на внешние объекты

prof = Professor("Smith")
uni = University([prof])    # prof живёт вне uni
```

**3. Делегирование:**
```python
class Printer:
    def print(self, text):
        print(f"Printing: {text}")

class Scanner:
    def scan(self):
        return "Scanned data"

class MultiFunctionDevice:
    def __init__(self):
        self.printer = Printer()
        self.scanner = Scanner()
    
    def print(self, text):
        self.printer.print(text)    # Делегирование
    
    def scan(self):
        return self.scanner.scan()
```

**Композиция + наследование = лучшее решение**

Часто лучший подход — комбинировать оба метода.

```python
# Базовый интерфейс через ABC
class Renderer(ABC):
    @abstractmethod
    def render(self, shape): pass

# Реализации через наследование
class VectorRenderer(Renderer):
    def render(self, shape):
        return f"Rendering {shape} as vectors"

class RasterRenderer(Renderer):
    def render(self, shape):
        return f"Rendering {shape} as pixels"

# Композиция в основном классе
class Shape:
    def __init__(self, renderer: Renderer):
        self.renderer = renderer
    
    def draw(self):
        return self.renderer.render(self.__class__.__name__)

class Circle(Shape):
    pass

class Square(Shape):
    pass

# Гибкость: можно комбинировать
circle_vector = Circle(VectorRenderer())
circle_raster = Circle(RasterRenderer())
```

**Стратегия выбора: наследование или композиция**

| Критерий | Наследование | Композиция |
|----------|--------------|------------|
| Отношение | is-a | has-a |
| Гибкость | Низкая (статическое) | Высокая (динамическое) |
| Связность | Высокая (жёсткая) | Низкая (слабая) |
| Повторное использование | Белое ящик (white-box) | Чёрный ящик (black-box) |
| Изменение в рантайме | Невозможно | Возможно |
| Код для простых случаев | Меньше | Больше |
| Риск поломки родителя | Высокий | Низкий |

**Рефакторинг: наследование → композиция**

```python
# Исходный код с наследованием
class Stack(list):
    def push(self, item):
        self.append(item)
    
    def pop(self):
        if not self:
            raise IndexError
        return super().pop()

# Проблема: наследует все методы list (insert, extend и т.д.)

# Рефакторинг: композиция
class Stack:
    def __init__(self):
        self._items = []      # Скрытая реализация
    
    def push(self, item):
        self._items.append(item)
    
    def pop(self):
        if not self._items:
            raise IndexError
        return self._items.pop()
    
    def peek(self):
        if not self._items:
            return None
        return self._items[-1]
    
    def __len__(self):
        return len(self._items)
```

**Антипаттерны**

❌ **Наследование ради повторного использования одного метода:**
```python
class Calculator:
    def add(self, a, b): return a + b

class MyClass(Calculator):    # Нет логического "is-a"
    def process(self):
        return self.add(1, 2)

# Лучше: композиция
class MyClass:
    def __init__(self):
        self.calc = Calculator()
    
    def process(self):
        return self.calc.add(1, 2)
```

❌ **Глубокая иерархия "лесенкой":**
```python
class A: pass
class B(A): pass
class C(B): pass
class D(C): pass
class E(D): pass   # 5 уровней — признак плохого дизайна
```

❌ **Нарушение принципа подстановки Лисков (LSP):**
```python
class Bird:
    def fly(self): return "Flying"

class Penguin(Bird):
    def fly(self):
        raise NotImplementedError("Penguins can't fly")
# Пингвин не может заменить Bird без поломки

# Решение: разделить интерфейсы
class Flyable(ABC):
    @abstractmethod
    def fly(self): pass

class Bird: pass

class Sparrow(Bird, Flyable):
    def fly(self): return "Flying"

class Penguin(Bird): pass    # Не реализует Flyable
```

**Практические рекомендации**

1. **Начинайте с композиции.** Если становится неудобно — рефакторите до наследования.

2. **Наследование — для полиморфизма, композиция — для переиспользования кода.**

3. **Глубина наследования > 3 — красный флаг.**

4. **Тест на наследование:** можно ли сказать "X — это Y" и будет ли это правдой во всех контекстах?

5. **Предпочитайте интерфейсы (ABC) вместо реализации при наследовании.**

```python
# Лучше
class Drawable(ABC):
    @abstractmethod
    def draw(self): pass

class Circle(Drawable):
    def draw(self): ...
```

```python
# А не так
class Drawable:
    def draw(self):
        return "default"    # Реализация по умолчанию — соблазн переиспользовать
```

**Итог:** Favour composition over inheritance — это не абсолютное правило, а руководство. Наследование даёт простой полиморфизм, но жёстко связывает классы. Композиция гибче, слабее связывает, позволяет менять поведение в рантайме, но требует больше кода. Используйте наследование для настоящих "is-a" отношений и когда полиморфизм — главная цель. Во всех остальных случаях — композиция.



**16. Магические (Dunder) методы: `__str__`, `__repr__`, `__eq__`, `__lt__`**

**Определение Dunder-методов**

Dunder (Double UNDERscore) методы — специальные методы, окружённые двойным подчёркиванием. Они переопределяют поведение объектов для встроенных операций: преобразование в строку, сравнение, арифметику, вызовы и другие.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __str__(self):
        return f"Point({self.x}, {self.y})"
    
    def __repr__(self):
        return f"Point({self.x}, {self.y})"

p = Point(3, 5)
print(str(p))      # Point(3, 5) — вызывает __str__
print(repr(p))     # Point(3, 5) — вызывает __repr__
```

**`__str__` и `__repr__` — строковое представление**

**`__str__`** — неформальное, "человекочитаемое" представление. Используется в `print()`, `str()`, f-строках.

**`__repr__`** — формальное, "машиночитаемое" представление. Используется в отладчике, REPL, `repr()`. Должно быть однозначным и, по возможности, воспроизводимым.

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __str__(self):
        return f"{self.name}, {self.age} years old"
    
    def __repr__(self):
        return f"Person('{self.name}', {self.age})"

p = Person("Alice", 30)
print(p)                    # Alice, 30 years old (__str__)
print(repr(p))              # Person('Alice', 30) (__repr__)
print(f"{p}")               # Alice, 30 years old
print(f"{p!r}")             # Person('Alice', 30) — форсируем repr
```

**Правила реализации:**

| Метод | Назначение | Должен возвращать | Пример |
|-------|------------|-------------------|--------|
| `__str__` | Для пользователя | Краткая читаемая строка | `"Apple (3 items)"` |
| `__repr__` | Для разработчика | Однозначное представление, часто код создания | `"Fruit('Apple', 3)"` |

**Паттерн: если нужно только одно представление, реализуйте `__repr__`, а `__str__` сделайте его алиасом:**

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius
    
    def __repr__(self):
        return f"Temperature({self.celsius})"
    
    def __str__(self):
        return repr(self)      # делегирование

t = Temperature(25)
print(t)          # Temperature(25)
print(repr(t))    # Temperature(25)
```

**`__eq__` — сравнение на равенство (==)**

По умолчанию `==` сравнивает идентичность объектов (как `is`). Переопределив `__eq__`, можно сравнивать по содержимому.

```python
class Book:
    def __init__(self, title, author, isbn):
        self.title = title
        self.author = author
        self.isbn = isbn
    
    def __eq__(self, other):
        if not isinstance(other, Book):
            return NotImplemented
        return self.isbn == other.isbn

b1 = Book("Python Crash Course", "Eric Matthes", "978-1593279288")
b2 = Book("Python Crash Course", "Eric Matthes", "978-1593279288")
b3 = Book("Fluent Python", "Luciano Ramalho", "978-1491946008")

print(b1 == b2)    # True — сравниваем по isbn
print(b1 == b3)    # False
print(b1 == "book")  # False (NotImplemented → Python пробует обратный вызов)
```

**Важно:** `__eq__` должен возвращать `NotImplemented` (не `raise NotImplementedError`), если тип не поддерживается. Это позволяет Python попробовать обратное сравнение (`other == self`).

**`__hash__` — хеширование для использования в множествах и словарях**

Если переопределён `__eq__`, следует переопределить и `__hash__`. Объект, который может быть ключом словаря, должен быть неизменяемым.

```python
class Book:
    def __init__(self, title, isbn):
        self.title = title
        self.isbn = isbn
    
    def __eq__(self, other):
        if not isinstance(other, Book):
            return NotImplemented
        return self.isbn == other.isbn
    
    def __hash__(self):
        return hash(self.isbn)   # хеш на основе isbn

b1 = Book("Python", "123")
b2 = Book("Python", "123")
books = {b1, b2}
print(len(books))    # 1 — дубликат удалён
```

**Правило:** если объект неизменяемый и переопределён `__eq__`, переопределите и `__hash__`. Если изменяемый — установите `__hash__ = None`, чтобы запретить использование в качестве ключа.

```python
class MutablePoint:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    
    __hash__ = None   # объект нельзя использовать как ключ

# d = {MutablePoint(1,2): "value"}  # TypeError: unhashable type
```

**`__lt__`, `__le__`, `__gt__`, `__ge__` — сравнения (<, <=, >, >=)**

Реализовав `__lt__` и `__eq__`, можно получить все сравнения через `@functools.total_ordering`.

```python
from functools import total_ordering

@total_ordering
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
    
    def __eq__(self, other):
        if not isinstance(other, Product):
            return NotImplemented
        return self.price == other.price
    
    def __lt__(self, other):
        if not isinstance(other, Product):
            return NotImplemented
        return self.price < other.price
    
    def __repr__(self):
        return f"{self.name}: ${self.price}"

p1 = Product("Apple", 1.5)
p2 = Product("Banana", 0.9)
p3 = Product("Cherry", 2.0)

print(p1 > p2)    # True (декоратор генерирует __gt__ из __lt__ и __eq__)
print(sorted([p1, p2, p3]))  # [Banana: $0.9, Apple: $1.5, Cherry: $2.0]
```

**Без `@total_ordering` нужно реализовать все методы вручную:**

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius
    
    def __eq__(self, other):
        return self.celsius == other.celsius
    
    def __lt__(self, other):
        return self.celsius < other.celsius
    
    def __le__(self, other):
        return self.celsius <= other.celsius
    
    def __gt__(self, other):
        return self.celsius > other.celsius
    
    def __ge__(self, other):
        return self.celsius >= other.celsius
```

**Полный пример с корзиной товаров**

```python
from functools import total_ordering

@total_ordering
class Item:
    def __init__(self, name, price, quantity=1):
        self.name = name
        self.price = price
        self.quantity = quantity
    
    def total(self):
        return self.price * self.quantity
    
    def __eq__(self, other):
        if not isinstance(other, Item):
            return NotImplemented
        return self.name == other.name
    
    def __lt__(self, other):
        if not isinstance(other, Item):
            return NotImplemented
        return self.total() < other.total()
    
    def __str__(self):
        return f"{self.name} x{self.quantity} = ${self.total():.2f}"
    
    def __repr__(self):
        return f"Item('{self.name}', {self.price}, {self.quantity})"

class Cart:
    def __init__(self):
        self.items = []
    
    def add(self, item):
        self.items.append(item)
    
    def __len__(self):
        return len(self.items)
    
    def __str__(self):
        if not self.items:
            return "Empty cart"
        return "\n".join(str(item) for item in self.items)
    
    def __repr__(self):
        return f"Cart({repr(self.items)})"

cart = Cart()
cart.add(Item("Apple", 0.5, 3))
cart.add(Item("Banana", 0.3, 5))
print(cart)          # Apple x3 = $1.50\nBanana x5 = $1.50
print(repr(cart))    # Cart([Item('Apple', 0.5, 3), Item('Banana', 0.3, 5)])
print(len(cart))     # 2
```

**Распространённые ошибки**

❌ **`__eq__` возвращает `False` для несовместимых типов вместо `NotImplemented`:**
```python
# Плохо
def __eq__(self, other):
    if not isinstance(other, Book):
        return False    # Ломает обратное сравнение

# Хорошо
def __eq__(self, other):
    if not isinstance(other, Book):
        return NotImplemented
    return self.isbn == other.isbn
```

❌ **Забытый `__hash__` при переопределённом `__eq__`:**
```python
class Broken:
    def __init__(self, value):
        self.value = value
    
    def __eq__(self, other):
        return self.value == other.value
    # __hash__ не переопределён → объект остаётся хешируемым,
    # но хеш не соответствует равенству

b1, b2 = Broken(1), Broken(1)
print(hash(b1) == hash(b2))  # В общем случае False — проблемы в множествах
```

❌ **Мутируемый объект с `__hash__`:**
```python
class BadKey:
    def __init__(self, value):
        self.value = value
    
    def __hash__(self):
        return hash(self.value)   # value может измениться!

k = BadKey(1)
d = {k: "data"}
k.value = 2
# d[k] теперь может не найти значение — ключ изменился
```

**Итоговые правила**

| Dunder-метод | Когда переопределять | Что возвращать |
|--------------|---------------------|----------------|
| `__str__` | Нужно красивое представление для пользователя | `str` |
| `__repr__` | Нужно отладочное представление (всегда полезно) | `str`, часто `f"{type(self).__name__}(...)"` |
| `__eq__` | Нужно сравнение по содержимому | `bool` или `NotImplemented` |
| `__hash__` | Объект неизменяемый и будет ключом словаря | `int` |
| `__lt__` | Нужна сортировка объекта | `bool` или `NotImplemented` |

**Стандартный шаблон для классов-значений:**

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return f"{type(self).__name__}({self.x}, {self.y})"
    
    def __str__(self):
        return f"({self.x}, {self.y})"
    
    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y
    
    def __hash__(self):
        return hash((self.x, self.y))
    
    def __lt__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return (self.x, self.y) < (other.x, other.y)
```

**Ключевая мысль:** магические методы делают ваши объекты "гражданами первого класса" в Python — они получают возможность работать с встроенными функциями, операторами и синтаксисом языка. Реализуйте `__repr__` всегда — это спасёт при отладке. `__str__` — для человекочитаемых выводов. `__eq__` и `__hash__` — для корректной работы в коллекциях.



**17. Перегрузка операторов через магические методы**

**Определение перегрузки операторов**

Перегрузка операторов позволяет определять поведение стандартных операторов (`+`, `-`, `*`, `/`, `[]`, `()` и др.) для пользовательских классов. В Python это реализуется через специальные (dunder) методы.

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    
    def __str__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
v3 = v1 + v2        # Вызывает v1.__add__(v2)
print(v3)           # Vector(4, 6)
```

**Арифметические операторы**

| Оператор | Метод | Описание |
|----------|-------|----------|
| `+` | `__add__(self, other)` | Сложение |
| `-` | `__sub__(self, other)` | Вычитание |
| `*` | `__mul__(self, other)` | Умножение |
| `/` | `__truediv__(self, other)` | Деление (float) |
| `//` | `__floordiv__(self, other)` | Целочисленное деление |
| `%` | `__mod__(self, other)` | Остаток от деления |
| `**` | `__pow__(self, other)` | Возведение в степень |
| `@` | `__matmul__(self, other)` | Матричное умножение (Python 3.5+) |

```python
class Money:
    def __init__(self, amount, currency="USD"):
        self.amount = amount
        self.currency = currency
    
    def __add__(self, other):
        if self.currency != other.currency:
            raise ValueError("Different currencies")
        return Money(self.amount + other.amount, self.currency)
    
    def __sub__(self, other):
        if self.currency != other.currency:
            raise ValueError("Different currencies")
        return Money(self.amount - other.amount, self.currency)
    
    def __mul__(self, factor):
        if not isinstance(factor, (int, float)):
            return NotImplemented
        return Money(self.amount * factor, self.currency)
    
    def __truediv__(self, divisor):
        return Money(self.amount / divisor, self.currency)
    
    def __repr__(self):
        return f"{self.amount} {self.currency}"

m1 = Money(100, "USD")
m2 = Money(50, "USD")
print(m1 + m2)      # 150 USD
print(m1 * 2)       # 200 USD
print(m2 / 2)       # 25.0 USD
```

**Рефлексивные (обратные) операторы**

Когда левый операнд не поддерживает операцию, Python пробует правый с рефлексивным методом.

| Оператор | Рефлексивный метод |
|----------|-------------------|
| `+` | `__radd__(self, other)` |
| `-` | `__rsub__(self, other)` |
| `*` | `__rmul__(self, other)` |
| `/` | `__rtruediv__(self, other)` |

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __mul__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Vector(self.x * scalar, self.y * scalar)
        return NotImplemented
    
    def __rmul__(self, scalar):
        return self.__mul__(scalar)   # скаляр * вектор = вектор * скаляр
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v = Vector(2, 3)
print(v * 4)        # Vector(8, 12) — v.__mul__(4)
print(4 * v)        # Vector(8, 12) — 4.__mul__(v) → NotImplemented → v.__rmul__(4)
```

**Инкрементальные операторы (+=, -= и др.)**

| Оператор | Метод |
|----------|-------|
| `+=` | `__iadd__(self, other)` |
| `-=` | `__isub__(self, other)` |
| `*=` | `__imul__(self, other)` |
| `/=` | `__itruediv__(self, other)` |

Если `__iadd__` не определён, Python использует `__add__` и присваивание.

```python
class Inventory:
    def __init__(self, items=None):
        self.items = items or {}
    
    def __iadd__(self, other):
        """Поддерживает += для добавления товаров"""
        if isinstance(other, dict):
            for item, count in other.items():
                self.items[item] = self.items.get(item, 0) + count
        elif isinstance(other, tuple):
            item, count = other
            self.items[item] = self.items.get(item, 0) + count
        else:
            return NotImplemented
        return self  # Должен вернуть self
    
    def __repr__(self):
        return str(self.items)

inv = Inventory({"apple": 5})
inv += {"banana": 3}
inv += ("apple", 2)
print(inv)          # {'apple': 7, 'banana': 3}
```

**Унарные операторы**

| Оператор | Метод | Описание |
|----------|-------|----------|
| `-obj` | `__neg__(self)` | Унарный минус |
| `+obj` | `__pos__(self)` | Унарный плюс |
| `~obj` | `__invert__(self)` | Побитовое НЕ |
| `abs(obj)` | `__abs__(self)` | Абсолютное значение |

```python
class Coordinate:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __neg__(self):
        return Coordinate(-self.x, -self.y)
    
    def __abs__(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5
    
    def __repr__(self):
        return f"({self.x}, {self.y})"

c = Coordinate(3, 4)
print(-c)      # (-3, -4)
print(abs(c))  # 5.0
```

**Операторы сравнения (краткое повторение)**

| Оператор | Метод |
|----------|-------|
| `==` | `__eq__(self, other)` |
| `!=` | `__ne__(self, other)` |
| `<` | `__lt__(self, other)` |
| `<=` | `__le__(self, other)` |
| `>` | `__gt__(self, other)` |
| `>=` | `__ge__(self, other)` |

```python
class Fraction:
    def __init__(self, numerator, denominator):
        self.num = numerator
        self.den = denominator
    
    def __eq__(self, other):
        return self.num * other.den == other.num * self.den
    
    def __lt__(self, other):
        return self.num * other.den < other.num * self.den
    
    def __repr__(self):
        return f"{self.num}/{self.den}"

f1 = Fraction(1, 2)
f2 = Fraction(2, 4)
print(f1 == f2)   # True
print(f1 < Fraction(3, 4))  # True
```

**Индексация и срезы (`[]`)**

| Оператор | Метод |
|----------|-------|
| `obj[key]` | `__getitem__(self, key)` |
| `obj[key] = value` | `__setitem__(self, key, value)` |
| `del obj[key]` | `__delitem__(self, key)` |

```python
class Matrix:
    def __init__(self, data):
        self.data = data  # список списков
    
    def __getitem__(self, index):
        if isinstance(index, tuple):
            i, j = index
            return self.data[i][j]
        return self.data[index]
    
    def __setitem__(self, index, value):
        if isinstance(index, tuple):
            i, j = index
            self.data[i][j] = value
        else:
            self.data[index] = value
    
    def __repr__(self):
        return "\n".join(str(row) for row in self.data)

m = Matrix([[1, 2, 3], [4, 5, 6]])
print(m[0, 1])      # 2
m[1, 2] = 99
print(m)
# [1, 2, 3]
# [4, 5, 99]
```

**Обработка срезов через `slice`:**

```python
class CustomList:
    def __init__(self, items):
        self._items = items
    
    def __getitem__(self, key):
        if isinstance(key, slice):
            start, stop, step = key.start, key.stop, key.step
            return self._items[start:stop:step]
        return self._items[key]
    
    def __setitem__(self, key, value):
        if isinstance(key, slice):
            self._items[key] = value
        else:
            self._items[key] = value

cl = CustomList([0, 1, 2, 3, 4, 5])
print(cl[1:4])      # [1, 2, 3]
cl[2:5] = [20, 30, 40]
print(cl._items)    # [0, 1, 20, 30, 40, 5]
```

**Вызов объекта как функции `()`**

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor
    
    def __call__(self, value):
        return value * self.factor

double = Multiplier(2)
triple = Multiplier(3)

print(double(5))    # 10
print(triple(5))    # 15
print(callable(double))  # True
```

**Длина и булево значение**

```python
class Playlist:
    def __init__(self, songs):
        self.songs = songs
    
    def __len__(self):
        return len(self.songs)
    
    def __bool__(self):
        return len(self) > 0

p1 = Playlist([])
p2 = Playlist(["Song 1"])

print(len(p1))      # 0
print(bool(p1))     # False
print(bool(p2))     # True
```

**Полный пример: комплексные числа**

```python
class Complex:
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag
    
    def __add__(self, other):
        return Complex(self.real + other.real, self.imag + other.imag)
    
    def __sub__(self, other):
        return Complex(self.real - other.real, self.imag - other.imag)
    
    def __mul__(self, other):
        return Complex(
            self.real * other.real - self.imag * other.imag,
            self.real * other.imag + self.imag * other.real
        )
    
    def __abs__(self):
        return (self.real ** 2 + self.imag ** 2) ** 0.5
    
    def __eq__(self, other):
        return self.real == other.real and self.imag == other.imag
    
    def __repr__(self):
        sign = '+' if self.imag >= 0 else ''
        return f"{self.real}{sign}{self.imag}i"

c1 = Complex(1, 2)
c2 = Complex(3, -4)
print(c1 + c2)      # 4-2i
print(c1 * c2)      # 11+2i
print(abs(c1))      # 2.236...
```

**Операторы сравнения с `NotImplemented`**

Всегда возвращайте `NotImplemented` (не `raise` и не `False`), если тип не поддерживается.

```python
class Distance:
    def __init__(self, meters):
        self.meters = meters
    
    def __add__(self, other):
        if isinstance(other, Distance):
            return Distance(self.meters + other.meters)
        if isinstance(other, (int, float)):
            return Distance(self.meters + other)
        return NotImplemented
    
    def __radd__(self, other):
        return self.__add__(other)

d = Distance(10)
print(d + 5)        # 15 м
print(5 + d)        # 15 м — через __radd__
print(d + "hello")  # TypeError (NotImplemented → ошибка)
```

**Практические ограничения**

❌ **Не перегружайте операторы для неочевидного поведения:**

```python
# Плохо
class User:
    def __add__(self, other):
        return User(self.name + other.name)  # Что это значит?

# Хорошо
class User:
    def merge(self, other):
        return User(f"{self.name} & {other.name}")
```

❌ **Нарушение математических ожиданий:**

```python
# Плохо — коммутативность нарушена
class Bad:
    def __mul__(self, other):
        return Bad()
    def __rmul__(self, other):
        return "different"  # v * 2 и 2 * v дают разное
```

✅ **Используйте `@total_ordering` для сравнений:**

```python
from functools import total_ordering

@total_ordering
class Version:
    def __init__(self, major, minor):
        self.major = major
        self.minor = minor
    
    def __eq__(self, other):
        return (self.major, self.minor) == (other.major, other.minor)
    
    def __lt__(self, other):
        return (self.major, self.minor) < (other.major, other.minor)

# Остальные сравнения генерируются автоматически
```

**Итоговая таблица часто используемых операторов**

| Категория | Методы |
|-----------|--------|
| Арифметика | `__add__`, `__sub__`, `__mul__`, `__truediv__` |
| Рефлексивные | `__radd__`, `__rsub__`, `__rmul__` |
| Инкрементальные | `__iadd__`, `__isub__`, `__imul__` |
| Сравнения | `__eq__`, `__lt__`, `__le__`, `__gt__`, `__ge__` |
| Индексация | `__getitem__`, `__setitem__`, `__delitem__` |
| Вызов | `__call__` |
| Унарные | `__neg__`, `__pos__`, `__abs__` |
| Преобразования | `__int__`, `__float__`, `__bool__`, `__len__` |

**Ключевая мысль:** перегрузка операторов делает пользовательские объекты интуитивно понятными. Пользователь вашего класса ожидает, что `+` будет складывать, а `*` — умножать. Нарушение этих ожиданий ведёт к трудноуловимых ошибкам. Перегружайте операторы только тогда, когда операция естественна, однозначна и соответствует математическому или логическому смыслу.



**18. Пользовательские исключения в контексте ООП**

**Определение пользовательского исключения**

Пользовательские исключения — классы, наследующие от встроенного `Exception` (или его подклассов). Они позволяют моделировать специфические ошибки предметной области, делая код самодокументированным и упрощая обработку.

```python
class BankAccountError(Exception):
    """Базовое исключение для банковского счёта"""
    pass

class InsufficientFundsError(BankAccountError):
    """Средств на счёте недостаточно"""
    pass

class AccountNotFoundError(BankAccountError):
    """Счёт не найден"""
    pass

class BankAccount:
    def __init__(self, account_id, balance=0):
        self.account_id = account_id
        self.balance = balance
    
    def withdraw(self, amount):
        if amount > self.balance:
            raise InsufficientFundsError(
                f"Account {self.account_id}: need {amount}, have {self.balance}"
            )
        self.balance -= amount

acc = BankAccount("123", 100)
try:
    acc.withdraw(200)
except InsufficientFundsError as e:
    print(f"Error: {e}")  # Error: Account 123: need 200, have 100
```

**Иерархия пользовательских исключений**

Создавайте иерархию для группировки связанных ошибок.

```python
# Базовое исключение модуля
class ValidationError(Exception):
    """Ошибка валидации данных"""
    pass

# Конкретные ошибки
class RequiredFieldError(ValidationError):
    """Отсутствует обязательное поле"""
    def __init__(self, field_name):
        self.field_name = field_name
        super().__init__(f"Field '{field_name}' is required")

class InvalidTypeError(ValidationError):
    """Неверный тип данных"""
    def __init__(self, field_name, expected, got):
        self.field_name = field_name
        self.expected = expected
        self.got = got
        super().__init__(
            f"Field '{field_name}': expected {expected}, got {type(got).__name__}"
        )

class RangeError(ValidationError):
    """Значение вне допустимого диапазона"""
    def __init__(self, field_name, min_val, max_val, actual):
        self.field_name = field_name
        self.min = min_val
        self.max = max_val
        self.actual = actual
        super().__init__(
            f"Field '{field_name}': value {actual} out of range [{min_val}, {max_val}]"
        )

# Использование
def validate_user(name, age):
    if not name:
        raise RequiredFieldError("name")
    if not isinstance(age, int):
        raise InvalidTypeError("age", int, age)
    if age < 0 or age > 150:
        raise RangeError("age", 0, 150, age)

try:
    validate_user("", 200)
except RequiredFieldError as e:
    print(f"Missing field: {e.field_name}")
except RangeError as e:
    print(f"Age {e.actual} out of range {e.min}-{e.max}")
except ValidationError as e:
    print(f"Validation failed: {e}")
```

**Добавление данных в исключение**

Передавайте контекст ошибки через атрибуты, а не только через строку сообщения.

```python
class DatabaseError(Exception):
    def __init__(self, query, params, original_exception=None):
        self.query = query
        self.params = params
        self.original = original_exception
        super().__init__(f"Query failed: {query}")
    
    def __str__(self):
        base = super().__str__()
        if self.original:
            return f"{base}\nOriginal error: {self.original}"
        return base

class Repository:
    def execute(self, query, params=None):
        try:
            # Симуляция ошибки БД
            raise ConnectionError("Connection timeout")
        except ConnectionError as e:
            raise DatabaseError(query, params, e)

try:
    repo = Repository()
    repo.execute("SELECT * FROM users", {"limit": 10})
except DatabaseError as e:
    print(f"Query: {e.query}")
    print(f"Params: {e.params}")
    print(f"Cause: {e.original}")
    # Повторная обработка с сохранением контекста
```

**Исключения в иерархии классов**

```python
class PaymentProcessor:
    class ProcessingError(Exception):
        pass
    
    class NetworkError(ProcessingError):
        pass
    
    class ValidationError(ProcessingError):
        pass

class StripeProcessor(PaymentProcessor):
    def process(self, amount, card_data):
        if not card_data.get('number'):
            raise self.ValidationError("Card number required")
        if not self._connect():
            raise self.NetworkError("Stripe unreachable")
        # Обработка платежа
    
    def _connect(self):
        return False

try:
    processor = StripeProcessor()
    processor.process(100, {})
except PaymentProcessor.ValidationError as e:
    print(f"Invalid input: {e}")
except PaymentProcessor.NetworkError as e:
    print(f"Network issue: {e}")
```

**Исключения и наследование: порядок `except` важен**

```python
class AppError(Exception): pass
class ConfigError(AppError): pass
class DatabaseError(AppError): pass

def risky_operation(mode):
    if mode == "config":
        raise ConfigError("Bad config")
    elif mode == "db":
        raise DatabaseError("Connection failed")
    else:
        raise AppError("Unknown error")

try:
    risky_operation("config")
except ConfigError as e:          # Должен быть ПЕРВЫМ
    print(f"Config issue: {e}")
except DatabaseError as e:
    print(f"DB issue: {e}")
except AppError as e:
    print(f"General error: {e}")

# Ошибка: если бы AppError был первым, ConfigError никогда не поймался бы
```

**Паттерн: исключения как замена флагов возврата**

```python
# Плохо: флаги возврата
def divide_flag(a, b):
    if b == 0:
        return (None, "Division by zero")
    return (a / b, None)

result, error = divide_flag(10, 0)
if error:
    print(error)

# Хорошо: исключения
def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return a / b

try:
    result = divide(10, 0)
except ZeroDivisionError as e:
    print(e)
```

**Исключения для контроля потока (антипаттерн)**

```python
# Очень плохо: исключения для нормального потока
class StopIterationSentinel(Exception):
    pass

def find_first(items, predicate):
    for item in items:
        if predicate(item):
            return item
    raise StopIterationSentinel()

# Почему плохо: исключения дороги, скрывают логику, усложняют отладку
```

**Исключения в конструкторах (`__init__`)**

```python
class Temperature:
    def __init__(self, celsius):
        if celsius < -273.15:
            raise ValueError(f"Temperature {celsius} below absolute zero")
        self._celsius = celsius
    
    def __setattr__(self, name, value):
        if name == '_celsius' and value < -273.15:
            raise ValueError("Temperature below absolute zero")
        super().__setattr__(name, value)

try:
    t = Temperature(-500)
except ValueError as e:
    print(e)  # Temperature -500 below absolute zero
```

**Исключения в свойствах (`@property`)**

```python
class BankAccount:
    def __init__(self, balance=0):
        self._balance = balance
    
    @property
    def balance(self):
        return self._balance
    
    @balance.setter
    def balance(self, value):
        if value < 0:
            raise InsufficientFundsError("Balance cannot be negative")
        self._balance = value

class InsufficientFundsError(Exception):
    pass

acc = BankAccount(100)
try:
    acc.balance = -50
except InsufficientFundsError as e:
    print(e)  # Balance cannot be negative
```

**Цепочки исключений (`raise ... from ...`)**

Сохраняет контекст исходной ошибки.

```python
class DataLoadError(Exception):
    pass

def load_data(filename):
    try:
        with open(filename) as f:
            return f.read()
    except FileNotFoundError as e:
        raise DataLoadError(f"Config {filename} missing") from e
    except PermissionError as e:
        raise DataLoadError(f"No permission to read {filename}") from e

try:
    config = load_data("config.txt")
except DataLoadError as e:
    print(f"Error: {e}")
    print(f"Original: {e.__cause__}")  # Доступ к исходной ошибке
```

**Переопределение `__str__` и `__repr__` в исключениях**

```python
class ValidationError(Exception):
    def __init__(self, errors):
        self.errors = errors  # словарь {поле: сообщение}
    
    def __str__(self):
        lines = ["Validation failed:"]
        for field, msg in self.errors.items():
            lines.append(f"  - {field}: {msg}")
        return "\n".join(lines)
    
    def __repr__(self):
        return f"ValidationError({self.errors})"

try:
    raise ValidationError({
        "email": "Invalid format",
        "age": "Must be at least 18"
    })
except ValidationError as e:
    print(str(e))
    # Validation failed:
    #   - email: Invalid format
    #   - age: Must be at least 18
```

**Пользовательские исключения с контекстом выполнения**

```python
import inspect
from datetime import datetime

class TransactionError(Exception):
    def __init__(self, message, transaction_id, original=None):
        self.transaction_id = transaction_id
        self.timestamp = datetime.now()
        self.call_stack = inspect.stack()
        self.original = original
        super().__init__(message)
    
    def log(self):
        print(f"[{self.timestamp}] TX {self.transaction_id}: {self}")
        if self.original:
            print(f"  Caused by: {self.original}")

class PaymentGateway:
    def charge(self, tx_id, amount):
        try:
            # Симуляция ошибки
            raise ConnectionError("Timeout")
        except ConnectionError as e:
            raise TransactionError(
                f"Payment failed for ${amount}",
                tx_id,
                e
            )

try:
    pg = PaymentGateway()
    pg.charge("TX-123", 99.99)
except TransactionError as e:
    e.log()
```

**Практические рекомендации**

| Сценарий | Решение |
|----------|---------|
| Базовое исключение модуля | Наследуйте от `Exception`, назовите `ModuleNameError` |
| Группировка ошибок | Создайте иерархию от базового исключения |
| Добавление данных ошибки | Переопределите `__init__`, сохраните атрибуты |
| Сохранение причины | Используйте `raise NewError() from original` |
| Исключения в библиотеке | Экспортируйте их в `__init__.py` |
| Исключения в приложении | Создайте корневое исключение приложения |

**Стандартные базовые классы для наследования**

```python
# Для проверки состояния
class AssertionError(Exception): pass

# Для ошибок типов
class TypeError(Exception): pass

# Для значений вне диапазона
class ValueError(Exception): pass

# Для операций с неверным состоянием объекта
class RuntimeError(Exception): pass

# Пользовательское
class MyAppError(RuntimeError): pass
```

**Итог:**

Пользовательские исключения в ООП — это не просто строки сообщений, а полноправные классы с поведением. Они позволяют:
- Группировать ошибки в иерархии для точной обработки
- Передавать структурированные данные (атрибуты) об ошибке
- Строить цепочки причин для отладки
- Делать API самодокументированным через типы исключений

Правило: создавайте новое исключение, когда встроенные не несут достаточного контекста для вашей предметной области. Но не плодите исключения без необходимости — одно общее исключение часто лучше десяти, которые никогда не различаются в обработке.



**19. Датаклассы (`@dataclass`) для сокращения шаблонного кода**

**Определение датакласса**

Датакласс — декоратор, автоматически генерирующий специальные методы (`__init__`, `__repr__`, `__eq__`, `__hash__` и др.) на основе аннотированных атрибутов. Доступен с Python 3.7.

```python
# Без датакласса — 15 строк шаблонного кода
class Point:
    def __init__(self, x, y, z=0):
        self.x = x
        self.y = y
        self.z = z
    
    def __repr__(self):
        return f"Point(x={self.x}, y={self.y}, z={self.z})"
    
    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return (self.x, self.y, self.z) == (other.x, other.y, other.z)

# С датаклассом — 3 строки
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float
    z: float = 0  # Значение по умолчанию

p1 = Point(1, 2)
p2 = Point(1, 2, 0)
print(p1)        # Point(x=1, y=2, z=0)
print(p1 == p2)  # True
```

**Базовые параметры декоратора `@dataclass`**

| Параметр | Значение по умолчанию | Описание |
|----------|----------------------|----------|
| `init` | `True` | Генерировать `__init__` |
| `repr` | `True` | Генерировать `__repr__` |
| `eq` | `True` | Генерировать `__eq__` |
| `order` | `False` | Генерировать `__lt__`, `__le__`, `__gt__`, `__ge__` |
| `unsafe_hash` | `False` | Генерировать `__hash__` (небезопасно при изменяемости) |
| `frozen` | `False` | Запретить изменение атрибутов после создания |

```python
@dataclass(order=True, frozen=True)
class Version:
    major: int
    minor: int
    patch: int

v1 = Version(1, 2, 0)
v2 = Version(1, 2, 1)
print(v1 < v2)    # True — порядок сгенерирован
# v1.major = 2    # FrozenInstanceError — изменить нельзя
```

**Атрибуты с типами и значениями по умолчанию**

```python
from dataclasses import dataclass, field
from typing import List, Dict, Optional

@dataclass
class User:
    name: str
    age: int = 18  # Значение по умолчанию
    
    # Атрибуты со значением по умолчанию должны идти после обязательных
    email: Optional[str] = None
    tags: List[str] = field(default_factory=list)  # Изменяемый тип — factory
    metadata: Dict[str, str] = field(default_factory=dict)
    
    # Поле, исключённое из __init__
    created_at: float = field(init=False, default_factory=time.time)

u = User("Alice", 25)
print(u)  # User(name='Alice', age=25, email=None, tags=[], metadata={}, created_at=164...)
```

**Важное правило:** нельзя использовать изменяемые значения по умолчанию напрямую (`field(default_factory=list)` вместо `default=[]`).

```python
# Плохо — общий список для всех экземпляров
@dataclass
class Bad:
    items: List[int] = []  # Опасно!

# Хорошо — новая копия для каждого экземпляра
@dataclass
class Good:
    items: List[int] = field(default_factory=list)
```

**Поле `field()` — полный контроль**

```python
from dataclasses import dataclass, field

@dataclass
class Product:
    name: str
    price: float
    
    # Скрытое поле (не в __init__, не в __repr__)
    _id: int = field(init=False, repr=False, default=0)
    
    # Поле с метаданными
    category: str = field(default="general", metadata={"priority": 1})
    
    # Вычисляемое поле (инициализируется после __init__)
    tax: float = field(init=False)
    
    def __post_init__(self):
        """Вызывается после автоматического __init__"""
        self.tax = self.price * 0.2
        self._id = hash(self.name)

p = Product("Laptop", 1000)
print(p)        # Product(name='Laptop', price=1000, category='general')
print(p.tax)    # 200.0
```

**Наследование датаклассов**

```python
@dataclass
class Person:
    name: str
    age: int

@dataclass
class Employee(Person):
    employee_id: str
    department: str = "General"

e = Employee("Bob", 30, "E123")
print(e)  # Employee(name='Bob', age=30, employee_id='E123', department='General')

# Порядок полей: сначала поля родителя, затем дочернего
print(Employee.__dataclass_fields__.keys())
# dict_keys(['name', 'age', 'employee_id', 'department'])
```

**Проблема переопределения полей**

```python
@dataclass
class Base:
    value: int = 10

@dataclass
class Derived(Base):
    value: int = 20  # Переопределение поля — работает, но осторожно

d = Derived()
print(d)  # Derived(value=20)
```

**Замена стандартных методов**

Датакласс не мешает переопределить любой метод вручную.

```python
@dataclass
class Book:
    title: str
    author: str
    pages: int
    
    def __str__(self):
        return f"'{self.title}' by {self.author}"
    
    def __post_init__(self):
        if self.pages <= 0:
            raise ValueError("Pages must be positive")

b = Book("1984", "Orwell", 328)
print(str(b))  # '1984' by Orwell
```

**Датаклассы и сериализация**

```python
from dataclasses import dataclass, asdict, astuple
import json

@dataclass
class Address:
    city: str
    street: str
    zipcode: str

@dataclass
class Contact:
    name: str
    email: str
    address: Address

c = Contact("Alice", "alice@example.com", Address("NYC", "5th Ave", "10001"))

# Преобразование в словарь
data_dict = asdict(c)
print(data_dict)  # {'name': 'Alice', 'email': '...', 'address': {'city': 'NYC', ...}}

# В JSON
json_str = json.dumps(asdict(c), indent=2)

# Восстановление
restored = Contact(**json.loads(json_str))
```

**Фабрики значений с `default_factory`**

```python
from dataclasses import dataclass, field
from datetime import datetime
import uuid

def generate_id() -> str:
    return str(uuid.uuid4())[:8]

def default_timestamp() -> float:
    return datetime.now().timestamp()

@dataclass
class Document:
    content: str
    doc_id: str = field(default_factory=generate_id)
    created: float = field(default_factory=default_timestamp)
    tags: set = field(default_factory=set)

d1 = Document("Hello")
d2 = Document("World")
print(d1.doc_id)  # "a1b2c3d4"
print(d2.doc_id)  # "e5f6g7h8" — разные
```

**Параметр `frozen=True` для неизменяемых объектов**

```python
@dataclass(frozen=True)
class ImmutablePoint:
    x: int
    y: int

p = ImmutablePoint(1, 2)
# p.x = 10  # FrozenInstanceError

# Для изменяемых полей — осторожно
@dataclass(frozen=True)
class BadFrozen:
    items: list

bf = BadFrozen([1, 2])
bf.items.append(3)  # Работает! Frozen только на атрибут, не на содержимое
print(bf.items)     # [1, 2, 3] — мутация возможна
```

**Параметр `order=True` для сортировки**

```python
@dataclass(order=True)
class Student:
    grade: float
    name: str = field(compare=False)  # Исключить из сравнения

s1 = Student(85.5, "Alice")
s2 = Student(92.0, "Bob")
s3 = Student(85.5, "Charlie")

print(s1 < s2)   # True (85.5 < 92.0)
print(s1 == s3)  # True (85.5 == 85.5 — name исключён)
```

**Атрибуты только для сравнения (`compare=False`)**

```python
@dataclass(eq=True, order=True)
class CacheEntry:
    priority: int
    created_at: float = field(compare=False)      # Игнорировать при сравнении
    data: str = field(repr=False, compare=False)  # Игнорировать везде

e1 = CacheEntry(1, 100, "secret")
e2 = CacheEntry(1, 200, "different")
print(e1 == e2)   # True — только priority сравнивается
```

**Практический пример: конфигурация приложения**

```python
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class DatabaseConfig:
    host: str = "localhost"
    port: int = 5432
    user: str = "admin"
    password: str = field(repr=False)  # Не показывать в repr
    
    @property
    def connection_string(self) -> str:
        return f"postgresql://{self.user}:{self.password}@{self.host}:{self.port}"

@dataclass
class LoggingConfig:
    level: str = "INFO"
    file: Optional[str] = None

@dataclass
class AppConfig:
    name: str
    version: str
    database: DatabaseConfig = field(default_factory=DatabaseConfig)
    logging: LoggingConfig = field(default_factory=LoggingConfig)
    
    def __post_init__(self):
        if self.version.startswith("0."):
            print("Warning: Development version")

config = AppConfig("MyApp", "1.0.0")
print(config.database.connection_string)
```

**Датакласс vs NamedTuple vs обычный класс**

| Характеристика | `@dataclass` | `NamedTuple` | Обычный класс |
|----------------|--------------|--------------|---------------|
| Изменяемость | Да (или `frozen`) | Нет | Да |
| Шаблонный код | Нет | Нет | Много |
| Аннотации типов | Да | Да | Вручную |
| Наследование | Да | Ограничено | Да |
| Производительность | Средняя | Высокая | Высокая |
| Методы | Можно добавлять | Ограничено | Всё что угодно |
| Применение | DTO, модели | Лёгкие неизменяемые структуры | Сложная логика |

**Антипаттерны датаклассов**

❌ **Слишком много полей:**
```python
@dataclass
class GodObject:
    field1: int
    field2: int
    # ... 50 полей
```
Решение: разделить на меньшие датаклассы.

❌ **Сложная логика в `__post_init__`:**
```python
def __post_init__(self):
    self.data = self.fetch_from_db()  # Не делайте так
    self.validate_complex_rules()
    self.send_notification()
```
Решение: выносите в отдельные методы.

❌ **Изменяемые поля с `frozen=True`:**
```python
@dataclass(frozen=True)
class MutableInside:
    items: list  # Опасно — список изменить можно
```

**Итоговые правила использования датаклассов:**

1. **Используйте для простых структур данных (DTO, Value Objects).**

2. **Всегда указывайте типы полей** — иначе они не попадут в `__init__`.

3. **`default_factory` для изменяемых типов** — списки, словари, множества.

4. **`__post_init__` для валидации и пост-обработки.**

5. **`frozen=True` для неизменяемых объектов** — хорошая практика для кэшей и ключей.

6. **Не перегружайте датакласс бизнес-логикой** — оставьте поведение в сервисах или отдельных методах.

**Ключевая мысль:** датаклассы — это не замена классам, а инструмент для быстрого создания классов-контейнеров. Они сокращают код с 70% шаблона до 5%, оставляя вам пространство для настоящей бизнес-логики. Используйте их везде, где класс служит только для хранения данных. Там, где нужна сложная логика, пишите обычный класс — датакласс не обязан быть единственным подходом.



**20. Продвинутые паттерны ООП и лучшие практики**

**Паттерн "Одиночка" (Singleton)**

Гарантирует существование только одного экземпляра класса.

```python
class Singleton:
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self, value=None):
        if not hasattr(self, 'initialized'):
            self.value = value
            self.initialized = True

s1 = Singleton(10)
s2 = Singleton(20)
print(s1 is s2)      # True
print(s1.value)      # 10
print(s2.value)      # 10 — второй вызов __init__ не изменил

# Более надёжный вариант с метаклассом
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Config(metaclass=SingletonMeta):
    def __init__(self, settings=None):
        self.settings = settings or {}
```

**Паттерн "Фабричный метод" (Factory Method)**

Делегирует создание объектов подклассам.

```python
from abc import ABC, abstractmethod

class Transport(ABC):
    @abstractmethod
    def deliver(self): pass

class Truck(Transport):
    def deliver(self):
        return "Delivering by truck (land)"

class Ship(Transport):
    def deliver(self):
        return "Delivering by ship (sea)"

class Logistics(ABC):
    @abstractmethod
    def create_transport(self) -> Transport:
        pass
    
    def plan_delivery(self):
        transport = self.create_transport()
        return transport.deliver()

class RoadLogistics(Logistics):
    def create_transport(self):
        return Truck()

class SeaLogistics(Logistics):
    def create_transport(self):
        return Ship()

logistics = RoadLogistics()
print(logistics.plan_delivery())  # Delivering by truck (land)
```

**Паттерн "Строитель" (Builder)**

Пошаговое создание сложных объектов.

```python
class Computer:
    def __init__(self):
        self.cpu = None
        self.ram = None
        self.storage = None
        self.gpu = None
    
    def __str__(self):
        return f"CPU: {self.cpu}, RAM: {self.ram}, Storage: {self.storage}, GPU: {self.gpu}"

class ComputerBuilder:
    def __init__(self):
        self.computer = Computer()
    
    def set_cpu(self, cpu):
        self.computer.cpu = cpu
        return self
    
    def set_ram(self, ram):
        self.computer.ram = ram
        return self
    
    def set_storage(self, storage):
        self.computer.storage = storage
        return self
    
    def set_gpu(self, gpu):
        self.computer.gpu = gpu
        return self
    
    def build(self):
        return self.computer

# Fluent interface
gaming_pc = (ComputerBuilder()
             .set_cpu("Intel i9")
             .set_ram("32GB")
             .set_storage("1TB SSD")
             .set_gpu("RTX 4080")
             .build())
```

**Паттерн "Наблюдатель" (Observer)**

Уведомление зависимых объектов об изменениях.

```python
from abc import ABC, abstractmethod

class Observer(ABC):
    @abstractmethod
    def update(self, subject, event): pass

class Subject:
    def __init__(self):
        self._observers = []
    
    def attach(self, observer: Observer):
        self._observers.append(observer)
    
    def detach(self, observer: Observer):
        self._observers.remove(observer)
    
    def notify(self, event):
        for observer in self._observers:
            observer.update(self, event)

class User(Subject):
    def __init__(self, name):
        super().__init__()
        self.name = name
        self._balance = 0
    
    @property
    def balance(self):
        return self._balance
    
    @balance.setter
    def balance(self, value):
        old = self._balance
        self._balance = value
        self.notify(("balance_changed", old, value))

class EmailNotifier(Observer):
    def update(self, subject, event):
        if event[0] == "balance_changed":
            print(f"Email to {subject.name}: Balance changed {event[1]} → {event[2]}")

class Logger(Observer):
    def update(self, subject, event):
        print(f"[LOG] {subject.name}: {event}")

user = User("Alice")
user.attach(EmailNotifier())
user.attach(Logger())
user.balance = 1000
# Email to Alice: Balance changed 0 → 1000
# [LOG] Alice: ('balance_changed', 0, 1000)
```

**Паттерн "Стратегия" (Strategy)**

Инкапсулирует алгоритм, делая его взаимозаменяемым.

```python
from abc import ABC, abstractmethod

class SortingStrategy(ABC):
    @abstractmethod
    def sort(self, data): pass

class BubbleSort(SortingStrategy):
    def sort(self, data):
        arr = data.copy()
        n = len(arr)
        for i in range(n):
            for j in range(0, n-i-1):
                if arr[j] > arr[j+1]:
                    arr[j], arr[j+1] = arr[j+1], arr[j]
        return arr

class QuickSort(SortingStrategy):
    def sort(self, data):
        if len(data) <= 1:
            return data
        pivot = data[0]
        left = [x for x in data[1:] if x <= pivot]
        right = [x for x in data[1:] if x > pivot]
        return self.sort(left) + [pivot] + self.sort(right)

class Sorter:
    def __init__(self, strategy: SortingStrategy):
        self._strategy = strategy
    
    def set_strategy(self, strategy: SortingStrategy):
        self._strategy = strategy
    
    def sort(self, data):
        return self._strategy.sort(data)

data = [3, 1, 4, 1, 5, 9, 2, 6]
sorter = Sorter(BubbleSort())
print(sorter.sort(data))  # [1, 1, 2, 3, 4, 5, 6, 9]
sorter.set_strategy(QuickSort())
print(sorter.sort(data))  # [1, 1, 2, 3, 4, 5, 6, 9]
```

**Паттерн "Декоратор" (Decorator)**

Динамическое добавление поведения без изменения исходного класса.

```python
from abc import ABC, abstractmethod

class Coffee(ABC):
    @abstractmethod
    def cost(self):
        pass
    
    @abstractmethod
    def description(self):
        pass

class SimpleCoffee(Coffee):
    def cost(self):
        return 2.0
    
    def description(self):
        return "Simple coffee"

class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        self._coffee = coffee
    
    def cost(self):
        return self._coffee.cost()
    
    def description(self):
        return self._coffee.description()

class MilkDecorator(CoffeeDecorator):
    def cost(self):
        return self._coffee.cost() + 0.5
    
    def description(self):
        return self._coffee.description() + ", milk"

class SugarDecorator(CoffeeDecorator):
    def cost(self):
        return self._coffee.cost() + 0.2
    
    def description(self):
        return self._coffee.description() + ", sugar"

coffee = SimpleCoffee()
coffee = MilkDecorator(coffee)
coffee = SugarDecorator(coffee)
print(coffee.description())  # Simple coffee, milk, sugar
print(coffee.cost())         # 2.7
```

**Паттерн "Компоновщик" (Composite)**

Древовидная структура для работы с отдельными объектами и их группами.

```python
from abc import ABC, abstractmethod

class FileSystemComponent(ABC):
    @abstractmethod
    def get_size(self):
        pass

class File(FileSystemComponent):
    def __init__(self, name, size):
        self.name = name
        self._size = size
    
    def get_size(self):
        return self._size

class Directory(FileSystemComponent):
    def __init__(self, name):
        self.name = name
        self._children = []
    
    def add(self, component: FileSystemComponent):
        self._children.append(component)
    
    def remove(self, component: FileSystemComponent):
        self._children.remove(component)
    
    def get_size(self):
        return sum(child.get_size() for child in self._children)

root = Directory("root")
docs = Directory("docs")
pics = Directory("pics")

docs.add(File("resume.pdf", 1024))
docs.add(File("cover.pdf", 512))
pics.add(File("photo.jpg", 2048))
root.add(docs)
root.add(pics)

print(root.get_size())  # 3584
```

**Лучшие практики ООП в Python**

**1. Следуйте принципам SOLID**

```python
# S: Single Responsibility
# Плохо
class User:
    def save_to_db(self): pass
    def send_email(self): pass
    def calculate_tax(self): pass

# Хорошо
class User: pass
class UserRepository: pass
class EmailService: pass
class TaxCalculator: pass

# O: Open/Closed (открыт для расширения, закрыт для изменения)
# Плохо
def calculate_area(shapes):
    if isinstance(shape, Circle): ...
    elif isinstance(shape, Rectangle): ...

# Хорошо
class Shape:
    def area(self): pass
class Circle(Shape):
    def area(self): return 3.14 * self.radius ** 2

# L: Liskov Substitution
class Bird: pass
class FlyingBird(Bird):
    def fly(self): pass
class Penguin(Bird): pass  # Не наследует FlyingBird

# I: Interface Segregation
# Плохо
class Worker(ABC):
    @abstractmethod
    def work(self): pass
    @abstractmethod
    def eat(self): pass

# Хорошо
class Workable(ABC):
    @abstractmethod
    def work(self): pass
class Eatable(ABC):
    @abstractmethod
    def eat(self): pass

# D: Dependency Inversion
# Плохо
class EmailSender:
    def send(self): pass
class Notification:
    def __init__(self):
        self.sender = EmailSender()  # Жёсткая связь

# Хорошо
class MessageSender(ABC):
    @abstractmethod
    def send(self): pass
class Notification:
    def __init__(self, sender: MessageSender):
        self.sender = sender
```

**2. Используйте композицию вместо наследования (повторение с примером)**

```python
# Плохо: наследование для переиспользования
class Loggable:
    def log(self, msg): print(msg)

class User(Loggable): pass  # User — не Loggable

# Хорошо: композиция + миксины
class LoggerMixin:
    def log(self, msg): print(f"[{self.__class__.__name__}] {msg}")

class User(LoggerMixin):  # Миксин для поведения
    pass
```

**3. Избегайте глубокой иерархии наследования**

```python
# Плохо: 5+ уровней
class Entity: pass
class Animal(Entity): pass
class Mammal(Animal): pass
class Primate(Mammal): pass
class Human(Primate): pass
class Employee(Human): pass

# Хорошо: плоская иерархия + композиция
class Person:
    def __init__(self, role: 'Role'):
        self.role = role
```

**4. Используйте `@property` для контролируемого доступа**

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius
    
    @property
    def celsius(self):
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Below absolute zero")
        self._celsius = value
    
    @property
    def fahrenheit(self):
        return self.celsius * 9/5 + 32
```

**5. Применяйте фабрики для сложного создания объектов**

```python
class PaymentMethod:
    pass

class CreditCard(PaymentMethod): pass
class PayPal(PaymentMethod): pass

class PaymentFactory:
    _methods = {
        'credit': CreditCard,
        'paypal': PayPal,
    }
    
    @classmethod
    def create(cls, method_type, **kwargs):
        method_class = cls._methods.get(method_type)
        if not method_class:
            raise ValueError(f"Unknown method: {method_type}")
        return method_class(**kwargs)

payment = PaymentFactory.create('credit', card_number='4111...')
```

**6. Используйте слабые ссылки (`weakref`) для кэшей и наблюдателей**

```python
import weakref

class EventBus:
    def __init__(self):
        self._listeners = weakref.WeakSet()  # Не阻止 сборку мусора
    
    def subscribe(self, listener):
        self._listeners.add(listener)
    
    def emit(self, event):
        for listener in self._listeners:
            listener(event)
```

**7. Переопределяйте `__repr__` всегда, `__str__` когда нужно**

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
    
    def __repr__(self):
        return f"Product('{self.name}', {self.price})"
    
    def __str__(self):
        return f"{self.name} — ${self.price:.2f}"
```

**8. Используйте `__slots__` для экономии памяти при множестве экземпляров**

```python
class Point:
    __slots__ = ('x', 'y')  # Запрещает динамические атрибуты, экономит память
    
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Для тысяч экземпляров экономия значительна (~50% памяти)
```

**9. Применяйте абстрактные базовые классы для интерфейсов**

```python
from collections.abc import Iterable

class MyIterable(Iterable):
    def __iter__(self):
        yield from [1, 2, 3]

# Проверка протокола
print(isinstance(MyIterable(), Iterable))  # True
```

**10. Документируйте публичное API класса**

```python
class BankAccount:
    """
    Банковский счёт с поддержкой депозитов и снятий.
    
    Attributes:
        owner: Владелец счёта
        balance: Текущий баланс (только чтение)
    
    Example:
        >>> acc = BankAccount("Alice", 1000)
        >>> acc.deposit(500)
        >>> acc.withdraw(200)
        >>> acc.balance
        1300
    """
    
    def __init__(self, owner: str, initial_balance: float = 0):
        self.owner = owner
        self._balance = initial_balance
    
    @property
    def balance(self) -> float:
        """Текущий баланс счёта (только чтение)."""
        return self._balance
```

**Итоговая таблица выбора паттернов**

| Сценарий | Паттерн |
|----------|---------|
| Нужен один экземпляр | Singleton |
| Создание семейства объектов | Factory Method / Abstract Factory |
| Пошаговое создание сложного объекта | Builder |
| Уведомление об изменениях | Observer |
| Взаимозаменяемые алгоритмы | Strategy |
| Динамическое добавление поведения | Decorator |
| Древовидные структуры | Composite |
| Единый интерфейс для разных API | Adapter |
| Оптимизация ресурсов | Flyweight |
| Цепочка обработчиков | Chain of Responsibility |

**Ключевые принципы, которые стоит запомнить:**

1. **KISS (Keep It Simple, Stupid)** — не усложняйте без необходимости.
2. **YAGNI (You Ain't Gonna Need It)** — не добавляйте функциональность "на всякий случай".
3. **DRY (Don't Repeat Yourself)** — извлекайте повторяющийся код.
4. **Слабые связи, сильная связность внутри модуля.**
5. **Тестируемость — признак хорошего дизайна.**
6. **Инкапсуляция в Python — на уровне соглашений, не синтаксиса.**
7. **Полиморфизм через утиную типизацию мощнее, чем через наследование.**

Продвинутое ООП в Python — не о том, чтобы использовать все паттерны сразу, а о том, чтобы выбрать правильный инструмент для конкретной задачи. Начинайте с простого, рефакторите когда появляется боль. Паттерны — не цель, а средство управления сложностью.
