
## Введение


C-extensions позволяют интегрировать код на C/C++ с Python для повышения производительности критически важных участков кода. Это особенно важно в ML, где вычисления могут быть очень интенсивными.


---
## 1. ctypes

### Что такое ctypes?

`ctypes` - это встроенная библиотека Python, которая позволяет вызывать функции из разделяемых библиотек (DLL в Windows, .so в Linux/macOS) и работать с C-типами данных.

### Основные типы данных

```python

import ctypes
# Основные C-типы
c_int = ctypes.c_int(42)
c_float = ctypes.c_float(3.14)
c_double = ctypes.c_double(2.71828)
c_char = ctypes.c_char(b'A')
c_bool = ctypes.c_bool(True)

print(f"c_int: {c_int.value}")
print(f"c_float: {c_float.value}")

```


### Работа с массивами

  

```python

import ctypes
# Создание массива
ArrayType = ctypes.c_int * 5 # Тип массива из 5 int
array = ArrayType(1, 2, 3, 4, 5) 
# Альтернативный способ
array2 = (ctypes.c_int * 3)()
array2[0] = 10
array2[1] = 20
array2[2] = 30

print("Массив:", [array[i] for i in range(5)])

```

  

### Создание и использование C-библиотеки
**Шаг 1: Создаем C-код (math_lib.c)**


```c

// math_lib.c
#include <math.h>

// Простая функция сложения

int add_numbers(int a, int b) {

return a + b;

}

// Функция для работы с массивами

void multiply_array(double* arr, int size, double multiplier) {

for (int i = 0; i < size; i++) {

arr[i] *= multiplier;

}

}

  

// Функция для вычисления среднего значения

double calculate_mean(double* arr, int size) {

double sum = 0.0;

for (int i = 0; i < size; i++) {

sum += arr[i];

}

return sum / size;

}

```

  

**Шаг 2: Компилируем библиотеку**

  

```bash

# Linux/macOS

gcc -shared -fPIC -o math_lib.so math_lib.c

  

# Windows

gcc -shared -o math_lib.dll math_lib.c

```

  

**Шаг 3: Используем в Python**

  

```python

import ctypes

import sys

  

# Загружаем библиотеку

if sys.platform == "win32":

lib = ctypes.CDLL("./math_lib.dll")

else:

lib = ctypes.CDLL("./math_lib.so")

  

# Определяем сигнатуры функций

# int add_numbers(int a, int b)

lib.add_numbers.argtypes = [ctypes.c_int, ctypes.c_int]

lib.add_numbers.restype = ctypes.c_int

  

# void multiply_array(double* arr, int size, double multiplier)

lib.multiply_array.argtypes = [ctypes.POINTER(ctypes.c_double), ctypes.c_int, ctypes.c_double]

lib.multiply_array.restype = None

  

# double calculate_mean(double* arr, int size)

lib.calculate_mean.argtypes = [ctypes.POINTER(ctypes.c_double), ctypes.c_int]

lib.calculate_mean.restype = ctypes.c_double

  

# Используем функции

result = lib.add_numbers(5, 3)

print(f"5 + 3 = {result}")

  

# Работа с массивом

data = [1.0, 2.0, 3.0, 4.0, 5.0]

array_type = ctypes.c_double * len(data)

c_array = array_type(*data)

  

print(f"Исходный массив: {list(c_array)}")

  

# Умножаем массив на 2

lib.multiply_array(c_array, len(data), 2.0)

print(f"После умножения на 2: {list(c_array)}")

  

# Вычисляем среднее

mean = lib.calculate_mean(c_array, len(data))

print(f"Среднее значение: {mean}")

```

  

---

  

## 2. CFFI (C Foreign Function Interface)

  

### Установка и основы

  

```bash

pip install cffi

```

  

CFFI предоставляет более удобный и безопасный способ работы с C-кодом.

  

### Режимы работы CFFI

  

#### ABI режим (Application Binary Interface)

  

