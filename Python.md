Основные и продвинутые Python функции для каждого уровня

🟢 JUNIOR DEVELOPER (Начинающий)

1. Базовые операции со строками

```python
def string_operations(text: str) -> dict:
    """Все базовые операции со строками"""
    return {
        'upper': text.upper(),
        'lower': text.lower(),
        'title': text.title(),
        'strip': text.strip(),
        'split': text.split(),
        'join': '-'.join(['a', 'b', 'c']),
        'replace': text.replace('old', 'new'),
        'find': text.find('substring'),
        'count': text.count('a'),
        'startswith': text.startswith('hello'),
        'endswith': text.endswith('world'),
        'isalpha': text.isalpha(),
        'isdigit': text.isdigit(),
        'format': 'Hello, {}!'.format('World'),
        'f-string': f'Length: {len(text)}',
    }
```

2. Работа со списками и словарями

```python
def list_dict_operations():
    """Операции со списками и словарями"""
    # Списки
    my_list = [1, 2, 3, 4, 5]
    
    # Добавление/удаление
    my_list.append(6)
    my_list.extend([7, 8])
    my_list.insert(0, 0)
    my_list.remove(3)
    popped = my_list.pop()
    
    # Срезы
    sliced = my_list[1:4]
    reversed_list = my_list[::-1]
    
    # Сортировка
    sorted_list = sorted(my_list)
    my_list.sort(reverse=True)
    
    # Словари
    my_dict = {'a': 1, 'b': 2, 'c': 3}
    
    # Основные операции
    keys = my_dict.keys()
    values = my_dict.values()
    items = my_dict.items()
    
    # Получение значений
    value_a = my_dict.get('a', 'default')
    value_d = my_dict.get('d', 'not found')
    
    return {
        'list': my_list,
        'sliced': sliced,
        'reversed': reversed_list,
        'sorted': sorted_list,
        'dict_keys': list(keys),
        'dict_values': list(values),
        'get_value': value_a,
        'get_default': value_d,
    }
```

3. Функции с аргументами

```python
def function_with_args(*args, **kwargs):
    """Функция с переменным числом аргументов"""
    print(f"Positional args: {args}")
    print(f"Keyword args: {kwargs}")
    return sum(args)

# Использование
result = function_with_args(1, 2, 3, name='John', age=25)
```

4. Работа с файлами

```python
def file_operations():
    """Чтение и запись файлов"""
    # Запись
    with open('file.txt', 'w', encoding='utf-8') as f:
        f.write('Hello, World!\n')
        f.write('Second line\n')
    
    # Чтение всего файла
    with open('file.txt', 'r', encoding='utf-8') as f:
        content = f.read()
    
    # Построчное чтение
    with open('file.txt', 'r', encoding='utf-8') as f:
        lines = f.readlines()
    
    # Добавление в файл
    with open('file.txt', 'a', encoding='utf-8') as f:
        f.write('Appended line\n')
    
    return {'content': content, 'lines': lines}
```

5. Обработка исключений

```python
def safe_division(a: float, b: float) -> float:
    """Безопасное деление с обработкой исключений"""
    try:
        result = a / b
    except ZeroDivisionError:
        print("Ошибка: деление на ноль!")
        return 0
    except TypeError as e:
        print(f"Ошибка типа: {e}")
        return 0
    except Exception as e:
        print(f"Неизвестная ошибка: {e}")
        return 0
    else:
        print("Деление выполнено успешно")
        return result
    finally:
        print("Блок finally выполняется всегда")
```

🟡 MIDDLE DEVELOPER (Средний уровень)

1. Декораторы

```python
from functools import wraps
import time
from typing import Callable, Any

def timer_decorator(func: Callable) -> Callable:
    """Декоратор для измерения времени выполнения"""
    @wraps(func)
    def wrapper(*args, **kwargs) -> Any:
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"{func.__name__} выполнилась за {end_time - start_time:.4f} секунд")
        return result
    return wrapper

def cache_decorator(func: Callable) -> Callable:
    """Декоратор для кэширования результатов"""
    cache = {}
    
    @wraps(func)
    def wrapper(*args, **kwargs) -> Any:
        key = str(args) + str(kwargs)
        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]
    return wrapper

# Использование
@timer_decorator
@cache_decorator
def fibonacci(n: int) -> int:
    """Вычисление числа Фибоначчи"""
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

2. Генераторы

```python
def fibonacci_generator(n: int):
    """Генератор чисел Фибоначчи"""
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

