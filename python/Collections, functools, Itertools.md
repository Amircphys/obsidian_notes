Специализированные контейнерные типы данных, предоставляющие альтернативы для встроенных  dict, list, set и tuple

1. **collections.namedtuple** - namedtuple(typename, field_names, *, rename=False, defaults=None, module=None)
```
>>> Point = collections.namedtuple("Point", ["x", "y"])
>>> p = Point(11, y=22)
# p = (11, 22)
>>> p[0] + p[1]
33
>>> x, y = p
>>> x, y
(11, 22)
>>> p.x + p.y
33
>>> p._asdict()
# {'x': 1, 'y': 4}
```
2. **collections.defaultdict** -  cловарь, у которого по умолчанию всегда вызывается функция default_factory
```
>>> import collections
>>> defdict = collections.defaultdict(list)
>>> print(defdict)
defaultdict(<class 'list'>, {})
>>> for i in range(5):
		defdict[i].append(i)
```
3. **collections.OrderedDict** - cловарь, который помнит порядок, в котором ему были даны ключи
4. **collections.Counter** - cловарь для подсчёта хешируемых объектов
```
	○ elements()
	○ most_common([n])
	○ subtract([iterable-or-mapping])
	○ update([iterable-or-mapping])
```
5. **collections.deque([iterable[, maxlen]])** - двусторонняя очередь из итерируемого объекта с максимальной длиной maxlen. Добавление и удаление элементов в начало или конец выполняется за константное время.
```
	○ append(x)                                >>> d = deque("ghi")
	○ appendleft(x)                            >>> d = deque("ghi")             
	○ extend(iterable)                         >>> d.append("j")   
	○ extendleft(iterable)                     >>> d.appendleft("f")
	○ insert(i, x)                             >>> d
	○ pop()/popleft()                          deque(['f', 'g', 'h', 'i', 'j'])
	○ remove(value)                            >>> d.pop()
                                               'j'
                                               >>> d.popleft()
                                               'f'
                                               >>> d
                                               deque(['g', 'h', 'i'])

```



---
## Itertools
```
○ accumulate(iterable[, func, *, initial=None])
○ chain(*iterables)
○ compress(data, selectors)
○ dropwhile(predicate, iterable)
○ takewhile(predicate, iterable)
○ filterfalse(predicate, iterable)
○ groupby(iterable, key=None)
○ islice(iterable, start, stop[, step])
○ pairwise(iterable)
○ starmap(function, iterable)
○ tee(iterable, n=2)
○ zip_longest(*iterables, fillvalue=None)
○ product(*iterables, repeat=1)
○ permutations(iterable, r=None)
○ combinations(iterable, r)
○ combinations_with_replacement(iterable, r)
```

### **1. `count` — Бесконечный счетчик**  
**Что делает:** Генерирует числа от стартового значения с заданным шагом.  
**Пример:**  
```python
from itertools import count

counter = count(10, 2)  # Начало: 10, шаг: 2
for i in range(3):
    print(next(counter))  # 10, 12, 14
```

---

### **2. `cycle` — Бесконечный цикл элементов**  
**Что делает:** Повторяет элементы итерируемого объекта бесконечно.  
**Пример:**  
```python
from itertools import cycle

cycler = cycle(["a", "b", "c"])
for _ in range(5):
    print(next(cycler))  # a, b, c, a, b
```

---

### **3. `repeat` — Повторение объекта**  
**Что делает:** Повторяет объект N раз или бесконечно.  
**Пример:**  
```python
from itertools import repeat

repeater = repeat("Python", 3)  # Повторить 3 раза
print(list(repeater))  # ['Python', 'Python', 'Python']
```

---

### **4. `accumulate` — Накопительные вычисления**  
**Что делает:** Возвращает последовательные результаты применения функции к элементам.  
**Пример:**  
```python
from itertools import accumulate

data = [1, 2, 3, 4]
result = accumulate(data, lambda a, b: a * b)  # Произведение
print(list(result))  # [1, 2, 6, 24]
```

---

### **5. `chain` — Объединение итерируемых объектов**  
**Что делает:** Склеивает несколько итерируемых объектов в один.  
**Пример:**  
```python
from itertools import chain

combined = chain([1, 2], "abc", (4, 5))
print(list(combined))  # [1, 2, 'a', 'b', 'c', 4, 5]
```

---

### **6. `compress` — Фильтрация по маске**  
**Что делает:** Оставляет элементы, соответствующие `True` в маске.  
**Пример:**  
```python
from itertools import compress

data = ["A", "B", "C", "D"]
mask = [1, 0, 1, 0]  # True/False можно передавать как 1/0
result = compress(data, mask)
print(list(result))  # ['A', 'C']
```

---

### **7. `dropwhile` — Пропуск элементов по условию**  
**Что делает:** Пропускает элементы, пока условие верно, затем возвращает все остальные.  
**Пример:**  
```python
from itertools import dropwhile

data = [1, 3, 5, 2, 4]
result = dropwhile(lambda x: x < 4, data)  # Пропустить числа <4
print(list(result))  # [5, 2, 4] (2 и 4 уже не проверяются!)
```

---

### **8. `takewhile` — Выбор элементов по условию**  
**Что делает:** Возвращает элементы, пока условие верно, затем останавливается.  
**Пример:**  
```python
from itertools import takewhile

data = [1, 3, 5, 2, 4]
result = takewhile(lambda x: x < 4, data)  # Брать, пока <4
print(list(result))  # [1, 3]
```

---