```python

from cffi import FFI

import sys

  

ffi = FFI()

  

# Определяем C-функции

ffi.cdef("""

int add_numbers(int a, int b);

void multiply_array(double* arr, int size, double multiplier);

double calculate_mean(double* arr, int size);

""")

  

# Загружаем библиотеку

if sys.platform == "win32":

lib = ffi.dlopen("./math_lib.dll")

else:

lib = ffi.dlopen("./math_lib.so")

  

# Используем функции

result = lib.add_numbers(10, 20)

print(f"10 + 20 = {result}")

  

# Работа с массивом

data = [1.5, 2.5, 3.5, 4.5, 5.5]

c_array = ffi.new("double[]", data)

  

print(f"Исходные данные: {[c_array[i] for i in range(len(data))]}")

  

lib.multiply_array(c_array, len(data), 1.5)

print(f"После умножения: {[c_array[i] for i in range(len(data))]}")

  

mean = lib.calculate_mean(c_array, len(data))

print(f"Среднее: {mean}")

```

  

#### API режим (более продвинутый)

  

```python

# build_cffi.py

from cffi import FFI

  

ffibuilder = FFI()

  

# Определяем C-код прямо в Python

ffibuilder.cdef("""

double vector_dot_product(double* a, double* b, int size);

void matrix_multiply(double* a, double* b, double* result, int rows_a, int cols_a, int cols_b);

""")

  

ffibuilder.set_source("_ml_math",

"""

#include <stdio.h>

// Скалярное произведение векторов

double vector_dot_product(double* a, double* b, int size) {

double result = 0.0;

for (int i = 0; i < size; i++) {

result += a[i] * b[i];

}

return result;

}

// Умножение матриц

void matrix_multiply(double* a, double* b, double* result, int rows_a, int cols_a, int cols_b) {

for (int i = 0; i < rows_a; i++) {

for (int j = 0; j < cols_b; j++) {

result[i * cols_b + j] = 0.0;

for (int k = 0; k < cols_a; k++) {

result[i * cols_b + j] += a[i * cols_a + k] * b[k * cols_b + j];

}

}

}

}

""")

  

if __name__ == "__main__":

ffibuilder.compile(verbose=True)

```

  

**Использование скомпилированного модуля:**

  

```python

# Сначала запускаем: python build_cffi.py

import _ml_math

from cffi import FFI

  

ffi = FFI()

  

# Скалярное произведение

vec_a = [1.0, 2.0, 3.0, 4.0]

vec_b = [2.0, 3.0, 4.0, 5.0]

  

c_vec_a = ffi.new("double[]", vec_a)

c_vec_b = ffi.new("double[]", vec_b)

  

dot_product = _ml_math.lib.vector_dot_product(c_vec_a, c_vec_b, len(vec_a))

print(f"Скалярное произведение: {dot_product}")

  

# Умножение матриц 2x3 * 3x2

matrix_a = [1.0, 2.0, 3.0, # первая строка

4.0, 5.0, 6.0] # вторая строка

  

matrix_b = [7.0, 8.0, # первая строка

9.0, 10.0, # вторая строка

11.0, 12.0] # третья строка

  

c_matrix_a = ffi.new("double[]", matrix_a)

c_matrix_b = ffi.new("double[]", matrix_b)

c_result = ffi.new("double[]", 4) # 2x2 результат

  

_ml_math.lib.matrix_multiply(c_matrix_a, c_matrix_b, c_result, 2, 3, 2)

  

print("Результат умножения матриц:")

for i in range(2):

for j in range(2):

print(f"{c_result[i * 2 + j]:.1f}", end=" ")

print()

```

  

---

  

## 3. C API (Python C Extension API)

  

C API - это официальный способ создания расширений на C для Python. Более сложен, но дает максимальный контроль.

  

### Простое расширение

  

**mathmodule.c:**

  