def read_large_file(file_path: str):
    """Генератор для чтения больших файлов"""
    with open(file_path, 'r', encoding='utf-8') as file:
        for line in file:
            yield line.strip()

# Использование
for num in fibonacci_generator(10):
    print(num)

for line in read_large_file('large_file.txt'):
    process_line(line)
```

3. Контекстные менеджеры

```python
from contextlib import contextmanager
import sqlite3
import threading

@contextmanager
def database_connection(db_path: str):
    """Контекстный менеджер для работы с БД"""
    connection = sqlite3.connect(db_path)
    try:
        yield connection
        connection.commit()
    except Exception as e:
        connection.rollback()
        raise e
    finally:
        connection.close()

@contextmanager
def thread_lock(lock: threading.Lock):
    """Контекстный менеджер для блокировки потоков"""
    lock.acquire()
    try:
        yield
    finally:
        lock.release()

# Использование
with database_connection('my_database.db') as conn:
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM users')
```

4. ООП: классы и магические методы

```python
class Vector:
    """Класс для работы с векторами"""
    
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y
    
    def __add__(self, other: 'Vector') -> 'Vector':
        return Vector(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other: 'Vector') -> 'Vector':
        return Vector(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar: float) -> 'Vector':
        return Vector(self.x * scalar, self.y * scalar)
    
    def __str__(self) -> str:
        return f'Vector({self.x}, {self.y})'
    
    def __repr__(self) -> str:
        return f'Vector({self.x}, {self.y})'
    
    def __len__(self) -> int:
        return 2
    
    def __getitem__(self, index: int) -> float:
        if index == 0:
            return self.x
        elif index == 1:
            return self.y
        raise IndexError("Index out of range")
    
    @property
    def magnitude(self) -> float:
        """Длина вектора"""
        return (self.x ** 2 + self.y ** 2) ** 0.5
    
    @classmethod
    def from_tuple(cls, coordinates: tuple) -> 'Vector':
        """Альтернативный конструктор"""
        return cls(coordinates[0], coordinates[1])
    
    @staticmethod
    def dot_product(v1: 'Vector', v2: 'Vector') -> float:
        """Скалярное произведение"""
        return v1.x * v2.x + v1.y * v2.y
```

5. Асинхронные функции

```python
import asyncio
import aiohttp

async def fetch_url(url: str) -> str:
    """Асинхронное получение URL"""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def fetch_multiple_urls(urls: list) -> list:
    """Параллельное получение нескольких URL"""
    tasks = [fetch_url(url) for url in urls]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    return results

async def process_with_semaphore(urls: list, limit: int = 5):
    """Ограничение количества одновременных запросов"""
    semaphore = asyncio.Semaphore(limit)
    
    async def fetch_with_semaphore(url: str):
        async with semaphore:
            return await fetch_url(url)
    
    tasks = [fetch_with_semaphore(url) for url in urls]
    return await asyncio.gather(*tasks)
```

6. Работа с датами и временем

```python
from datetime import datetime, timedelta, timezone
from dateutil import parser
import pytz

def datetime_operations():
    """Продвинутая работа с датами"""
    # Текущее время
    now = datetime.now()
    now_utc = datetime.now(timezone.utc)
    
    # Форматирование
    formatted = now.strftime("%Y-%m-%d %H:%M:%S")
    
    # Парсинг
    parsed = parser.parse("2024-01-15 14:30:00")
    
    # Разница во времени
    delta = timedelta(days=7, hours=3)
    future = now + delta
    
    # Часовые пояса
    moscow_tz = pytz.timezone('Europe/Moscow')
    moscow_time = now.astimezone(moscow_tz)
    
    # Unix timestamp
    timestamp = now.timestamp()
    from_timestamp = datetime.fromtimestamp(timestamp)
    
    return {
        'now': now,
        'formatted': formatted,
        'parsed': parsed,
        'future': future,
        'moscow_time': moscow_time,
        'timestamp': timestamp,
    }
```

🔴 SENIOR DEVELOPER (Продвинутый уровень)

1. Метапрограммирование: метаклассы

```python
class SingletonMeta(type):
    """Метакласс для реализации Singleton"""
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class RegistryMeta(type):
    """Метакласс для автоматической регистрации классов"""
    registry = {}
    
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if name not in ['Base']:
            mcs.registry[name] = cls
        return cls

