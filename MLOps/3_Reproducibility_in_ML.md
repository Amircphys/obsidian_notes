

![[Pasted image 20241029105406.png]]Проблемы воспроизводимости:
 - Не могу воспроизвести обучение модели от начала до конца
 - Не могу воспроизвести любого сделанного эксперимента

Знания о сделанных экспериментах через год:
 - Постановка задачи - в джире может быть что-то есть
 - Работа с  данными - никто не знает, на каких данных обучалась модель 
 - Обучение модели - никто не знает, какие модели пробовал обучать
 - Выбор лучшей  - есть модель, никто не знает почему выбрали именно ее

Рецепт ML модели:
	 **Data** + **Code** + **Зависимости**


Почему плохо коммитить большие файлы в GIT:
 - При выполнении операции clone будут выкачиваться все версии всех
больших файлов со всех коммитов всех веток(и это будет работать
медленно)
- git не сможет оптимизировать хранение таких файлов (pack-файлы)

Где же все таки их хранить?
 - hadoop hdfs
 - Amazon S3
 - google cloud storage
 - ...

**Чек-лист воспроизводимости**
 - Знаем ли мы на каких данных была обучена  наша модель(можем ли их получить)?
 - Знаем ли какой версией кода была обучена модель?
 - Какие параметры передавались на вход модели?
 - Насколько хорошо была обучена модель?
 - Можем ли повторить обучение?
 - Есть ли связь между моделью на продакшеном и кодом, данными, параметрами, оффлайн оценками?

**ML metadata store**
Вместе с сериализованным файлом модели автоматически сохраняем:
	- ссылку на данные
	- ссылку на git repo и коммит в нем(либо релизной версии кода)
	- параметры
	- метрики
Называем папку modelname_v1, где-нибудь в вике указываем ссылку на все это дело:
- Какой пайплайн исполнялся
- Кто его исполнял
- Какие параметры были даны на вход
- Какие метрики и артефакты были на выходе
- input, output для каждой из стадий


# DVC
DVC (Data Version Control) - это система версионирования датасетов и не только, которая является надстройкой над git. 
Если передать под контроль DVC какие-то данные, то он начнет отслеживать все изменения. А мы можем работать с этими данными точно так же, как с Git: сохранять версию, отправлять в удалённый репозиторий, получать нужную версию данных, изменять и переключаться между версиями
![[Pasted image 20241029114642.png]]
dvc remote:
- Версии файлов хранятся локально, есть возможность делать checkout
- Есть возможность синхронизировать cache с удаленным специализированным хранилищем(dvc pull, dvc push)

Для каждого файла, которые версионируется в dvc есть файл `file_name.расширение.dvc` в котором указаны информация - хеш файла, ссылка откуда стягивают файл и тд



# MLFLOW

 MLFlow - это инструмент для управления всеми стадиями жизненного цикла модели машинного обучения. MLflow - это инструмент с открытым исходным кодом для разработки, сопровождения и взаимодействия на каждом этапе жизненного цикла ML-модели. Более того, MLflow является инструментом, не зависящим от фреймворка, поэтому любой ML/DL-фреймворк может быстро адаптироваться к экосистеме, которую предлагает MLflow.
 MLflow выступает в качестве платформы, предлагающей инструменты для отслеживания метрик, артефактов и метаданных. Она также предоставляет стандартные форматы для упаковки, распространения и развертывания моделей и проектов.
 MLflow также предлагает инструменты для управления версиями модели. Эти инструменты инкапсулированы в ее четырех основных компонентах:
- MLflow Tracking - это инструмент на основе API для регистрации метрик, параметров, версий моделей, кода и файлов.

- MLflow Models - позволяют упаковывать модели машинного обучения в стандартный формат для непосредственного использования через различные сервисы, такие как REST API, Microsoft Azure ML, Amazon SageMaker или Apache Spark. Для упаковки MLflow создает каталог с двумя файлами - моделью и файлом, в котором указаны детали упаковки и загрузки модели. Одним из преимуществ стандарта MLflow Models является то, что упаковка может быть осуществлена для нескольких языков (multi-language) или вариантов (вкусов) (multi-flavor).