```c

#include <Python.h>

#include <math.h>

  

// Функция для быстрого вычисления суммы массива

static PyObject* fast_sum(PyObject* self, PyObject* args) {

PyObject* list_obj;

// Парсим аргументы

if (!PyArg_ParseTuple(args, "O", &list_obj)) {

return NULL;

}

// Проверяем, что это список

if (!PyList_Check(list_obj)) {

PyErr_SetString(PyExc_TypeError, "Argument must be a list");

return NULL;

}

Py_ssize_t size = PyList_Size(list_obj);

double sum = 0.0;

// Суммируем элементы

for (Py_ssize_t i = 0; i < size; i++) {

PyObject* item = PyList_GetItem(list_obj, i);

if (!PyNumber_Check(item)) {

PyErr_SetString(PyExc_TypeError, "All elements must be numbers");

return NULL;

}

double value = PyFloat_AsDouble(item);

if (PyErr_Occurred()) {

return NULL;

}

sum += value;

}

return PyFloat_FromDouble(sum);

}

  

// Функция для вычисления статистик массива

static PyObject* array_stats(PyObject* self, PyObject* args) {

PyObject* list_obj;

if (!PyArg_ParseTuple(args, "O", &list_obj)) {

return NULL;

}

if (!PyList_Check(list_obj)) {

PyErr_SetString(PyExc_TypeError, "Argument must be a list");

return NULL;

}

Py_ssize_t size = PyList_Size(list_obj);

if (size == 0) {

PyErr_SetString(PyExc_ValueError, "List cannot be empty");

return NULL;

}

double sum = 0.0;

double min_val = INFINITY;

double max_val = -INFINITY;

// Проходим по всем элементам

for (Py_ssize_t i = 0; i < size; i++) {

PyObject* item = PyList_GetItem(list_obj, i);

double value = PyFloat_AsDouble(item);

if (PyErr_Occurred()) {

return NULL;

}

sum += value;

if (value < min_val) min_val = value;

if (value > max_val) max_val = value;

}

double mean = sum / size;

// Вычисляем стандартное отклонение

double variance = 0.0;

for (Py_ssize_t i = 0; i < size; i++) {

PyObject* item = PyList_GetItem(list_obj, i);

double value = PyFloat_AsDouble(item);

double diff = value - mean;

variance += diff * diff;

}

variance /= size;

double std_dev = sqrt(variance);

// Возвращаем словарь с результатами

PyObject* result = PyDict_New();

PyDict_SetItemString(result, "mean", PyFloat_FromDouble(mean));

PyDict_SetItemString(result, "min", PyFloat_FromDouble(min_val));

PyDict_SetItemString(result, "max", PyFloat_FromDouble(max_val));

PyDict_SetItemString(result, "std", PyFloat_FromDouble(std_dev));

PyDict_SetItemString(result, "sum", PyFloat_FromDouble(sum));

return result;

}

  

// Определяем методы модуля

static PyMethodDef MathMethods[] = {

{"fast_sum", fast_sum, METH_VARARGS, "Calculate sum of list elements"},

{"array_stats", array_stats, METH_VARARGS, "Calculate array statistics"},

{NULL, NULL, 0, NULL}

};

  

// Определяем модуль

static struct PyModuleDef mathmodule = {

PyModuleDef_HEAD_INIT,

"fastmath",

"Fast math operations",

-1,

MathMethods

};

  

// Инициализация модуля

PyMODINIT_FUNC PyInit_fastmath(void) {

return PyModule_Create(&mathmodule);

}

```

  

**setup.py:**

  

```python

from distutils.core import setup, Extension

  

module = Extension('fastmath', sources=['mathmodule.c'])

  

setup(

name='FastMath',

version='1.0',

description='Fast mathematical operations',

ext_modules=[module]

)

```

  

**Компиляция и использование:**

  

```bash

python setup.py build_ext --inplace

```

  