# Использование
class Database(metaclass=SingletonMeta):
    def __init__(self):
        print("Создано подключение к БД")

class Base(metaclass=RegistryMeta):
    pass

class User(Base):
    pass

class Product(Base):
    pass

print(RegistryMeta.registry)  # {'User': <class '__main__.User'>, ...}
```

2. Дескрипторы

```python
class ValidatedAttribute:
    """Дескриптор с валидацией"""
    
    def __init__(self, min_value=None, max_value=None):
        self.min_value = min_value
        self.max_value = max_value
        self.data = {}
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.data.get(id(instance))
    
    def __set__(self, instance, value):
        if self.min_value is not None and value < self.min_value:
            raise ValueError(f"Value must be >= {self.min_value}")
        if self.max_value is not None and value > self.max_value:
            raise ValueError(f"Value must be <= {self.max_value}")
        self.data[id(instance)] = value
    
    def __delete__(self, instance):
        del self.data[id(instance)]

class Product:
    price = ValidatedAttribute(min_value=0)
    quantity = ValidatedAttribute(min_value=0, max_value=1000)
    
    def __init__(self, price, quantity):
        self.price = price
        self.quantity = quantity
```

3. Асинхронные контекстные менеджеры

```python
import asyncio
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_database_pool(dsn: str, pool_size: int = 10):
    """Асинхронный контекстный менеджер для пула соединений"""
    pool = await create_connection_pool(dsn, pool_size)
    try:
        yield pool
    finally:
        await pool.close()

@asynccontextmanager
async def rate_limiter(rate: int, per: float):
    """Ограничение частоты запросов"""
    semaphore = asyncio.Semaphore(rate)
    delay = per / rate
    
    async def limited_coro(coro):
        async with semaphore:
            await asyncio.sleep(delay)
            return await coro
    
    yield limited_coro
```

4. Продвинутые декораторы с параметрами

```python
def retry(max_attempts: int = 3, delay: float = 1.0, 
          exceptions: tuple = (Exception,)):
    """Декоратор для повторных попыток с экспоненциальной задержкой"""
    def decorator(func):
        @wraps(func)
        async def async_wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts - 1:
                        raise e
                    wait = delay * (2 ** attempt)
                    print(f"Attempt {attempt + 1} failed. Retrying in {wait}s")
                    await asyncio.sleep(wait)
        
        @wraps(func)
        def sync_wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts - 1:
                        raise e
                    wait = delay * (2 ** attempt)
                    print(f"Attempt {attempt + 1} failed. Retrying in {wait}s")
                    time.sleep(wait)
        
        return async_wrapper if asyncio.iscoroutinefunction(func) else sync_wrapper
    return decorator
```

5. Пул потоков/процессов

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed
from functools import partial
import multiprocessing

def parallel_processing(data: list, func: callable, 
                       max_workers: int = None, 
                       use_processes: bool = False):
    """
    Параллельная обработка данных
    
    Args:
        data: список данных для обработки
        func: функция обработки
        max_workers: максимальное число воркеров
        use_processes: использовать процессы вместо потоков
    """
    Executor = ProcessPoolExecutor if use_processes else ThreadPoolExecutor
    
    if max_workers is None:
        max_workers = multiprocessing.cpu_count()
    
    with Executor(max_workers=max_workers) as executor:
        # Отправляем задачи на выполнение
        futures = [executor.submit(func, item) for item in data]
        
        # Собираем результаты по мере готовности
        results = []
        for future in as_completed(futures):
            try:
                result = future.result()
                results.append(result)
            except Exception as e:
                print(f"Error processing: {e}")
        
        return results

# Оптимизированная версия с chunking
def parallel_map(func: callable, iterable: list, 
                chunk_size: int = 100,
                max_workers: int = None) -> list:
    """
    Параллельный map с чанками для лучшей производительности
    """
    from itertools import islice
    
    def process_chunk(chunk):
        return [func(item) for item in chunk]
    
    # Разбиваем на чанки
    chunks = []
    it = iter(iterable)
    while True:
        chunk = list(islice(it, chunk_size))
        if not chunk:
            break
        chunks.append(chunk)
    
    # Обрабатываем чанки параллельно
    results = parallel_processing(chunks, process_chunk, max_workers, True)
    
    # Объединяем результаты
    return [item for sublist in results for item in sublist]
```