- MLflow Projects - предоставляет стандартный формат для упаковки, обмена и повторного использования проектов машинного обучения. В отличие от MLflow Models, MLflow Projects ориентирован на удобство переноса и дистрибуции проектов машинного обучения. Проект MLflow определяется манифестом YAML под названием `MLProject`, в котором раскрываются спецификации проекта.





# DVC (Data Version Control) - Подробный конспект

## 1. Что такое DVC и зачем он нужен?

### 1.1 Определение
DVC (Data Version Control) — это инструмент с открытым исходным кодом для версионирования данных, моделей машинного обучения и управления ML-пайплайнами. DVC работает поверх Git и расширяет его возможности для работы с большими файлами и сложными ML-проектами.
### 1.2 Основные проблемы, которые решает DVC
**Проблема 1: Git плохо работает с большими файлами**

```bash
# Попытка добавить большой датасет в Git
git add large_dataset.csv # 2GB файл
git commit -m "Add dataset"
# Результат: медленная работа, большой размер репозитория
```


**Проблема 2: Невозможность отследить изменения в данных**
- Какая версия модели обучена на каких данных?
- Как воспроизвести эксперiment с точно теми же данными?
- Кто и когда изменил датасет?
  
**Проблема 3: Сложность совместной работы с данными**
- Как поделиться 10GB датасетом с командой?
- Как синхронизировать изменения в данных между разработчиками?

### 1.3 Как DVC решает эти проблемы
DVC создает легковесные метафайлы (`.dvc` файлы), которые хранятся в Git, а сами данные хранятся в удаленном хранилище (S3, Google Drive, SSH и др.).

## 2. Архитектура и принципы работы DVC
### 2.1 Основные компоненты  

```graph

┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐

│ Git Repo │ │ DVC Cache │ │ Remote Storage │

│ │ │ │ │ │

│ ├── .dvc/ │◄──►│ ├── files/ │◄──►│ ├── AWS S3 │

│ ├── data.dvc │ │ ├── tmp/ │ │ ├── Google Cloud│

│ ├── model.dvc │ │ └── cache/ │ │ ├── Azure Blob │

│ └── dvc.yaml │ │ │ │ └── SSH/Local │

└─────────────────┘ └─────────────────┘ └─────────────────┘

```

  

### 2.2 Принцип работы

1. **Хеширование**: DVC вычисляет MD5-хеш файлов
2. **Кеширование**: Файлы хранятся в локальном кеше
3. **Метаданные**: Создаются `.dvc` файлы с метаинформацией
4. **Синхронизация**: Данные синхронизируются с удаленным хранилищем


## 3. Установка и настройка DVC

### 3.1 Установка

```bash

# Установка через pip
pip install dvc
# Установка с поддержкой S3
pip install dvc[s3]
# Установка с поддержкой всех облачных провайдеров
pip install dvc[all]
# Проверка установки
dvc version
```


### 3.2 Инициализация проекта


```bash
# Создание нового проекта
mkdir my-ml-project
cd my-ml-project
# Инициализация Git
git init
# Инициализация DVC
dvc init
# Структура после инициализации
tree -a
.

├── .dvc

│ ├── .gitignore

│ └── config

├── .dvcignore

└── .git/

```

  

### 3.3 Настройка удаленного хранилища


```bash

# AWS S3
dvc remote add -d myremote s3://my-bucket/dvc-storage
# Google Cloud Storage
dvc remote add -d myremote gs://my-bucket/dvc-storage
# Azure Blob Storage
dvc remote add -d myremote azure://my-container/dvc-storage
# Локальная папка (для тестирования)
dvc remote add -d myremote /path/to/remote/storage
# SSH
dvc remote add -d myremote ssh://user@example.com/path/to/remote/storage
# Проверка конфигурации
dvc remote list

```

## 4. Основные операции с DVC
### 4.1 Добавление данных под версионный контроль


```bash
# Создадим тестовый датасет
mkdir data
echo "sample,label\n1,A\n2,B\n3,C" > data/train.csv
echo "sample,label\n4,D\n5,E" > data/test.csv
# Добавление файлов в DVC
dvc add data/train.csv
dvc add data/test.csv
# Или добавление целой папки
dvc add data/
# Структура после добавления
tree
.
├── data

│ ├── .gitignore # Автоматически создан DVC

│ ├── test.csv

│ ├── train.csv

│ └── train.csv.dvc # Метафайл DVC

└── .dvc/

```