```python

import fastmath

import time

  

# Тестируем производительность

data = list(range(1000000))

  

# Python sum

start = time.time()

python_sum = sum(data)

python_time = time.time() - start

  

# C sum

start = time.time()

c_sum = fastmath.fast_sum(data)

c_time = time.time() - start

  

print(f"Python sum: {python_sum}, время: {python_time:.4f}s")

print(f"C sum: {c_sum}, время: {c_time:.4f}s")

print(f"Ускорение: {python_time/c_time:.2f}x")

  

# Тестируем статистики

small_data = [1.5, 2.3, 4.1, 3.7, 2.9, 5.2, 1.8]

stats = fastmath.array_stats(small_data)

print(f"Статистики: {stats}")

```

  

---

  

## 4. Cython

  

Cython позволяет писать C-расширения, используя Python-подобный синтаксис с аннотациями типов.

  

### Установка

  

```bash

pip install cython

```

  

### Основы Cython

  

**math_ops.pyx:**

  

```python

# math_ops.pyx

import cython

import numpy as np

cimport numpy as cnp

from libc.math cimport sqrt, pow

  

# Включаем проверки границ для безопасности (отключить в продакшене)

@cython.boundscheck(False)

@cython.wraparound(False)

def fast_euclidean_distance(double[:] a, double[:] b):

"""

Быстрое вычисление евклидового расстояния между двумя векторами

"""

cdef int n = a.shape[0]

cdef double sum_sq = 0.0

cdef int i

if n != b.shape[0]:

raise ValueError("Vectors must have the same length")

for i in range(n):

sum_sq += pow(a[i] - b[i], 2)

return sqrt(sum_sq)

  

@cython.boundscheck(False)

@cython.wraparound(False)

def matrix_vector_multiply(double[:, :] matrix, double[:] vector):

"""

Умножение матрицы на вектор

"""

cdef int rows = matrix.shape[0]

cdef int cols = matrix.shape[1]

cdef int i, j

if cols != vector.shape[0]:

raise ValueError("Matrix columns must match vector length")

# Создаем результирующий массив

result = np.zeros(rows, dtype=np.float64)

cdef double[:] result_view = result

for i in range(rows):

for j in range(cols):

result_view[i] += matrix[i, j] * vector[j]

return result

  

# Класс для работы с точками в 2D

cdef class Point2D:

cdef double x, y

def __init__(self, double x, double y):

self.x = x

self.y = y

def distance_to(self, Point2D other):

return sqrt(pow(self.x - other.x, 2) + pow(self.y - other.y, 2))

def __repr__(self):

return f"Point2D({self.x}, {self.y})"

  

# Функция для кластеризации точек (простой k-means шаг)

@cython.boundscheck(False)

def assign_clusters(points, centroids):

"""

Назначает каждую точку к ближайшему центроиду

"""

cdef int n_points = len(points)

cdef int n_centroids = len(centroids)

cdef int i, j, best_cluster

cdef double min_dist, dist

clusters = np.zeros(n_points, dtype=np.int32)

cdef int[:] clusters_view = clusters

for i in range(n_points):

min_dist = float('inf')

best_cluster = 0

for j in range(n_centroids):

dist = points[i].distance_to(centroids[j])

if dist < min_dist:

min_dist = dist

best_cluster = j

clusters_view[i] = best_cluster

return clusters

  

# Оптимизированная функция для вычисления градиента

@cython.boundscheck(False)

@cython.wraparound(False)

def compute_gradient(double[:, :] X, double[:] y, double[:] weights):

"""

Вычисляет градиент для линейной регрессии

"""

cdef int n_samples = X.shape[0]

cdef int n_features = X.shape[1]

cdef int i, j

cdef double prediction, error

gradient = np.zeros(n_features, dtype=np.float64)

cdef double[:] grad_view = gradient

for i in range(n_samples):

# Вычисляем предсказание

prediction = 0.0

for j in range(n_features):

prediction += X[i, j] * weights[j]

# Вычисляем ошибку

error = prediction - y[i]

# Обновляем градиент

for j in range(n_features):

grad_view[j] += error * X[i, j]

# Нормализуем на количество образцов

for j in range(n_features):

grad_view[j] /= n_samples

return gradient

```

  

**setup.py для Cython:**

  