6. Кэширование и мемоизация

```python
from functools import lru_cache
import hashlib
import pickle
from typing import Any

class PersistentCache:
    """Постоянное кэширование на диске"""
    
    def __init__(self, cache_dir: str = '.cache', max_size: int = 1000):
        self.cache_dir = Path(cache_dir)
        self.cache_dir.mkdir(exist_ok=True)
        self.max_size = max_size
    
    def _get_key(self, func_name: str, *args, **kwargs) -> str:
        """Генерация ключа кэша"""
        data = pickle.dumps((func_name, args, kwargs))
        return hashlib.md5(data).hexdigest()
    
    def _get_cache_path(self, key: str) -> Path:
        return self.cache_dir / f"{key}.pkl"
    
    def get(self, func_name: str, *args, **kwargs) -> Any:
        """Получение из кэша"""
        key = self._get_key(func_name, *args, **kwargs)
        cache_path = self._get_cache_path(key)
        
        if cache_path.exists():
            with open(cache_path, 'rb') as f:
                return pickle.load(f)
        return None
    
    def set(self, func_name: str, value: Any, *args, **kwargs):
        """Сохранение в кэш"""
        key = self._get_key(func_name, *args, **kwargs)
        cache_path = self._get_cache_path(key)
        
        with open(cache_path, 'wb') as f:
            pickle.dump(value, f)
        
        # Очистка старых файлов
        self._cleanup()

def cached(func):
    """Декоратор с постоянным кэшированием"""
    cache = PersistentCache()
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        func_name = f"{func.__module__}.{func.__name__}"
        cached_value = cache.get(func_name, *args, **kwargs)
        
        if cached_value is not None:
            return cached_value
        
        result = func(*args, **kwargs)
        cache.set(func_name, result, *args, **kwargs)
        return result
    
    return wrapper
```

7. Динамическое создание классов и функций

```python
def create_class(class_name: str, base_classes: tuple, 
                attributes: dict, methods: dict):
    """Динамическое создание класса"""
    
    class_dict = {}
    class_dict.update(attributes)
    
    for method_name, method_func in methods.items():
        class_dict[method_name] = method_func
    
    return type(class_name, base_classes, class_dict)

def create_dynamic_function(code_string: str, 
                           function_name: str = "dynamic_func",
                           imports: dict = None):
    """Динамическое создание функции из строки кода"""
    
    namespace = {}
    
    # Добавляем импорты
    if imports:
        namespace.update(imports)
    
    # Выполняем код
    exec(code_string, namespace)
    
    # Получаем функцию
    return namespace.get(function_name)

# Пример использования
DynamicClass = create_class(
    "DynamicClass",
    (object,),
    {"attribute": 42},
    {"method": lambda self: f"Value: {self.attribute}"}
)

obj = DynamicClass()
print(obj.method())  # Value: 42
```

8. Профилирование и оптимизация

```python
import cProfile
import pstats
import io
from memory_profiler import profile
import tracemalloc
from line_profiler import LineProfiler

def profile_performance(func):
    """Декоратор для профилирования производительности"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        # Профилирование CPU
        pr = cProfile.Profile()
        pr.enable()
        
        result = func(*args, **kwargs)
        
        pr.disable()
        
        # Вывод результатов
        s = io.StringIO()
        ps = pstats.Stats(pr, stream=s).sort_stats('cumulative')
        ps.print_stats(20)
        print(s.getvalue())
        
        return result
    return wrapper

def profile_memory(func):
    """Декоратор для профилирования памяти"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        tracemalloc.start()
        
        result = func(*args, **kwargs)
        
        snapshot = tracemalloc.take_snapshot()
        top_stats = snapshot.statistics('lineno')
        
        print("[ Top 10 memory usage ]")
        for stat in top_stats[:10]:
            print(stat)
        
        tracemalloc.stop()
        return result
    return wrapper

def optimize_with_lru():
    """Пример оптимизации с LRU кэшем"""
    
    @lru_cache(maxsize=128)
    def expensive_calculation(n: int) -> int:
        # Медленная операция
        return sum(i * i for i in range(n))
    
    return expensive_calculation
```

9. Создание DSL (Domain Specific Language)