### 4.2 Изучение .dvc файлов

```bash
# Содержимое train.csv.dvc
cat data/train.csv.dvc
```

  

```yaml
outs:
- md5: 5d8c8e51e0b8b4c9d7e8f1a2b3c4d5e6
size: 45
path: train.csv
```

### 4.3 Коммит изменений в Git

```bash
# Добавляем .dvc файлы в Git
git add data/train.csv.dvc data/test.csv.dvc data/.gitignore
git commit -m "Add training and test datasets"

```

### 4.4 Загрузка данных в удаленное хранилище

```bash
# Отправка данных в удаленное хранилище
dvc push
# Проверка статуса
dvc status
```

### 4.5 Получение данных из удаленного хранилища


```bash
# Удаляем локальные данные (симуляция новой машины)
rm -rf data/train.csv data/test.csv
# Получаем данные из удаленного хранилища
dvc pull
# Или получение конкретного файла
dvc pull data/train.csv.dvc
```

## 5. Работа с версиями данных

### 5.1 Изменение данных

```bash
# Изменяем данные
echo "sample,label\n1,A\n2,B\n3,C\n6,F" > data/train.csv
# Проверяем статус
dvc status
# Вывод: data/train.csv.dvc:
# changed outs:
# modified: data/train.csv
# Обновляем DVC файл
dvc add data/train.csv
# Коммитим изменения
git add data/train.csv.dvc
git commit -m "Add new training sample"
# Отправляем новую версию
dvc push
```
### 5.2 Переключение между версиями
```bash
# Смотрим историю коммитов
git log --oneline
# Переключаемся на предыдущую версию
git checkout HEAD~1
# Получаем соответствующую версию данных
dvc checkout
# Возвращаемся к последней версии
git checkout main
dvc checkout
```

### 5.3 Создание веток для экспериментов

```bash
# Создаем ветку для эксперимента
git checkout -b experiment-1
# Изменяем данные для эксперимента
echo "sample,label,feature\n1,A,0.1\n2,B,0.2\n3,C,0.3" > data/train.csv
# Версионируем изменения
dvc add data/train.csv
git add data/train.csv.dvc
git commit -m "Add feature column for experiment-1"
dvc push
# Переключаемся обратно на main
git checkout main
dvc checkout
```


## 6. DVC Pipelines - автоматизация ML-процессов

### 6.1 Создание простого пайплайна
Создадим файл `dvc.yaml`:

```yaml
stages:
prepare:
cmd: python src/prepare.py
deps:
- src/prepare.py
- data/raw/
outs:
- data/prepared/train.csv
- data/prepared/test.csv
train:
cmd: python src/train.py
deps:
- src/train.py
- data/prepared/train.csv
params:
- train.learning_rate
- train.epochs
outs:
- models/model.pkl
metrics:
- metrics/train_metrics.json
evaluate:
cmd: python src/evaluate.py
deps:
- src/evaluate.py
- models/model.pkl
- data/prepared/test.csv

outs:
- results/predictions.csv
metrics:
- metrics/eval_metrics.json

```


### 6.2 Создание скриптов для пайплайна

**src/prepare.py**:

```python
import pandas as pd
import os
from sklearn.model_selection import train_test_split

def prepare_data():
# Создаем папки если не существуют
	os.makedirs('data/prepared', exist_ok=True)
	# Загружаем сырые данные
	raw_data = pd.read_csv('data/raw/dataset.csv')
	# Простая предобработка
	processed_data = raw_data.dropna()
	processed_data['feature_scaled'] = (processed_data['feature'] - processed_data['feature'].mean()) / processed_data['feature'].std()	
	# Разделяем на train/test	
	train_data, test_data = train_test_split(processed_data, test_size=0.2, random_state=42)
	
	# Сохраняем
	train_data.to_csv('data/prepared/train.csv', index=False)
	test_data.to_csv('data/prepared/test.csv', index=False)
	print(f"Prepared {len(train_data)} training samples and {len(test_data)} test samples")


if __name__ == "__main__":
	prepare_data()

```

  

