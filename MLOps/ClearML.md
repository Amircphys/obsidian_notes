
# ClearML: Полное руководство по инструменту MLOps

  

## Содержание

1. [Введение в ClearML](#введение)

2. [Архитектура ClearML](#архитектура)

3. [Установка и настройка](#установка)

4. [Основные компоненты](#компоненты)

5. [Эксперименты](#эксперименты)

6. [Работа с данными](#данные)

7. [Управление моделями](#модели)

8. [Автоматизация и пайплайны](#пайплайны)

9. [Практические примеры](#примеры)

10. [Лучшие практики](#практики)

  

## 1. Введение в ClearML {#введение}

  

### Что такое ClearML?

  

ClearML (ранее Allegro Trains) — это open-source платформа для MLOps, которая помогает исследователям и инженерам машинного обучения управлять всем жизненным циклом ML-проектов. Это комплексное решение для:

  

- **Отслеживания экспериментов** (experiment tracking)

- **Управления данными** (data management)

- **Версионирования моделей** (model versioning)

- **Оркестрации пайплайнов** (pipeline orchestration)

- **Удаленного выполнения задач** (remote execution)

  

### Зачем нужен ClearML?

  

**Проблемы, которые решает ClearML:**

  

1. **Хаос в экспериментах**: Без системы отслеживания легко потерять важные результаты

2. **Невоспроизводимость**: Сложно повторить успешный эксперимент

3. **Потеря данных**: Версии датасетов и моделей теряются

4. **Неэффективное использование ресурсов**: Ручное управление вычислительными ресурсами

5. **Сложность деплоя**: Отсутствие единой системы развертывания моделей

  

**Преимущества ClearML:**

  

- 📊 Автоматическое логирование экспериментов

- 🔄 Полная воспроизводимость результатов

- 📈 Визуализация метрик и артефактов

- 🗄️ Централизованное хранение моделей и данных

- 🚀 Масштабируемое выполнение задач

- 🔗 Интеграция с популярными ML-фреймворками

  

## 2. Архитектура ClearML {#архитектура}

  

### Основные компоненты архитектуры

  

```

┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐

│ ClearML SDK │ │ ClearML Agent │ │ ClearML Server │

│ │ │ │ │ │

│ • Python API │◄──►│ • Task Runner │◄──►│ • Web UI │

│ • Auto-logging │ │ • Queue Manager │ │ • API Server │

│ • Data handling │ │ • Environment │ │ • File Server │

└─────────────────┘ └─────────────────┘ └─────────────────┘

```

  

**ClearML Server:**

- **Web UI**: Веб-интерфейс для просмотра экспериментов

- **API Server**: REST API для взаимодействия

- **File Server**: Хранение артефактов и моделей

  

**ClearML SDK:**

- Python библиотека для интеграции в код

- Автоматическое отслеживание экспериментов

- Управление данными и моделями

  

**ClearML Agent:**

- Исполнитель задач на удаленных машинах

- Управление очередями задач

- Автоматическая установка зависимостей

  

## 3. Установка и настройка {#установка}

  

### Установка ClearML SDK

  

```bash

# Установка основного пакета

pip install clearml

  

# Установка с дополнительными зависимостями

pip install clearml[s3,gs,azure] # для облачных хранилищ

```

  

### Настройка учетных данных

  

```bash

# Инициализация конфигурации

clearml-init

```

  

После выполнения команды будет создан файл `~/clearml.conf`:

  

```ini

api {

web_server: https://app.clear.ml

api_server: https://api.clear.ml

files_server: https://files.clear.ml

credentials {

"access_key": "YOUR_ACCESS_KEY"

"secret_key": "YOUR_SECRET_KEY"

}

}

```

  

### Локальная установка ClearML Server (опционально)

  

```bash

# Запуск через Docker Compose

curl https://raw.githubusercontent.com/allegroai/clearml-server/master/docker/docker-compose.yml -o docker-compose.yml

docker-compose up -d

```

  

## 4. Основные компоненты {#компоненты}

  

### Task (Задача)

  

Task — основная единица работы в ClearML. Каждый эксперимент автоматически становится задачей.

  

```python

from clearml import Task

  

# Создание задачи

task = Task.init(

project_name="My Project", # Название проекта

task_name="Experiment 1", # Название эксперимента

task_type="training" # Тип задачи

)

  

# Получение логгера

logger = task.get_logger()

```

  

### Project (Проект)

  

Проект группирует связанные задачи. Создается автоматически при первом использовании.

  

```python

# Проекты могут иметь иерархическую структуру

task = Task.init(

project_name="Computer Vision/Image Classification",

task_name="ResNet50 Training"

)

```

  

### Logger (Логгер)

  

Логгер отвечает за запись метрик, изображений, текста и других артефактов.

  

```python

logger = task.get_logger()

  

# Логирование скалярных метрик

logger.report_scalar("Loss", "Training", iteration=100, value=0.5)

  

# Логирование изображений

logger.report_image("Samples", "Training Batch",

iteration=100, image=image_array)

  

# Логирование гистограмм

logger.report_histogram("Weights", "Layer1",

iteration=100, values=weights)

```

  

## 5. Эксперименты {#эксперименты}

  

### Базовый пример отслеживания эксперимента

  

```python

from clearml import Task

import numpy as np

import matplotlib.pyplot as plt

from sklearn.model_selection import train_test_split

from sklearn.ensemble import RandomForestClassifier

from sklearn.datasets import make_classification

from sklearn.metrics import accuracy_score, confusion_matrix

import seaborn as sns

  

# Инициализация задачи

task = Task.init(

project_name="ML Classification",

task_name="Random Forest Experiment",

tags=["sklearn", "classification", "random-forest"]

)

  

# Получение логгера

logger = task.get_logger()

  

# Параметры эксперимента

params = {

'n_estimators': 100,

'max_depth': 10,

'random_state': 42,

'n_samples': 1000,

'n_features': 20,

'n_classes': 3

}

  

# Подключение параметров к задаче

task.connect(params)

  

# Генерация данных

X, y = make_classification(

n_samples=params['n_samples'],

n_features=params['n_features'],

n_classes=params['n_classes'],

n_informative=15,

random_state=params['random_state']

)

  

# Разделение данных

X_train, X_test, y_train, y_test = train_test_split(

X, y, test_size=0.2, random_state=params['random_state']

)

  

# Создание и обучение модели

model = RandomForestClassifier(

n_estimators=params['n_estimators'],

max_depth=params['max_depth'],

random_state=params['random_state']

)

  

print("Начинаем обучение...")

model.fit(X_train, y_train)

  

# Предсказания

y_pred = model.predict(X_test)

accuracy = accuracy_score(y_test, y_pred)

  

# Логирование метрик

logger.report_scalar("Metrics", "Accuracy", value=accuracy, iteration=0)

logger.report_scalar("Data", "Train Size", value=len(X_train), iteration=0)

logger.report_scalar("Data", "Test Size", value=len(X_test), iteration=0)

  

# Логирование confusion matrix

cm = confusion_matrix(y_test, y_pred)

plt.figure(figsize=(8, 6))

sns.heatmap(cm, annot=True, fmt='d', cmap='Blues')

plt.title('Confusion Matrix')

plt.ylabel('True Label')

plt.xlabel('Predicted Label')

logger.report_matplotlib_figure("Confusion Matrix", "Test Set",

figure=plt.gcf(), iteration=0)

plt.close()

  

# Логирование важности признаков

feature_importance = model.feature_importances_

plt.figure(figsize=(10, 6))

indices = np.argsort(feature_importance)[::-1][:10]

plt.bar(range(10), feature_importance[indices])

plt.title('Top 10 Feature Importances')

plt.xlabel('Feature Index')

plt.ylabel('Importance')

logger.report_matplotlib_figure("Feature Importance", "Top 10",

figure=plt.gcf(), iteration=0)

plt.close()

  

print(f"Эксперимент завершен! Точность: {accuracy:.4f}")

```

  

### Автоматическое отслеживание популярных фреймворков

  

#### Пример с PyTorch

  

```python

from clearml import Task

import torch

import torch.nn as nn

import torch.optim as optim

from torch.utils.data import DataLoader, TensorDataset

  

# Инициализация задачи

task = Task.init(project_name="Deep Learning", task_name="PyTorch CNN")

  

# ClearML автоматически отслеживает:

# - Архитектуру модели

# - Параметры оптимизатора

# - Значения loss и метрик

# - Градиенты (опционально)

  

class SimpleCNN(nn.Module):

def __init__(self, num_classes=10):

super(SimpleCNN, self).__init__()

self.conv1 = nn.Conv2d(1, 32, 3, 1)

self.conv2 = nn.Conv2d(32, 64, 3, 1)

self.dropout1 = nn.Dropout2d(0.25)

self.dropout2 = nn.Dropout2d(0.5)

self.fc1 = nn.Linear(9216, 128)

self.fc2 = nn.Linear(128, num_classes)

  

def forward(self, x):

x = self.conv1(x)

x = torch.relu(x)

x = self.conv2(x)

x = torch.relu(x)

x = torch.max_pool2d(x, 2)

x = self.dropout1(x)

x = torch.flatten(x, 1)

x = self.fc1(x)

x = torch.relu(x)

x = self.dropout2(x)

x = self.fc2(x)

return torch.log_softmax(x, dim=1)

  

# Создание модели

model = SimpleCNN()

optimizer = optim.Adam(model.parameters(), lr=0.001)

criterion = nn.NLLLoss()

  

# Фиктивные данные для примера

X = torch.randn(1000, 1, 28, 28)

y = torch.randint(0, 10, (1000,))

dataset = TensorDataset(X, y)

dataloader = DataLoader(dataset, batch_size=32, shuffle=True)

  

# Обучение

logger = task.get_logger()

for epoch in range(5):

running_loss = 0.0

correct = 0

total = 0

for batch_idx, (data, target) in enumerate(dataloader):

optimizer.zero_grad()

output = model(data)

loss = criterion(output, target)

loss.backward()

optimizer.step()

running_loss += loss.item()

_, predicted = torch.max(output.data, 1)

total += target.size(0)

correct += (predicted == target).sum().item()

# Логирование каждые 10 батчей

if batch_idx % 10 == 0:

iteration = epoch * len(dataloader) + batch_idx

logger.report_scalar("Loss", "Training",

iteration=iteration, value=loss.item())

# Логирование эпохи

epoch_loss = running_loss / len(dataloader)

epoch_acc = 100 * correct / total

logger.report_scalar("Loss", "Epoch", iteration=epoch, value=epoch_loss)

logger.report_scalar("Accuracy", "Epoch", iteration=epoch, value=epoch_acc)

print(f'Epoch {epoch+1}: Loss={epoch_loss:.4f}, Accuracy={epoch_acc:.2f}%')

```

  

#### Пример с TensorFlow/Keras

  

```python

from clearml import Task

import tensorflow as tf

from tensorflow import keras

import numpy as np

  

# Инициализация задачи

task = Task.init(project_name="Deep Learning", task_name="TensorFlow MNIST")

  

# Загрузка данных MNIST

(x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()

  

# Предобработка

x_train = x_train.reshape(-1, 28, 28, 1).astype('float32') / 255.0

x_test = x_test.reshape(-1, 28, 28, 1).astype('float32') / 255.0

y_train = keras.utils.to_categorical(y_train, 10)

y_test = keras.utils.to_categorical(y_test, 10)

  

# Создание модели

model = keras.Sequential([

keras.layers.Conv2D(32, (3, 3), activation='relu', input_shape=(28, 28, 1)),

keras.layers.MaxPooling2D((2, 2)),

keras.layers.Conv2D(64, (3, 3), activation='relu'),

keras.layers.MaxPooling2D((2, 2)),

keras.layers.Conv2D(64, (3, 3), activation='relu'),

keras.layers.Flatten(),

keras.layers.Dense(64, activation='relu'),

keras.layers.Dense(10, activation='softmax')

])

  

# Компиляция модели

model.compile(

optimizer='adam',

loss='categorical_crossentropy',

metrics=['accuracy']

)

  

# ClearML автоматически создаст колбэк для логирования

# Дополнительные колбэки (опционально)

from clearml.binding.keras_bind import KerasModelLogger

  

# Обучение модели

history = model.fit(

x_train, y_train,

batch_size=128,

epochs=10,

validation_data=(x_test, y_test),

verbose=1

)

  

# Дополнительное логирование

logger = task.get_logger()

  

# Логирование архитектуры модели

model_summary = []

model.summary(print_fn=lambda x: model_summary.append(x))

logger.report_text('\n'.join(model_summary), print_console=False)

  

# Оценка модели

test_loss, test_accuracy = model.evaluate(x_test, y_test, verbose=0)

logger.report_scalar("Final Metrics", "Test Loss", value=test_loss, iteration=0)

logger.report_scalar("Final Metrics", "Test Accuracy", value=test_accuracy, iteration=0)

  

print(f"Test accuracy: {test_accuracy:.4f}")

```

  

### Сравнение экспериментов

  

```python

from clearml import Task

  

# Функция для запуска эксперимента с разными параметрами

def run_experiment(learning_rate, batch_size, epochs):

task = Task.init(

project_name="Hyperparameter Tuning",

task_name=f"LR-{learning_rate}_BS-{batch_size}_E-{epochs}"

)

# Параметры

params = {

'learning_rate': learning_rate,

'batch_size': batch_size,

'epochs': epochs

}

task.connect(params)

logger = task.get_logger()

# Симуляция обучения

for epoch in range(epochs):

# Симуляция метрик (в реальности здесь был бы код обучения)

train_loss = np.random.exponential(scale=1.0) * (0.9 ** epoch)

val_loss = train_loss * (1 + np.random.normal(0, 0.1))

val_accuracy = 1 - val_loss * 0.1 + np.random.normal(0, 0.02)

logger.report_scalar("Loss", "Training", iteration=epoch, value=train_loss)

logger.report_scalar("Loss", "Validation", iteration=epoch, value=val_loss)

logger.report_scalar("Accuracy", "Validation", iteration=epoch, value=val_accuracy)

return val_accuracy

  

# Запуск нескольких экспериментов

import numpy as np

  

learning_rates = [0.001, 0.01, 0.1]

batch_sizes = [32, 64, 128]

  

results = []

for lr in learning_rates:

for bs in batch_sizes:

final_accuracy = run_experiment(lr, bs, epochs=10)

results.append({

'lr': lr,

'batch_size': bs,

'final_accuracy': final_accuracy

})

  

# Вывод результатов

for result in sorted(results, key=lambda x: x['final_accuracy'], reverse=True):

print(f"LR: {result['lr']}, BS: {result['batch_size']}, "

f"Accuracy: {result['final_accuracy']:.4f}")

```

  

## 6. Работа с данными {#данные}

  

### Dataset Management

  

ClearML предоставляет мощные инструменты для управления датасетами:

  

```python

from clearml import Dataset

import pandas as pd

import numpy as np

from pathlib import Path

  

# Создание нового датасета

dataset = Dataset.create(

dataset_name="Customer Data v1.0",

dataset_project="Data Management",

description="Customer dataset with features and labels"

)

  

# Генерация примера данных

np.random.seed(42)

n_samples = 10000

  

data = {

'customer_id': range(1, n_samples + 1),

'age': np.random.randint(18, 80, n_samples),

'income': np.random.normal(50000, 15000, n_samples),

'spending_score': np.random.randint(1, 100, n_samples),

'years_customer': np.random.randint(0, 20, n_samples),

'purchased': np.random.choice([0, 1], n_samples, p=[0.7, 0.3])

}

  

df = pd.DataFrame(data)

  

# Создание локальной папки с данными

data_dir = Path("./dataset_example")

data_dir.mkdir(exist_ok=True)

  

# Сохранение файлов

df.to_csv(data_dir / "customer_data.csv", index=False)

  

# Создание метаданных

metadata = {

"version": "1.0",

"total_samples": len(df),

"features": list(df.columns[:-1]),

"target": "purchased",

"class_distribution": df['purchased'].value_counts().to_dict()

}

  

import json

with open(data_dir / "metadata.json", "w") as f:

json.dump(metadata, f, indent=2)

  

# Добавление файлов в датасет

dataset.add_files(path=str(data_dir))

  

# Загрузка датасета в хранилище

dataset.upload()

  

# Финализация датасета

dataset.finalize()

  

print(f"Dataset created with ID: {dataset.id}")

print(f"Dataset URL: {dataset.get_default_storage()}")

```

  

### Использование существующего датасета

  

```python

from clearml import Dataset, Task

import pandas as pd

  

# Инициализация задачи

task = Task.init(project_name="Data Processing", task_name="Load Dataset Example")

  

# Получение датасета по имени и проекту

dataset = Dataset.get(

dataset_name="Customer Data v1.0",

dataset_project="Data Management"

)

  

# Загрузка локальной копии

local_dataset_path = dataset.get_local_copy()

print(f"Dataset downloaded to: {local_dataset_path}")

  

# Загрузка данных

data_file = Path(local_dataset_path) / "customer_data.csv"

df = pd.read_csv(data_file)

  

# Загрузка метаданных

metadata_file = Path(local_dataset_path) / "metadata.json"

with open(metadata_file, 'r') as f:

metadata = json.load(f)

  

print(f"Dataset info:")

print(f"- Samples: {metadata['total_samples']}")

print(f"- Features: {metadata['features']}")

print(f"- Target: {metadata['target']}")

print(f"- Class distribution: {metadata['class_distribution']}")

  

# Логирование информации о датасете

logger = task.get_logger()

logger.report_table("Dataset Info", "Metadata",

table_plot=pd.DataFrame([metadata]))

  

# Базовая статистика

stats = df.describe()

logger.report_table("Dataset Stats", "Descriptive Statistics",

table_plot=stats)

```

  

### Версионирование датасетов

  

```python

# Создание новой версии датасета

def create_dataset_version(parent_dataset_id, version_name, modifications):

# Получение родительского датасета

parent_dataset = Dataset.get(dataset_id=parent_dataset_id)

# Создание новой версии

dataset = Dataset.create(

dataset_name=parent_dataset.name,

dataset_project=parent_dataset.project,

description=f"Updated version: {version_name}",

parent_datasets=[parent_dataset_id]

)

# Получение данных родительского датасета

parent_path = parent_dataset.get_local_copy()

df = pd.read_csv(Path(parent_path) / "customer_data.csv")

# Применение модификаций

if 'remove_outliers' in modifications:

# Удаление выбросов по доходу

q1 = df['income'].quantile(0.25)

q3 = df['income'].quantile(0.75)

iqr = q3 - q1

lower_bound = q1 - 1.5 * iqr

upper_bound = q3 + 1.5 * iqr

df = df[(df['income'] >= lower_bound) & (df['income'] <= upper_bound)]

if 'add_features' in modifications:

# Добавление новых признаков

df['income_per_year'] = df['income'] / df['years_customer'].replace(0, 1)

df['age_group'] = pd.cut(df['age'], bins=[0, 30, 50, 100],

labels=['Young', 'Middle', 'Senior'])

# Сохранение новой версии

new_data_dir = Path("./dataset_v2")

new_data_dir.mkdir(exist_ok=True)

df.to_csv(new_data_dir / "customer_data.csv", index=False)

# Обновление метаданных

metadata = {

"version": version_name,

"parent_version": parent_dataset.name,

"total_samples": len(df),

"features": [col for col in df.columns if col != 'purchased'],

"target": "purchased",

"class_distribution": df['purchased'].value_counts().to_dict(),

"modifications": modifications

}

with open(new_data_dir / "metadata.json", "w") as f:

json.dump(metadata, f, indent=2)

# Добавление в датасет

dataset.add_files(path=str(new_data_dir))

dataset.upload()

dataset.finalize()

return dataset

  

# Создание новой версии с улучшениями

parent_id = "your_dataset_id_here" # ID предыдущего датасета

new_dataset = create_dataset_version(

parent_id,

"v2.0",

['remove_outliers', 'add_features']

)

  

print(f"New dataset version created: {new_dataset.id}")

```

  

## 7. Управление моделями {#модели}

  

### Сохранение и загрузка моделей

  

```python

from clearml import Task, OutputModel

import joblib

from sklearn.ensemble import RandomForestClassifier

from sklearn.datasets import make_classification

from sklearn.model_selection import train_test_split

from sklearn.metrics import accuracy_score

import numpy as np

  

# Инициализация задачи

task = Task.init(project_name="Model Management", task_name="Model Training and Saving")

  

# Подготовка данных

X, y = make_classification(n_samples=1000, n_features=20, n_classes=2, random_state=42)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

  

# Обучение модели

model = RandomForestClassifier(n_estimators=100, random_state=42)

model.fit(X_train, y_train)

  

# Оценка модели

y_pred = model.predict(X_test)

accuracy = accuracy_score(y_test, y_pred)

  

# Логирование метрик

logger = task.get_logger()

logger.report_scalar("Performance", "Accuracy", value=accuracy, iteration=0)

  

# Сохранение модели локально

model_filename = "random_forest_model.pkl"

joblib.dump(model, model_filename)

  

# Создание OutputModel для ClearML

output_model = OutputModel(

task=task,

framework="scikit-learn",

name="Random Forest Classifier"

)

  

# Добавление метаданных модели

output_model.set_model_metadata({

"accuracy": accuracy,

"n_features": X.shape[1],

"n_classes": len(np.unique(y)),

"model_type": "RandomForestClassifier",

"hyperparameters": {

"n_estimators": 100,

"random_state": 42

}

})

  

# Загрузка файла модели

output_model.update_weights(weights_filename=model_filename)

  

print(f"Model saved with accuracy: {accuracy:.4f}")

print(f"Model ID: {output_model.id}")

```

  

### Загрузка и использование сохраненных моделей

  

```python

from clearml import Task, InputModel

import joblib

import numpy as np

  

# Инициализация новой задачи для инференса

task = Task.init(project_name="Model Management", task_name="Model Inference")

  

# Загрузка модели по ID

model_id = "your_model_id_here" # ID модели из предыдущего примера

input_model = InputModel(model_id=model_id)

  

# Получение локального пути к файлу модели

model_path = input_model.get_local_copy()

print(f"Model downloaded to: {model_path}")

  

# Загрузка модели

model = joblib.load(model_path)

  

# Получение метаданных модели

metadata = input_model.get_model_metadata()

print("Model metadata:")

for key, value in metadata.items():

print(f" {key}: {value}")

  

# Генерация новых данных для предсказания

np.random.seed(123)

new_data = np.random.randn(5, 20) # 5 образцов с 20 признаками

  

# Предсказания

predictions = model.predict(new_data)

probabilities = model.predict_proba(new_data)

  

# Логирование результатов

logger = task.get_logger()

  

# Создание таблицы с результатами

import pandas as pd

results_df = pd.DataFrame({

'Sample': range(1, len(predictions) + 1),

'Prediction': predictions,

'Probability_Class_0': probabilities[:, 0],

'Probability_Class_1': probabilities[:, 1]

})

  

logger.report_table("Predictions", "Results", table_plot=results_df)

  

print("Predictions completed:")

print(results_df)

```

  

### Model Registry - Управление версиями моделей

  

```python

from clearml import Task, Model

import joblib

from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier

from sklearn.datasets import make_classification

from sklearn.model_selection import train_test_split, cross_val_score

from sklearn.metrics import accuracy_score

import numpy as np

  

def train_and_register_model(model_class, model_params, model_name, tags=None):

"""Обучает модель и регистрирует её в Model Registry"""

# Создание задачи

task = Task.init(

project_name="Model Registry",

task_name=f"Training {model_name}",

tags=tags or []

)

# Подготовка данных

X, y = make_classification(n_samples=1000, n_features=20, n_classes=2, random_state=42)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Создание и обучение модели

model = model_class(**model_params)

model.fit(X_train, y_train)

# Оценка модели

y_pred = model.predict(X_test)

accuracy = accuracy_score(y_test, y_pred)

# Кросс-валидация

cv_scores = cross_val_score(model, X_train, y_train, cv=5)

# Логирование метрик

logger = task.get_logger()

logger.report_scalar("Performance", "Test Accuracy", value=accuracy, iteration=0)

logger.report_scalar("Performance", "CV Mean", value=cv_scores.mean(), iteration=0)

logger.report_scalar("Performance", "CV Std", value=cv_scores.std(), iteration=0)

# Сохранение модели

model_filename = f"{model_name.lower().replace(' ', '_')}.pkl"

joblib.dump(model, model_filename)

# Создание модели в реестре

from clearml import OutputModel

output_model = OutputModel(

task=task,

framework="scikit-learn",

name=model_name

)

# Метаданные модели

metadata = {

"test_accuracy": float(accuracy),

"cv_mean_accuracy": float(cv_scores.mean()),

"cv_std_accuracy": float(cv_scores.std()),

"n_features": X.shape[1],

"n_samples_train": len(X_train),

"n_samples_test": len(X_test),

"model_params": model_params,

"model_class": model_class.__name__

}

output_model.set_model_metadata(metadata)

output_model.update_weights(weights_filename=model_filename)

# Добавление тегов

if tags:

output_model.set_model_tags(tags)

return output_model, accuracy

  

# Обучение разных моделей

models_to_train = [

{

'class': RandomForestClassifier,

'params': {'n_estimators': 100, 'max_depth': 10, 'random_state': 42},

'name': 'Random Forest v1',

'tags': ['baseline', 'tree-based']

},

{

'class': RandomForestClassifier,

'params': {'n_estimators': 200, 'max_depth': 15, 'random_state': 42},

'name': 'Random Forest v2',

'tags': ['improved', 'tree-based']

},

{

'class': GradientBoostingClassifier,

'params': {'n_estimators': 100, 'learning_rate': 0.1, 'random_state': 42},

'name': 'Gradient Boosting v1',

'tags': ['boosting', 'tree-based']

}

]

  

# Обучение и регистрация всех моделей

model_results = []

for model_config in models_to_train:

print(f"Training {model_config['name']}...")

model, accuracy = train_and_register_model(

model_config['class'],

model_config['params'],

model_config['name'],

model_config['tags']

)

model_results.append({

'name': model_config['name'],

'model_id': model.id,

'accuracy': accuracy

})

print(f" Completed. Accuracy: {accuracy:.4f}, Model ID: {model.id}")

  

# Сравнение результатов

print("\nModel Comparison:")

print("-" * 50)

for result in sorted(model_results, key=lambda x: x['accuracy'], reverse=True):

print(f"{result['name']:<20} | Accuracy: {result['accuracy']:.4f} | ID: {result['model_id']}")

```

  

### Продвинутая работа с Model Registry

  

```python

from clearml import Model

import pandas as pd

  

def compare_models_in_project(project_name):

"""Сравнивает все модели в проекте"""

# Получение всех моделей в проекте

models = Model.query_models(project_name=project_name)

model_data = []

for model in models:

metadata = model.get_model_metadata()

model_info = {

'name': model.name,

'id': model.id,

'created': model.created,

'framework': model.framework,

'tags': ', '.join(model.tags) if model.tags else '',

}

# Добавление метрик из метаданных

if metadata:

model_info.update({

key: value for key, value in metadata.items()

if isinstance(value, (int, float, str)) and len(str(value)) < 50

})

model_data.append(model_info)

# Создание DataFrame для анализа

df = pd.DataFrame(model_data)

return df

  

# Получение сравнения моделей

comparison_df = compare_models_in_project("Model Registry")

print("Model Comparison:")

print(comparison_df.to_string(index=False))

  

# Выбор лучшей модели

if 'test_accuracy' in comparison_df.columns:

best_model_row = comparison_df.loc[comparison_df['test_accuracy'].idxmax()]

print(f"\nBest model: {best_model_row['name']} (Accuracy: {best_model_row['test_accuracy']:.4f})")

# Пометка лучшей модели специальным тегом

best_model = Model(model_id=best_model_row['id'])

current_tags = best_model.tags or []

if 'production-candidate' not in current_tags:

best_model.set_model_tags(current_tags + ['production-candidate'])

print("Model tagged as production candidate")

```

  

## 8. Автоматизация и пайплайны {#пайплайны}

  

### Создание простого пайплайна

  

```python

from clearml import Task, Dataset

from clearml.automation import PipelineController

import pandas as pd

import numpy as np

from sklearn.model_selection import train_test_split

from sklearn.ensemble import RandomForestClassifier

from sklearn.metrics import accuracy_score

import joblib

  

def create_ml_pipeline():

"""Создает пайплайн машинного обучения"""

# Инициализация контроллера пайплайна

pipe = PipelineController(

name="ML Pipeline Example",

project="Pipelines",

version="1.0",

description="End-to-end ML pipeline: Data → Train → Evaluate"

)

# Шаг 1: Подготовка данных

pipe.add_step(

name="data_preparation",

base_task_project="Pipeline Steps",

base_task_name="Data Preparation Template",

parameter_override={

"General/dataset_size": 10000,

"General/test_size": 0.2,

"General/random_state": 42

}

)

# Шаг 2: Обучение модели

pipe.add_step(

name="model_training",

parents=["data_preparation"],

base_task_project="Pipeline Steps",

base_task_name="Model Training Template",

parameter_override={

"General/n_estimators": 100,

"General/max_depth": 10,

"General/random_state": 42

}

)

# Шаг 3: Оценка модели

pipe.add_step(

name="model_evaluation",

parents=["model_training"],

base_task_project="Pipeline Steps",

base_task_name="Model Evaluation Template",

parameter_override={

"General/threshold": 0.8

}

)

# Запуск пайплайна

pipe.start()

print("Pipeline started!")

print(f"Pipeline ID: {pipe.id}")

return pipe

  

# Создание шаблонных задач для пайплайна

  

def create_data_preparation_template():

"""Создает шаблон для подготовки данных"""

task = Task.init(

project_name="Pipeline Steps",

task_name="Data Preparation Template"

)

# Параметры по умолчанию

params = {

"dataset_size": 1000,

"test_size": 0.2,

"random_state": 42,

"n_features": 20,

"n_classes": 2

}

task.connect(params)

# Генерация данных

from sklearn.datasets import make_classification

X, y = make_classification(

n_samples=params["dataset_size"],

n_features=params["n_features"],

n_classes=params["n_classes"],

random_state=params["random_state"]

)

# Разделение данных

X_train, X_test, y_train, y_test = train_test_split(

X, y,

test_size=params["test_size"],

random_state=params["random_state"]

)

# Сохранение данных

np.save("X_train.npy", X_train)

np.save("X_test.npy", X_test)

np.save("y_train.npy", y_train)

np.save("y_test.npy", y_test)

# Загрузка как артефакты

task.upload_artifact("X_train", artifact_object=X_train)

task.upload_artifact("X_test", artifact_object=X_test)

task.upload_artifact("y_train", artifact_object=y_train)

task.upload_artifact("y_test", artifact_object=y_test)

# Логирование статистики

logger = task.get_logger()

logger.report_scalar("Data", "Train Size", value=len(X_train), iteration=0)

logger.report_scalar("Data", "Test Size", value=len(X_test), iteration=0)

logger.report_scalar("Data", "Features", value=X.shape[1], iteration=0)

print(f"Data prepared: {len(X_train)} train, {len(X_test)} test samples")

return task

  

def create_training_template():

"""Создает шаблон для обучения модели"""

task = Task.init(

project_name="Pipeline Steps",

task_name="Model Training Template"

)

# Параметры модели

params = {

"n_estimators": 100,

"max_depth": 10,

"random_state": 42

}

task.connect(params)

# Получение данных от предыдущего шага

# В реальном пайплайне эти данные будут переданы автоматически

parent_task = Task.get_task(task_id="data_preparation_task_id") # В реальности ID будет передан

X_train = parent_task.artifacts["X_train"].get()

y_train = parent_task.artifacts["y_train"].get()

# Обучение модели

model = RandomForestClassifier(

n_estimators=params["n_estimators"],

max_depth=params["max_depth"],

random_state=params["random_state"]

)

model.fit(X_train, y_train)

# Сохранение модели

model_filename = "trained_model.pkl"

joblib.dump(model, model_filename)

# Загрузка модели как артефакт

task.upload_artifact("trained_model", artifact_object=model_filename)

# Логирование параметров модели

logger = task.get_logger()

logger.report_scalar("Model", "N Estimators", value=params["n_estimators"], iteration=0)

logger.report_scalar("Model", "Max Depth", value=params["max_depth"], iteration=0)

print("Model training completed")

return task

  

def create_evaluation_template():

"""Создает шаблон для оценки модели"""

task = Task.init(

project_name="Pipeline Steps",

task_name="Model Evaluation Template"

)

# Параметры оценки

params = {

"threshold": 0.8

}

task.connect(params)

# Получение данных и модели от предыдущих шагов

data_task = Task.get_task(task_id="data_preparation_task_id")

model_task = Task.get_task(task_id="model_training_task_id")

X_test = data_task.artifacts["X_test"].get()

y_test = data_task.artifacts["y_test"].get()

# Загрузка модели

model_path = model_task.artifacts["trained_model"].get_local_copy()

model = joblib.load(model_path)

# Предсказания

y_pred = model.predict(X_test)

accuracy = accuracy_score(y_test, y_pred)

# Логирование результатов

logger = task.get_logger()

logger.report_scalar("Performance", "Accuracy", value=accuracy, iteration=0)

# Проверка порога

meets_threshold = accuracy >= params["threshold"]

logger.report_scalar("Validation", "Meets Threshold", value=int(meets_threshold), iteration=0)

if meets_threshold:

print(f"✅ Model passed validation! Accuracy: {accuracy:.4f} >= {params['threshold']}")

# Можно добавить логику для деплоя модели

else:

print(f"❌ Model failed validation. Accuracy: {accuracy:.4f} < {params['threshold']}")

return task

  

# Создание шаблонов (выполнить один раз)

print("Creating pipeline templates...")

# create_data_preparation_template()

# create_training_template()

# create_evaluation_template()

  

# Создание и запуск пайплайна

# pipeline = create_ml_pipeline()

```

  

### Условные пайплайны и параллельное выполнение

  

```python

from clearml.automation import PipelineController

  

def create_advanced_pipeline():

"""Создает продвинутый пайплайн с условной логикой и параллельным выполнением"""

pipe = PipelineController(

name="Advanced ML Pipeline",

project="Advanced Pipelines",

version="2.0"

)

# Шаг 1: Подготовка данных

pipe.add_step(

name="data_preparation",

base_task_project="Pipeline Steps",

base_task_name="Data Preparation Template"

)

# Параллельное обучение разных моделей

models = [

{"name": "random_forest", "type": "RandomForest", "n_estimators": 100},

{"name": "gradient_boosting", "type": "GradientBoosting", "n_estimators": 100},

{"name": "svm", "type": "SVM", "C": 1.0}

]

model_steps = []

for model_config in models:

step_name = f"train_{model_config['name']}"

pipe.add_step(

name=step_name,

parents=["data_preparation"],

base_task_project="Pipeline Steps",

base_task_name="Multi Model Training Template",

parameter_override={

"General/model_type": model_config["type"],

**{f"General/{k}": v for k, v in model_config.items() if k != "name"}

}

)

model_steps.append(step_name)

# Сравнение моделей

pipe.add_step(

name="model_comparison",

parents=model_steps,

base_task_project="Pipeline Steps",

base_task_name="Model Comparison Template"

)

# Условный деплой лучшей модели

pipe.add_step(

name="conditional_deploy",

parents=["model_comparison"],

base_task_project="Pipeline Steps",

base_task_name="Conditional Deploy Template",

parameter_override={

"General/min_accuracy_threshold": 0.85,

"General/deploy_environment": "staging"

}

)

# Запуск пайплайна

pipe.start()

return pipe

  

# Шаблон для обучения множественных моделей

def create_multi_model_training_template():

"""Шаблон для обучения разных типов моделей"""

task = Task.init(

project_name="Pipeline Steps",

task_name="Multi Model Training Template"

)

params = {

"model_type": "RandomForest",

"n_estimators": 100,

"max_depth": 10,

"C": 1.0, # для SVM

"random_state": 42

}

task.connect(params)

# Получение данных

# (В реальном пайплайне данные передаются автоматически)

# Выбор модели на основе параметра

if params["model_type"] == "RandomForest":

from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(

n_estimators=params["n_estimators"],

max_depth=params["max_depth"],

random_state=params["random_state"]

)

elif params["model_type"] == "GradientBoosting":

from sklearn.ensemble import GradientBoostingClassifier

model = GradientBoostingClassifier(

n_estimators=params["n_estimators"],

random_state=params["random_state"]

)

elif params["model_type"] == "SVM":

from sklearn.svm import SVC

model = SVC(

C=params["C"],

probability=True,

random_state=params["random_state"]

)

else:

raise ValueError(f"Unknown model type: {params['model_type']}")

# Обучение и сохранение модели

# (код обучения)

return task

```

  

### Мониторинг и уведомления в пайплайнах

  

```python

from clearml.automation import PipelineController

import smtplib

from email.mime.text import MIMEText

  

def create_monitored_pipeline():

"""Пайплайн с мониторингом и уведомлениями"""

pipe = PipelineController(

name="Monitored ML Pipeline",

project="Production Pipelines",

version="1.0"

)

# Добавление хуков для мониторинга

def on_step_completed(pipeline_controller, node, parameters):

"""Колбэк при завершении шага"""

print(f"Step '{node.name}' completed successfully")

# Логирование времени выполнения

if hasattr(node, 'executed_task'):

task = node.executed_task

logger = task.get_logger()

logger.report_scalar("Pipeline", "Step Duration",

value=task.get_runtime_properties().get('duration', 0),

iteration=0)

def on_step_failed(pipeline_controller, node, parameters):

"""Колбэк при ошибке в шаге"""

print(f"Step '{node.name}' failed!")

# Отправка уведомления

send_notification(

subject=f"Pipeline Failed: {node.name}",

message=f"Step {node.name} in pipeline {pipeline_controller.name} failed."

)

# Регистрация колбэков

pipe.set_default_execution_queue("default")

# Добавление шагов пайплайна

pipe.add_step(

name="data_validation",

base_task_project="Pipeline Steps",

base_task_name="Data Validation Template"

)

pipe.add_step(

name="model_training",

parents=["data_validation"],

base_task_project="Pipeline Steps",

base_task_name="Robust Training Template"

)

pipe.add_step(

name="model_validation",

parents=["model_training"],

base_task_project="Pipeline Steps",

base_task_name="Model Validation Template"

)

pipe.add_step(

name="performance_monitoring",

parents=["model_validation"],

base_task_project="Pipeline Steps",

base_task_name="Performance Monitor Template"

)

return pipe

  

def send_notification(subject, message, recipients=None):

"""Отправка email уведомлений"""

if recipients is None:

recipients = ["ml-team@company.com"]

# Настройки SMTP (замените на ваши)

smtp_server = "smtp.company.com"

smtp_port = 587

smtp_username = "pipeline@company.com"

smtp_password = "password"

try:

msg = MIMEText(message)

msg['Subject'] = subject

msg['From'] = smtp_username

msg['To'] = ', '.join(recipients)

server = smtplib.SMTP(smtp_server, smtp_port)

server.starttls()

server.login(smtp_username, smtp_password)

server.send_message(msg)

server.quit()

print(f"Notification sent: {subject}")

except Exception as e:

print(f"Failed to send notification: {e}")

  

# Шаблон с проверкой качества данных

def create_data_validation_template():

"""Шаблон для валидации данных"""

task = Task.init(

project_name="Pipeline Steps",

task_name="Data Validation Template"

)

params = {

"min_samples": 1000,

"max_missing_ratio": 0.1,

"required_features": ["feature1", "feature2", "target"]

}

task.connect(params)

logger = task.get_logger()

# Загрузка данных (пример)

# data = load_data()

# Проверки качества данных

validation_results = {

"sample_count_ok": True, # len(data) >= params["min_samples"]

"missing_data_ok": True, # missing_ratio <= params["max_missing_ratio"]

"features_ok": True, # all required features present

}

# Логирование результатов валидации

for check, passed in validation_results.items():

logger.report_scalar("Data Validation", check, value=int(passed), iteration=0)

# Общий результат валидации

all_passed = all(validation_results.values())

logger.report_scalar("Data Validation", "Overall", value=int(all_passed), iteration=0)

if not all_passed:

raise ValueError("Data validation failed!")

print("Data validation passed!")

return task

```

  

## 9. Практические примеры {#примеры}

  

### Полный пример: Computer Vision проект

  

```python

from clearml import Task, Dataset, Logger

import torch

import torch.nn as nn

import torch.optim as optim

from torch.utils.data import DataLoader

import torchvision

import torchvision.transforms as transforms

import matplotlib.pyplot as plt

import numpy as np

from sklearn.metrics import classification_report, confusion_matrix

import seaborn as sns

  

# Инициализация проекта

task = Task.init(

project_name="Computer Vision Project",

task_name="CIFAR-10 Classification with CNN",

tags=["computer-vision", "cnn", "cifar10", "pytorch"]

)

  

# Конфигурация эксперимента

config = {

# Параметры данных

'batch_size': 128,

'num_workers': 4,

'download_data': True,

# Параметры модели

'num_classes': 10,

'dropout_rate': 0.5,

# Параметры обучения

'learning_rate': 0.001,

'momentum': 0.9,

'weight_decay': 1e-4,

'epochs': 20,

# Другие параметры

'device': 'cuda' if torch.cuda.is_available() else 'cpu',

'save_every': 5,

'log_every': 100

}

  

# Подключение конфигурации к задаче

task.connect(config)

logger = task.get_logger()

  

# Классы CIFAR-10

classes = ('plane', 'car', 'bird', 'cat', 'deer', 'dog', 'frog', 'horse', 'ship', 'truck')

  

# Трансформации данных

transform_train = transforms.Compose([

transforms.RandomCrop(32, padding=4),

transforms.RandomHorizontalFlip(),

transforms.ToTensor(),

transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010)),

])

  

transform_test = transforms.Compose([

transforms.ToTensor(),

transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010)),

])

  

# Загрузка данных

print("Loading CIFAR-10 dataset...")

trainset = torchvision.datasets.CIFAR10(

root='./data', train=True, download=config['download_data'], transform=transform_train

)

trainloader = DataLoader(

trainset, batch_size=config['batch_size'], shuffle=True, num_workers=config['num_workers']

)

  

testset = torchvision.datasets.CIFAR10(

root='./data', train=False, download=config['download_data'], transform=transform_test

)

testloader = DataLoader(

testset, batch_size=config['batch_size'], shuffle=False, num_workers=config['num_workers']

)

  

# Логирование информации о данных

logger.report_scalar("Dataset", "Train Size", value=len(trainset), iteration=0)

logger.report_scalar("Dataset", "Test Size", value=len(testset), iteration=0)

logger.report_scalar("Dataset", "Num Classes", value=config['num_classes'], iteration=0)

  

# Визуализация примеров данных

def show_sample_images():

dataiter = iter(trainloader)

images, labels = next(dataiter)

# Денормализация для визуализации

def denormalize(tensor):

mean = torch.tensor([0.4914, 0.4822, 0.4465]).view(3, 1, 1)

std = torch.tensor([0.2023, 0.1994, 0.2010]).view(3, 1, 1)

return tensor * std + mean

fig, axes = plt.subplots(2, 8, figsize=(15, 4))

axes = axes.ravel()

for idx in range(16):

img = denormalize(images[idx])

img = torch.clamp(img, 0, 1)

axes[idx].imshow(np.transpose(img.numpy(), (1, 2, 0)))

axes[idx].set_title(f'{classes[labels[idx]]}')

axes[idx].axis('off')

plt.tight_layout()

logger.report_matplotlib_figure("Data Samples", "Training Examples", figure=fig, iteration=0)

plt.close()

  

show_sample_images()

  

# Определение модели CNN

class CIFAR10CNN(nn.Module):

def __init__(self, num_classes=10, dropout_rate=0.5):

super(CIFAR10CNN, self).__init__()

# Первый блок свертки

self.conv1 = nn.Conv2d(3, 32, 3, padding=1)

self.bn1 = nn.BatchNorm2d(32)

self.conv2 = nn.Conv2d(32, 32, 3, padding=1)

self.bn2 = nn.BatchNorm2d(32)

self.pool1 = nn.MaxPool2d(2, 2)

self.dropout1 = nn.Dropout2d(0.25)

# Второй блок свертки

self.conv3 = nn.Conv2d(32, 64, 3, padding=1)

self.bn3 = nn.BatchNorm2d(64)

self.conv4 = nn.Conv2d(64, 64, 3, padding=1)

self.bn4 = nn.BatchNorm2d(64)

self.pool2 = nn.MaxPool2d(2, 2)

self.dropout2 = nn.Dropout2d(0.25)

# Третий блок свертки

self.conv5 = nn.Conv2d(64, 128, 3, padding=1)

self.bn5 = nn.BatchNorm2d(128)

self.conv6 = nn.Conv2d(128, 128, 3, padding=1)

self.bn6 = nn.BatchNorm2d(128)

self.pool3 = nn.MaxPool2d(2, 2)

self.dropout3 = nn.Dropout2d(0.25)

# Полносвязные слои

self.fc1 = nn.Linear(128 * 4 * 4, 512)

self.dropout4 = nn.Dropout(dropout_rate)

self.fc2 = nn.Linear(512, num_classes)

def forward(self, x):

# Первый блок

x = torch.relu(self.bn1(self.conv1(x)))

x = torch.relu(self.bn2(self.conv2(x)))

x = self.pool1(x)

x = self.dropout1(x)

# Второй блок

x = torch.relu(self.bn3(self.conv3(x)))

x = torch.relu(self.bn4(self.conv4(x)))

x = self.pool2(x)

x = self.dropout2(x)

# Третий блок

x = torch.relu(self.bn5(self.conv5(x)))

x = torch.relu(self.bn6(self.conv6(x)))

x = self.pool3(x)

x = self.dropout3(x)

# Полносвязные слои

x = x.view(-1, 128 * 4 * 4)

x = torch.relu(self.fc1(x))

x = self.dropout4(x)

x = self.fc2(x)

return x

  

# Создание модели

device = torch.device(config['device'])

model = CIFAR10CNN(num_classes=config['num_classes'], dropout_rate=config['dropout_rate'])

model.to(device)

  

# Подсчет параметров модели

total_params = sum(p.numel() for p in model.parameters())

trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)

  

logger.report_scalar("Model", "Total Parameters", value=total_params, iteration=0)

logger.report_scalar("Model", "Trainable Parameters", value=trainable_params, iteration=0)

  

print(f"Model created with {trainable_params:,} trainable parameters")

  

# Определение функции потерь и оптимизатора

criterion = nn.CrossEntropyLoss()

optimizer = optim.SGD(

model.parameters(),

lr=config['learning_rate'],

momentum=config['momentum'],

weight_decay=config['weight_decay']

)

  

# Планировщик обучения

scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=10, gamma=0.1)

  

# Функция для оценки модели

def evaluate_model(model, dataloader, criterion, device):

model.eval()

total_loss = 0.0

correct = 0

total = 0

all_predictions = []

all_labels = []

with torch.no_grad():

for data, target in dataloader:

data, target = data.to(device), target.to(device)

output = model(data)

loss = criterion(output, target)

total_loss += loss.item()

_, predicted = torch.max(output.data, 1)

total += target.size(0)

correct += (predicted == target).sum().item()

all_predictions.extend(predicted.cpu().numpy())

all_labels.extend(target.cpu().numpy())

avg_loss = total_loss / len(dataloader)

accuracy = 100 * correct / total

return avg_loss, accuracy, all_predictions, all_labels

  

# Обучение модели

print("Starting training...")

best_accuracy = 0.0

train_losses = []

train_accuracies = []

val_losses = []

val_accuracies = []

  

for epoch in range(config['epochs']):

model.train()

running_loss = 0.0

correct = 0

total = 0

for batch_idx, (data, target) in enumerate(trainloader):

data, target = data.to(device), target.to(device)

optimizer.zero_grad()

output = model(data)

loss = criterion(output, target)

loss.backward()

optimizer.step()

running_loss += loss.item()

_, predicted = torch.max(output.data, 1)

total += target.size(0)

correct += (predicted == target).sum().item()

# Логирование каждые N батчей

if batch_idx % config['log_every'] == 0:

iteration = epoch * len(trainloader) + batch_idx

logger.report_scalar("Loss", "Training Batch", iteration=iteration, value=loss.item())

current_acc = 100 * correct / total if total > 0 else 0

logger.report_scalar("Accuracy", "Training Batch", iteration=iteration, value=current_acc)

# Статистика эпохи

epoch_train_loss = running_loss / len(trainloader)

epoch_train_acc = 100 * correct / total

# Валидация

val_loss, val_acc, _, _ = evaluate_model(model, testloader, criterion, device)

# Сохранение метрик

train_losses.append(epoch_train_loss)

train_accuracies.append(epoch_train_acc)

val_losses.append(val_loss)

val_accuracies.append(val_acc)

# Логирование эпохи

logger.report_scalar("Loss", "Training Epoch", iteration=epoch, value=epoch_train_loss)

logger.report_scalar("Loss", "Validation Epoch", iteration=epoch, value=val_loss)

logger.report_scalar("Accuracy", "Training Epoch", iteration=epoch, value=epoch_train_acc)

logger.report_scalar("Accuracy", "Validation Epoch", iteration=epoch, value=val_acc)

logger.report_scalar("Learning Rate", "Current", iteration=epoch, value=optimizer.param_groups[0]['lr'])

# Сохранение лучшей модели

if val_acc > best_accuracy:

best_accuracy = val_acc

torch.save(model.state_dict(), 'best_model.pth')

logger.report_scalar("Best", "Validation Accuracy", iteration=epoch, value=best_accuracy)

# Обновление планировщика

scheduler.step()

print(f'Epoch {epoch+1}/{config["epochs"]}: '

f'Train Loss: {epoch_train_loss:.4f}, Train Acc: {epoch_train_acc:.2f}%, '

f'Val Loss: {val_loss:.4f}, Val Acc: {val_acc:.2f}%')

  

# Финальная оценка

print("Final evaluation...")

model.load_state_dict(torch.load('best_model.pth'))

final_loss, final_acc, predictions, true_labels = evaluate_model(model, testloader, criterion, device)

  

logger.report_scalar("Final", "Test Loss", value=final_loss, iteration=0)

logger.report_scalar("Final", "Test Accuracy", value=final_acc, iteration=0)

  

# Матрица ошибок

cm = confusion_matrix(true_labels, predictions)

plt.figure(figsize=(10, 8))

sns.heatmap(cm, annot=True, fmt='d', cmap='Blues',

xticklabels=classes, yticklabels=classes)

plt.title('Confusion Matrix')

plt.ylabel('True Label')

plt.xlabel('Predicted Label')

logger.report_matplotlib_figure("Results", "Confusion Matrix", figure=plt.gcf(), iteration=0)

plt.close()

  

# Отчет по классификации

classification_rep = classification_report(true_labels, predictions,

target_names=classes, output_dict=True)

  

# Логирование метрик по классам

for class_name in classes:

if class_name in classification_rep:

metrics = classification_rep[class_name]

logger.report_scalar(f"Class Metrics/{class_name}", "Precision",

value=metrics['precision'], iteration=0)

logger.report_scalar(f"Class Metrics/{class_name}", "Recall",

value=metrics['recall'], iteration=0)

logger.report_scalar(f"Class Metrics/{class_name}", "F1-Score",

value=metrics['f1-score'], iteration=0)

  

# График обучения

fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(15, 10))

  

# Потери

ax1.plot(train_losses, label='Training Loss', color='blue')

ax1.plot(val_losses, label='Validation Loss', color='red')

ax1.set_title('Loss over Epochs')

ax1.set_xlabel('Epoch')

ax1.set_ylabel('Loss')

ax1.legend()

ax1.grid(True)

  

# Точность

ax2.plot(train_accuracies, label='Training Accuracy', color='blue')

ax2.plot(val_accuracies, label='Validation Accuracy', color='red')

ax2.set_title('Accuracy over Epochs')

ax2.set_xlabel('Epoch')

ax2.set_ylabel('Accuracy (%)')

ax2.legend()

ax2.grid(True)

  

# Точность по классам

class_accuracies = []

for class_idx, class_name in enumerate(classes):

class_mask = np.array(true_labels) == class_idx

if np.sum(class_mask) > 0:

class_acc = np.mean(np.array(predictions)[class_mask] == class_idx) * 100

class_accuracies.append(class_acc)

else:

class_accuracies.append(0)

  

ax3.bar(classes, class_accuracies, color='skyblue')

ax3.set_title('Accuracy by Class')

ax3.set_xlabel('Class')

ax3.set_ylabel('Accuracy (%)')

ax3.tick_params(axis='x', rotation=45)

  

# Распределение предсказаний

ax4.hist(predictions, bins=len(classes), alpha=0.7, label='Predictions', color='lightcoral')

ax4.hist(true_labels, bins=len(classes), alpha=0.7, label='True Labels', color='lightblue')

ax4.set_title('Distribution of Predictions vs True Labels')

ax4.set_xlabel('Class')

ax4.set_ylabel('Count')

ax4.legend()

  

plt.tight_layout()

logger.report_matplotlib_figure("Training", "Complete Analysis", figure=fig, iteration=0)

plt.close()

  

print(f"\nTraining completed!")

print(f"Best validation accuracy: {best_accuracy:.2f}%")

print(f"Final test accuracy: {final_acc:.2f}%")

  

# Сохранение финальной модели

task.upload_artifact('final_model', artifact_object='best_model.pth')

print("Model and artifacts uploaded to ClearML!")

```

  

### Пример: NLP проект с трансформерами

  

```python

from clearml import Task

import torch

from transformers import (

AutoTokenizer, AutoModelForSequenceClassification,

TrainingArguments, Trainer, DataCollatorWithPadding

)

from datasets import Dataset

import pandas as pd

import numpy as np

from sklearn.metrics import accuracy_score, precision_recall_fscore_support

import matplotlib.pyplot as plt

import seaborn as sns

  

# Инициализация задачи

task = Task.init(

project_name="NLP Projects",

task_name="Sentiment Analysis with BERT",

tags=["nlp", "transformers", "bert", "sentiment-analysis"]

)

  

# Конфигурация

config = {

'model_name': 'bert-base-uncased',

'num_labels': 3, # положительный, нейтральный, отрицательный

'max_length': 512,

'batch_size': 16,

'learning_rate': 2e-5,

'num_epochs': 3,

'warmup_steps': 500,

'weight_decay': 0.01,

'seed': 42

}

  

task.connect(config)

logger = task.get_logger()

  

# Установка seed для воспроизводимости

torch.manual_seed(config['seed'])

np.random.seed(config['seed'])

  

# Создание синтетических данных для примера

def create_sample_data():

"""Создает примерные данные для анализа настроений"""

positive_texts = [

"I love this product! It's amazing!",

"Great service and excellent quality.",

"Highly recommended, fantastic experience!",

"Best purchase I've made this year.",

"Outstanding customer support and fast delivery."

] * 200

negative_texts = [

"Terrible product, waste of money.",

"Poor quality and bad customer service.",

"I regret buying this, very disappointed.",

"Completely useless, doesn't work at all.",

"Worst experience ever, avoid at all costs."

] * 200

neutral_texts = [

"The product is okay, nothing special.",

"Average quality, meets basic expectations.",

"It's fine, does what it's supposed to do.",

"Decent product for the price.",

"Neither good nor bad, just average."

] * 200

# Объединение данных

texts = positive_texts + negative_texts + neutral_texts

labels = [2] * len(positive_texts) + [0] * len(negative_texts) + [1] * len(neutral_texts)

# Перемешивание

data = list(zip(texts, labels))

np.random.shuffle(data)

texts, labels = zip(*data)

return list(texts), list(labels)

  

# Создание данных

texts, labels = create_sample_data()

  

# Разделение на train/validation/test

train_size = int(0.7 * len(texts))

val_size = int(0.15 * len(texts))

  

train_texts = texts[:train_size]

train_labels = labels[:train_size]

val_texts = texts[train_size:train_size + val_size]

val_labels = labels[train_size:train_size + val_size]

test_texts = texts[train_size + val_size:]

test_labels = labels[train_size + val_size:]

  

# Логирование статистики данных

logger.report_scalar("Dataset", "Train Size", value=len(train_texts), iteration=0)

logger.report_scalar("Dataset", "Validation Size", value=len(val_texts), iteration=0)

logger.report_scalar("Dataset", "Test Size", value=len(test_texts), iteration=0)

  

# Распределение классов

label_names = ['Negative', 'Neutral', 'Positive']

train_label_counts = pd.Series(train_labels).value_counts().sort_index()

  

plt.figure(figsize=(10, 6))

plt.subplot(1, 2, 1)

plt.bar(label_names, train_label_counts.values, color=['red',