```python
class QueryBuilder:
    """Построитель SQL-запросов как DSL"""
    
    def __init__(self):
        self._select = []
        self._from = None
        self._where = []
        self._group_by = []
        self._having = []
        self._order_by = []
    
    def select(self, *columns):
        self._select.extend(columns)
        return self
    
    def from_(self, table):
        self._from = table
        return self
    
    def where(self, condition):
        self._where.append(condition)
        return self
    
    def group_by(self, *columns):
        self._group_by.extend(columns)
        return self
    
    def having(self, condition):
        self._having.append(condition)
        return self
    
    def order_by(self, *columns):
        self._order_by.extend(columns)
        return self
    
    def build(self) -> str:
        parts = []
        parts.append(f"SELECT {', '.join(self._select)}")
        parts.append(f"FROM {self._from}")
        
        if self._where:
            parts.append(f"WHERE {' AND '.join(self._where)}")
        
        if self._group_by:
            parts.append(f"GROUP BY {', '.join(self._group_by)}")
        
        if self._having:
            parts.append(f"HAVING {' AND '.join(self._having)}")
        
        if self._order_by:
            parts.append(f"ORDER BY {', '.join(self._order_by)}")
        
        return ' '.join(parts)

# Использование DSL
query = (QueryBuilder()
         .select('id', 'name', 'COUNT(*)')
         .from_('users')
         .where('age > 18')
         .where('status = "active"')
         .group_by('id', 'name')
         .order_by('name')
         .build())
```

10. Расширенные паттерны проектирования

```python
from abc import ABC, abstractmethod
from typing import Generic, TypeVar
from dataclasses import dataclass

T = TypeVar('T')

class Repository(Generic[T], ABC):
    """Generic Repository Pattern"""
    
    @abstractmethod
    def get(self, id: int) -> T:
        pass
    
    @abstractmethod
    def save(self, entity: T) -> T:
        pass
    
    @abstractmethod
    def delete(self, id: int) -> bool:
        pass

class UnitOfWork:
    """Unit of Work Pattern"""
    
    def __init__(self):
        self.new_objects = []
        self.dirty_objects = []
        self.removed_objects = []
    
    def register_new(self, obj):
        self.new_objects.append(obj)
    
    def register_dirty(self, obj):
        if obj not in self.new_objects and obj not in self.dirty_objects:
            self.dirty_objects.append(obj)
    
    def register_removed(self, obj):
        if obj in self.new_objects:
            self.new_objects.remove(obj)
        else:
            self.removed_objects.append(obj)
    
    def commit(self):
        # Сохранение всех изменений
        self._insert_new()
        self._update_dirty()
        self._delete_removed()
        
        self.new_objects.clear()
        self.dirty_objects.clear()
        self.removed_objects.clear()
    
    def rollback(self):
        self.new_objects.clear()
        self.dirty_objects.clear()
        self.removed_objects.clear()

class Specification(ABC):
    """Specification Pattern"""
    
    @abstractmethod
    def is_satisfied_by(self, item) -> bool:
        pass
    
    def __and__(self, other: 'Specification') -> 'Specification':
        return AndSpecification(self, other)
    
    def __or__(self, other: 'Specification') -> 'Specification':
        return OrSpecification(self, other)
    
    def __invert__(self) -> 'Specification':
        return NotSpecification(self)

class AndSpecification(Specification):
    def __init__(self, left: Specification, right: Specification):
        self.left = left
        self.right = right
    
    def is_satisfied_by(self, item) -> bool:
        return self.left.is_satisfied_by(item) and self.right.is_satisfied_by(item)
```

🎯 Чек-лист навыков по уровням

Junior должен знать:

· Базовый синтаксис Python
· Работа с основными типами данных
· Условные операторы и циклы
· Функции и модули
· Обработка исключений
· Работа с файлами
· Базовое ООП (классы, объекты)

Middle должен знать:

· Декораторы, генераторы, итераторы
· Контекстные менеджеры
· Продвинутое ООП (наследование, полиморфизм, магические методы)
· Многопоточность/многопроцессность
· Асинхронное программирование
· Работа с сетью (HTTP, сокеты)
· Тестирование (unittest, pytest)
· Работа с базами данных

Senior должен знать:

· Метапрограммирование (метаклассы, дескрипторы)
· Профилирование и оптимизация
· Паттерны проектирования
· Архитектурные паттерны
· Создание DSL
· Расширенные структуры данных
· Системное программирование
· Распределенные системы
· Безопасность и криптография