**src/train.py**:

```python

import pandas as pd

import pickle

import json

import os

import yaml

from sklearn.ensemble import RandomForestClassifier

from sklearn.metrics import accuracy_score

  

def load_params():

with open('params.yaml', 'r') as f:

params = yaml.safe_load(f)

return params['train']

  

def train_model():

# Загружаем параметры

params = load_params()

# Создаем папки

os.makedirs('models', exist_ok=True)

os.makedirs('metrics', exist_ok=True)

# Загружаем данные

train_data = pd.read_csv('data/prepared/train.csv')

X = train_data.drop(['label'], axis=1)

y = train_data['label']

# Обучаем модель

model = RandomForestClassifier(

n_estimators=params['n_estimators'],

max_depth=params['max_depth'],

random_state=42

)

model.fit(X, y)

# Сохраняем модель

with open('models/model.pkl', 'wb') as f:

pickle.dump(model, f)

# Считаем метрики на обучающей выборке

train_predictions = model.predict(X)

train_accuracy = accuracy_score(y, train_predictions)

# Сохраняем метрики

metrics = {

'train_accuracy': train_accuracy,

'n_estimators': params['n_estimators'],

'max_depth': params['max_depth']

}

with open('metrics/train_metrics.json', 'w') as f:

json.dump(metrics, f, indent=2)

print(f"Model trained with accuracy: {train_accuracy:.4f}")

  

if __name__ == "__main__":

train_model()

```

  

**params.yaml**:

```yaml

train:

n_estimators: 100

max_depth: 10

random_state: 42

```

  

### 6.3 Запуск пайплайна

  

```bash

# Запуск всего пайплайна

dvc repro

  

# Запуск конкретного этапа

dvc repro train

  

# Принудительный перезапуск

dvc repro --force

  

# Проверка статуса пайплайна

dvc status

```

  

### 6.4 Визуализация пайплайна

  

```bash

# Показать граф зависимостей

dvc dag

  

# Вывод:

# +─────────+

# │ prepare │

# +─────────+

# │

# │

# +─────────+

# │ train │

# +─────────+

# │

# │

# +─────────────+

# │ evaluate │

# +─────────────+

```

  

## 7. Работа с параметрами и метриками

  

### 7.1 Отслеживание параметров

  

```bash

# Изменение параметров

echo "train:

n_estimators: 200

max_depth: 15" > params.yaml

  

# DVC автоматически отследит изменения параметров

dvc repro

  

# Сравнение метрик

dvc metrics show

dvc metrics diff

```

  

### 7.2 Сравнение экспериментов

  

```bash

# Создание эксперимента с другими параметрами

dvc exp run --set-param train.n_estimators=50

  

# Просмотр всех экспериментов

dvc exp show

  

# Сравнение экспериментов

dvc exp diff

```

  

## 8. Удаленное хранилище - детальная настройка

  

### 8.1 Настройка AWS S3

  

```bash

# Добавление S3 remote

dvc remote add -d s3remote s3://my-dvc-bucket/project-data

  

# Настройка аутентификации

dvc remote modify s3remote access_key_id myaccesskey

dvc remote modify s3remote secret_access_key mysecretkey

  

# Или использование AWS профилей

dvc remote modify s3remote profile myprofile

  

# Настройка региона

dvc remote modify s3remote region us-west-2

  

# Шифрование

dvc remote modify s3remote sse AES256

```

  

### 8.2 Настройка Google Cloud Storage

  

```bash

# Добавление GCS remote

dvc remote add -d gcsremote gs://my-dvc-bucket/project-data

  

# Аутентификация через service account

dvc remote modify gcsremote credentialpath /path/to/service-account.json

  

# Или через gcloud

gcloud auth application-default login

```

  

### 8.3 Работа с несколькими remote

  

```bash

# Добавление нескольких remotes

dvc remote add s3backup s3://backup-bucket/data

dvc remote add local_backup /mnt/backup/dvc-data

  

# Установка приоритетов

dvc remote default s3remote

  

# Синхронизация с конкретным remote

dvc push -r s3backup

dvc pull -r local_backup

```

  