### **9. `islice` — Срез для итераторов**  
**Что делает:** Работает как срез `[start:stop:step]`, но для итераторов.  
**Пример:**  
```python
from itertools import islice

data = range(10)
result = islice(data, 2, 8, 2)  # Элементы 2, 4, 6
print(list(result))  # [2, 4, 6]
```

---

### **10. `pairwise` — Пары соседних элементов**  
**Что делает:** Создает кортежи из последовательных пар элементов.  
**Пример (Python 3.10+):**  
```python
from itertools import pairwise

data = [1, 2, 3, 4]
result = pairwise(data)
print(list(result))  # [(1, 2), (2, 3), (3, 4)]
```

---

### **11. `starmap` — Распаковка аргументов**  
**Что делает:** Применяет функцию к аргументам, распакованным из кортежей.  
**Пример:**  
```python
from itertools import starmap

data = [(2, 3), (3, 2)]  # (основание, степень)
result = starmap(pow, data)  # pow(2,3)=8, pow(3,2)=9
print(list(result))  # [8, 9]
```

---

### **Итог:**  
- **`count`**, **`cycle`**, **`repeat`** — для работы с бесконечными последовательностями.  
- **`accumulate`**, **`chain`**, **`compress`** — для преобразования данных.  
- **`dropwhile`**, **`takewhile`**, **`islice`** — для фильтрации и срезов.  
- **`pairwise`**, **`starmap`** — для удобной работы с парами и функциями.  

Примеры можно адаптировать под свои задачи. Например, `starmap` полезен для применения функций с несколькими аргументами, а `chain` — для объединения данных из разных источников. 🛠️

Конечно! Разберём ключевые функции и декораторы из модуля `functools` на простых примерах.

---
# functools
```
	○ cache(user_function)
	○ cached_property(func)
	○ lru_cache(user_function)
	○ lru_cache(maxsize=128, typed=False)

@functools.cache
def factorial(n):
	if n <= 1:
		return 1
	return n * factorial(n - 1)
```
### **1. `partial` — Фиксация аргументов функции**
**Что делает:** Создает новую функцию с частично заданными аргументами.  
**Пример:**
```python
from functools import partial

def multiply(a, b):
    return a * b

# Фиксируем аргумент a=5
multiply_by_5 = partial(multiply, 5)

print(multiply_by_5(3))  # 15 (5 * 3)
```
**Где использовать:** Когда нужно создать функцию с предустановленными параметрами (например, для обработки событий с фиксированными значениями).

---

### **2. `partialmethod` — Аналог `partial` для методов класса**
**Что делает:** Фиксирует аргументы методов класса.  
**Пример:**
```python
from functools import partialmethod

class Calculator:
    def add(self, a, b):
        return a + b

    # Фиксируем аргумент a=10 для метода add
    add_10 = partialmethod(add, 10)

calc = Calculator()
print(calc.add_10(5))  # 15 (10 + 5)
```
**Где использовать:** Для создания специализированных методов класса с предустановленными параметрами.

---

### **3. `total_ordering` — Автоматическое создание методов сравнения**
**Что делает:** Генерирует недостающие методы сравнения (`>`, `<=`, и т.д.) на основе `__eq__` и одного из (`__lt__`, `__le__`, `__gt__`, `__ge__`).  
**Пример:**
```python
from functools import total_ordering

@total_ordering
class Student:
    def __init__(self, score):
        self.score = score

    def __eq__(self, other):
        return self.score == other.score

    def __lt__(self, other):
        return self.score < other.score

student1 = Student(80)
student2 = Student(90)

print(student1 <= student2)  # True (80 <= 90)
print(student1 > student2)   # False
```
**Где использовать:** Для классов, где нужно реализовать полный набор операций сравнения без ручного написания каждого метода.

---

### **4. `reduce` — Свёртка последовательности**
**Что делает:** Применяет функцию к элементам последовательности, сводя её к одному значению.  
**Пример:**
```python
from functools import reduce

numbers = [1, 2, 3, 4]

# Сумма элементов: (((1+2)+3)+4) = 10
sum_result = reduce(lambda a, b: a + b, numbers)
print(sum_result)  # 10

# Произведение элементов: 1*2*3*4 = 24
product = reduce(lambda a, b: a * b, numbers, 1)
print(product)  # 24
```
**Где использовать:** Для цепочечных вычислений (например, агрегация данных).

---

### **5. `wraps` и `update_wrapper` — Сохранение метаданных функции**
**Что делает:** Делают декораторы «невидимыми», сохраняя имя и документацию исходной функции.  
**Пример:**
```python
from functools import wraps

def logger(func):
    @wraps(func)  # Сохраняем метаданные func
    def wrapper(*args, **kwargs):
        print(f"Вызов функции {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@logger
def greet(name):
    """Приветствует пользователя."""
    print(f"Привет, {name}!")

print(greet.__name__)  # greet (без @wraps было бы wrapper)
print(greet.__doc__)   # Приветствует пользователя.
```
**Разница между `wraps` и `update_wrapper`:**
- `@wraps(func)` — это удобный декоратор для `update_wrapper`.
- `update_wrapper(wrapper, func)` делает то же самое, но вызывается явно.

---

### **Итог:**
- **`partial`** — Фиксирует аргументы функции.
- **`partialmethod`** — Фиксирует аргументы методов класса.
- **`total_ordering`** — Автоматизирует методы сравнения.
- **`reduce`** — Сворачивает последовательность в одно значение.
- **`wraps`/`update_wrapper`** — Сохраняют метаданные декорируемой функции.

Примеры можно запустить в Python, чтобы увидеть их в действии. 🚀