```python

from setuptools import setup

from Cython.Build import cythonize

import numpy

  

setup(

ext_modules=cythonize("math_ops.pyx"),

include_dirs=[numpy.get_include()],

zip_safe=False,

)

```

  

**Компиляция и использование:**

  

```bash

python setup.py build_ext --inplace

```

  

```python

import math_ops

import numpy as np

import time

  

# Тестируем евклидово расстояние

a = np.array([1.0, 2.0, 3.0])

b = np.array([4.0, 5.0, 6.0])

  

distance = math_ops.fast_euclidean_distance(a, b)

print(f"Евклидово расстояние: {distance}")

  

# Тестируем умножение матрицы на вектор

matrix = np.array([[1.0, 2.0, 3.0],

[4.0, 5.0, 6.0]], dtype=np.float64)

vector = np.array([1.0, 2.0, 3.0], dtype=np.float64)

  

result = math_ops.matrix_vector_multiply(matrix, vector)

print(f"Результат умножения: {result}")

  

# Тестируем работу с классом Point2D

point1 = math_ops.Point2D(0.0, 0.0)

point2 = math_ops.Point2D(3.0, 4.0)

dist = point1.distance_to(point2)

print(f"Расстояние между точками: {dist}")

  

# Тестируем кластеризацию

points = [

math_ops.Point2D(1.0, 1.0),

math_ops.Point2D(2.0, 1.0),

math_ops.Point2D(1.0, 2.0),

math_ops.Point2D(5.0, 5.0),

math_ops.Point2D(6.0, 5.0),

math_ops.Point2D(5.0, 6.0),

]

  

centroids = [

math_ops.Point2D(1.5, 1.5),

math_ops.Point2D(5.5, 5.5),

]

  

clusters = math_ops.assign_clusters(points, centroids)

print(f"Назначения кластеров: {clusters}")

  

# Сравниваем производительность градиента

np.random.seed(42)

X = np.random.randn(1000, 10).astype(np.float64)

y = np.random.randn(1000).astype(np.float64)

weights = np.random.randn(10).astype(np.float64)

  

# Cython версия

start = time.time()

grad_cython = math_ops.compute_gradient(X, y, weights)

cython_time = time.time() - start

  

# Numpy версия

start = time.time()

predictions = X @ weights

errors = predictions - y

grad_numpy = X.T @ errors / len(y)

numpy_time = time.time() - start

  

print(f"Cython время: {cython_time:.4f}s")

print(f"NumPy время: {numpy_time:.4f}s")

print(f"Ускорение: {numpy_time/cython_time:.2f}x")

print(f"Результаты близки: {np.allclose(grad_cython, grad_numpy)}")

```

  

---

  

## Практические задачи

  

### Задача 1: Реализация быстрого скалярного произведения

  

Реализуйте функцию вычисления скалярного произведения двух векторов используя **один из изученных методов** (ctypes, CFFI, C API, или Cython). Сравните производительность с NumPy.

  

**Требования:**

- Функция должна принимать два списка чисел

- Возвращать скалярное произведение

- Обрабатывать ошибки (разная длина векторов, не числовые элементы)

  

### Задача 2: Фильтрация изображений

  

Используя **Cython**, реализуйте простой фильтр размытия для изображения (усреднение в окне 3x3).

  

**Требования:**

- Функция принимает 2D массив NumPy (изображение в градациях серого)

- Применяет фильтр размытия к каждому пикселю

- Обрабатывает границы изображения

- Возвращает отфильтрованное изображение

  

**Подсказка:** Используйте `@cython.boundscheck(False)` для ускорения и работайте с memory views.

  

---

  

## Заключение

  

Каждый из методов имеет свои преимущества:

  

- **ctypes**: Простой, для работы с существующими C-библиотеками

- **CFFI**: Более безопасный и удобный чем ctypes

- **C API**: Максимальная производительность и контроль

- **Cython**: Лучший баланс между простотой и производительностью

  

Для ML-задач чаще всего используют Cython, так как он позволяет легко оптимизировать узкие места в Python-коде без полного переписывания на C.