## 9. Расширенные возможности DVC

  

### 9.1 DVC Import - импорт данных из других проектов

  

```bash

# Импорт данных из другого DVC проекта

dvc import https://github.com/example/ml-project data/dataset.csv

  

# Импорт с автоматическим обновлением

dvc import --rev main https://github.com/example/ml-project data/dataset.csv

  

# Обновление импортированных данных

dvc update data/dataset.csv.dvc

```

  

### 9.2 DVC Get - получение файлов без версионирования

  

```bash

# Простое скачивание файла из DVC проекта

dvc get https://github.com/example/ml-project data/model.pkl

  

# Скачивание конкретной версии

dvc get --rev v1.0 https://github.com/example/ml-project data/model.pkl

```

  

### 9.3 Работа с большими датасетами

  

```bash

# Добавление папки с множеством файлов

dvc add data/images/ # Папка с 100,000 изображений

  

# DVC создаст единый .dvc файл для всей папки

cat data/images.dvc

```

  

```yaml

outs:

- md5: 68b329da9893e34099c7d8ad5cb9c940.dir

size: 1073741824

nfiles: 100000

path: images

```

  

### 9.4 Кеширование и оптимизация

  

```bash

# Проверка использования кеша

dvc cache dir

  

# Очистка кеша

dvc gc

  

# Принудительная очистка всего кеша

dvc gc --force

  

# Настройка типа кеширования (hardlink, copy, symlink)

dvc config cache.type hardlink

```

  

## 10. Интеграция с CI/CD

  

### 10.1 GitHub Actions пример

  

**.github/workflows/dvc.yml**:

```yaml

name: DVC Pipeline

on: [push, pull_request]

  

jobs:

run-pipeline:

runs-on: ubuntu-latest

steps:

- uses: actions/checkout@v2

- name: Setup Python

uses: actions/setup-python@v2

with:

python-version: '3.8'

- name: Install dependencies

run: |

pip install dvc[s3] pandas scikit-learn

- name: Configure AWS credentials

uses: aws-actions/configure-aws-credentials@v1

with:

aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}

aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

aws-region: us-west-2

- name: Pull data

run: dvc pull

- name: Run pipeline

run: dvc repro

- name: Push results

run: dvc push

- name: Show metrics

run: dvc metrics show

```

  

### 10.2 Continuous Training

  

```yaml

# В dvc.yaml добавляем автоматическую переобучение

stages:

check_data_drift:

cmd: python src/check_drift.py

deps:

- src/check_drift.py

- data/new_data/

- models/current_model.pkl

outs:

- reports/drift_report.json

retrain:

cmd: python src/retrain.py

deps:

- src/retrain.py

- data/prepared/

- reports/drift_report.json

outs:

- models/retrained_model.pkl

metrics:

- metrics/retrain_metrics.json

```

  

## 11. Мониторинг и отладка

  

### 11.1 Логирование и отладка

  

```bash

# Включение verbose режима

dvc pull -v

  

# Отладочный режим

dvc repro --verbose

  

# Проверка конфигурации

dvc config --list

  

# Диагностика

dvc doctor

```

  

### 11.2 Решение проблем

  

**Проблема**: Медленная синхронизация с S3

```bash

# Увеличение числа параллельных потоков

dvc remote modify myremote jobs 8

  

# Настройка размера chunk'ов

dvc remote modify myremote chunk_size 16777216

```

  

**Проблема**: Конфликты при слиянии

```bash

# При конфликте в .dvc файлах

git status

# Разрешаем конфликт в .dvc файле

# Затем:

dvc checkout

```

  

## 12. Лучшие практики

  

### 12.1 Структура проекта

  

```

my-ml-project/

├── data/

│ ├── raw/ # Сырые данные

│ ├── interim/ # Промежуточные данные

│ ├── processed/ # Обработанные данные

│ └── external/ # Внешние данные

├── models/ # Обученные модели

├── notebooks/ # Jupyter notebooks

├── src/ # Исходный код

│ ├── data/ # Скрипты обработки данных

│ ├── features/ # Feature engineering

│ ├── models/ # Модели

│ └── visualization/ # Визуализация

├── reports/ # Отчеты и метрики

├── requirements.txt

├── params.yaml # Параметры

├── dvc.yaml # DVC пайплайн

└── .dvc/ # DVC конфигурация

```

  

### 12.2 Соглашения об именовании

  

```bash

# Версионирование моделей

models/

├── model_v1.0.pkl

├── model_v1.1.pkl

└── production_model.pkl

  

# Версионирование данных

data/

├── train_2023_01.csv

├── train_2023_02.csv

└── test_latest.csv

```

  

### 12.3 Параметризация

  

```yaml

# params.yaml - хорошая структура

data:

train_path: data/processed/train.csv

test_path: data/processed/test.csv

target_column: label

  

preprocessing:

normalize: true

handle_missing: mean

feature_selection: true

  

model:

algorithm: random_forest

n_estimators: 100

max_depth: 10

random_state: 42

  

training:

validation_split: 0.2

early_stopping: true

patience: 10

```

  

## 13. Интеграция с MLflow

  

```python

# src/train_with_mlflow.py

import mlflow

import mlflow.sklearn

import pandas as pd

import yaml

from sklearn.ensemble import RandomForestClassifier

from sklearn.metrics import accuracy_score, f1_score

  

def train_with_mlflow():

# Загружаем параметры из DVC

with open('params.yaml', 'r') as f:

params = yaml.safe_load(f)

train_params = params['model']

with mlflow.start_run():

# Логируем параметры

mlflow.log_params(train_params)

# Загружаем данные (управляемые DVC)

train_data = pd.read_csv('data/processed/train.csv')

X = train_data.drop('target', axis=1)

y = train_data['target']

# Обучаем модель

model = RandomForestClassifier(**train_params)

model.fit(X, y)

# Считаем метрики

predictions = model.predict(X)

accuracy = accuracy_score(y, predictions)

f1 = f1_score(y, predictions, average='weighted')

# Логируем метрики

mlflow.log_metric('accuracy', accuracy)

mlflow.log_metric('f1_score', f1)

# Сохраняем модель

mlflow.sklearn.log_model(model, "model")

# Также сохраняем для DVC

import pickle

with open('models/model.pkl', 'wb') as f:

pickle.dump(model, f)

  

if __name__ == "__main__":

train_with_mlflow()

```

  

## 14. Заключение и следующие шаги

  

### 14.1 Что мы изучили

  

1. **Основы DVC**: версионирование данных и моделей

2. **Пайплайны**: автоматизация ML-процессов

3. **Удаленное хранилище**: работа с облачными провайдерами

4. **Эксперименты**: отслеживание параметров и метрик

5. **Интеграция**: CI/CD и другие MLOps инструменты

  

### 14.2 Следующие шаги для изучения

  

1. **DVC Studio** - веб-интерфейс для управления экспериментами

2. **CML (Continuous Machine Learning)** - CI/CD для ML

3. **Iterative Studio** - платформа для командной работы

4. **DVCLive** - логирование метрик в реальном времени

  

### 14.3 Полезные команды для справки

  

```bash

# Основные команды

dvc init # Инициализация DVC

dvc add <file> # Добавление файла под контроль DVC

dvc push # Загрузка в удаленное хранилище

dvc pull # Скачивание из удаленного хранилища

dvc repro # Запуск пайплайна

dvc status # Проверка статуса

  

# Работа с remote

dvc remote add <name> <url> # Добавление remote

dvc remote list # Список remotes

dvc remote remove <name> # Удаление remote

  

# Эксперименты

dvc exp run # Запуск эксперимента

dvc exp show # Показать эксперименты

dvc exp diff # Сравнить эксперименты

  

# Метрики и параметры

dvc metrics show # Показать метрики

dvc metrics diff # Сравнить метрики

dvc params diff # Сравнить параметры

  

# Отладка

dvc doctor # Диагностика

dvc dag # Граф зависимостей

dvc cache dir # Путь к кешу

```

  

Этот конспект покрывает основные аспекты работы с DVC. Начните с простых операций добавления файлов и постепенно переходите к созданию пайплайнов и интеграции с другими инструментами. DVC - мощный инструмент, который значительно упростит управление данными и экспериментами в ваших ML-проектах.