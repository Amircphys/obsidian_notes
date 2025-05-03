
Давайте создадим подробный гайд по работе с OpenAI и LangChain. Вот структурированная инструкция:

# Гайд по использованию OpenAI и LangChain для работы с LLM

## 1. Установка библиотек
```bash
pip install openai langchain-openai
```

## 2. Работа с OpenAI API напрямую

### 2.1 Настройка API ключа
```python
import openai

openai.api_key = "ваш_api_ключ"
openai.base_url = "https://api.proxyapi.ru/openai/v1"
# Или через переменные среды:
# export OPENAI_API_KEY="ваш_ключ"
```

### 2.2 Простой запрос (Completion)
```python
response = openai.Completion.create(
    model="gpt-3.5-turbo-instruct",  # Модель для завершения текста
    prompt="Напиши короткий стих про Python",
    max_tokens=100,      # Максимальное количество токенов в ответе
    temperature=0.7,     # Контроль случайности (0-2)
    top_p=1,             # Альтернатива temperature
    n=1,                 # Количество вариантов ответа
    stop=None,           # Стоп-слова для остановки генерации
)

print(response.choices[0].text)
```

### 2.3 Чат-взаимодействие (Chat Completion)
```python
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",  # Или "gpt-4"
    messages=[
        {"role": "system", "content": "Ты помощник-поэт"},
        {"role": "user", "content": "Напиши хайку про нейросети"}
    ],
    temperature=0.5,
    max_tokens=500,
    presence_penalty=0.6,  # Штраф за повторение тем
    frequency_penalty=0.6, # Штраф за повторение слов
)

print(response.choices[0].message.content)
```

## 3. Работа с LangChain

### 3.1 Базовое использование ChatOpenAI
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model_name="gpt-3.5-turbo",
    temperature=0.7,
    max_tokens=300,
    streaming=True,  # Для потоковой передачи
)

response = llm.invoke("Объясни квантовую запутанность простыми словами")
print(response.content)
```

### 3.2 Создание цепочки с промптом
```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template(
    "Ответь как {role}. Вопрос: {question}"
)

chain = prompt | llm | StrOutputParser()

response = chain.invoke({
    "role": "саркастичный IT-специалист",
    "question": "Почему программа не работает?"
})

print(response)
```

## 4. Основные параметры и аргументы

### Для OpenAI API:
- **model**: Идентификатор модели (gpt-3.5-turbo, gpt-4 и т.д.)
- **temperature** (0-2): 
  - 0 = детерминированные ответы
  - 1 = стандартная случайность
- **max_tokens**: Максимальная длина ответа
- **top_p**: Альтернатива температуре (0-1)
- **frequency_penalty** (-2.0 - 2.0): Штраф за частые слова
- **presence_penalty** (-2.0 - 2.0): Штраф за повтор тем
- **n**: Количество вариантов ответа
- **stop**: Список стоп-слов для остановки генерации

### Для LangChain:
```python
ChatOpenAI(
    model_name="gpt-3.5-turbo",  # Модель
    temperature=0.7,             # Креативность
    max_tokens=256,              # Макс. длина ответа
    openai_api_key="...",        # Опционально, если не через .env
    streaming=True,              # Потоковый вывод
    max_retries=3,               # Попытки при ошибках
    model_kwargs={}              # Доп. параметры модели
)
```

## 5. Обработка ошибок
```python
from openai import APIError

try:
    # Ваш код запроса
except APIError as e:
    print(f"Ошибка API: {e}")
except Exception as e:
    print(f"Общая ошибка: {e}")
```

## 6. Советы и лучшие практики
1. Всегда устанавливайте max_tokens для контроля расходов
2. Начинайте с temperature=0.5 и регулируйте по необходимости
3. Для сложных задач используйте цепочки LangChain
4. Для длинных диалогов сохраняйте историю сообщений
5. Используйте system-роли для управления поведением модели

Пример сохранения истории чата:
```python
chat_history = []

def chat(message):
    chat_history.append({"role": "user", "content": message})
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=chat_history
    )
    assistant_response = response.choices[0].message.content
    chat_history.append({"role": "assistant", "content": assistant_response})
    return assistant_response
```

## 7. Когда использовать что:
- **OpenAI API напрямую**: Простые запросы, быстрое прототипирование
- **LangChain**: 
  - Сложные цепочки вызовов
  - Интеграция с другими источниками данных
  - Работа с векторными базами данных
  - Многомодельные системы

Этот гайд покрывает основные сценарии работы с LLM через Python. Всегда проверяйте актуальность версий библиотек и доступность моделей в вашем API-аккаунте.



**PromptTemplate из langchain.prompts** — это инструмент для создания стандартизированных шаблонов промптов, которые позволяют динамически подставлять значения переменных в текст запроса. Он упрощает работу с LLM, делая промпты переиспользуемыми и управляемыми.

---

**FewShotPromptTemplate** — это специальный тип шаблона в LangChain, который позволяет добавлять примеры ввода-вывода непосредственно в промпт. Это особенно полезно для задач, где нужно "научить" модель определенному формату или стилю ответов через демонстрацию конкретных примеров (few-shot learning).

---

### **Для чего используют?**
1. **Обучение через примеры**: Показ модели, как обрабатывать特定ные типы запросов.
2. **Сохранение контекста**: Поддержание единого стиля/формата во всех ответах.
3. **Сложные задачи**: Классификация текста, генерация по шаблону, извлечение данных.

---

### **Основные компоненты**

| Параметр | Описание |
|----------|----------|
| `examples` | Список примеров (например, `[{"input": "...", "output": "..."}]`) |
| `example_prompt` | Шаблон для форматирования каждого примера (`PromptTemplate`) |
| `prefix` | Текст перед примерами (инструкция для модели) |
| `suffix` | Текст после примеров (текущий запрос пользователя) |
| `example_separator` | Разделитель между примерами (по умолчанию `\n\n`) |
| `input_variables` | Переменные для подстановки в `suffix` |

---

### **Пример 1: Простая классификация тональности**
```python
from langchain.prompts import FewShotPromptTemplate, PromptTemplate

# Примеры для обучения
examples = [
    {"text": "Этот фильм просто шедевр!", "label": "POSITIVE"},
    {"text": "Ужасное обслуживание в кафе.", "label": "NEGATIVE"},
    {"text": "Погода сегодня обычная.", "label": "NEUTRAL"}
]

# Шаблон для оформления каждого примера
example_template = PromptTemplate(
    input_variables=["text", "label"],
    template="Текст: {text}\nТональность: {label}"
)

# Создание FewShot-шаблона
few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_template,
    prefix="Определи тональность текста. Примеры:",
    suffix="Текст: {input_text}\nТональность:",
    input_variables=["input_text"],
    example_separator="\n---\n"
)

# Генерация промпта
prompt = few_shot_prompt.format(input_text="Новый ноутбук работает быстро.")
print(prompt)
```

**Результат:**
```
Определи тональность текста. Примеры:

Текст: Этот фильм просто шедевр!
Тональность: POSITIVE
---
Текст: Ужасное обслуживание в кафе.
Тональность: NEGATIVE
---
Текст: Погода сегодня обычная.
Тональность: NEUTRAL

Текст: Новый ноутбук работает быстро.
Тональность:
```

---

### **Пример 2: Генерация ответов в заданном стиле**
```python
examples = [
    {
        "question": "Как научиться программировать?",
        "answer": "11 шагов:\n1. Выбери язык\n2. Установи среду разработки..."
    },
    {
        "question": "Как приготовить омлет?",
        "answer": "5 этапов:\n1. Взбить яйца\n2. Разогреть сковороду..."
    }
]

example_template = PromptTemplate(
    input_variables=["question", "answer"],
    template="Вопрос: {question}\nОтвет: {answer}"
)

few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_template,
    prefix="Отвечай в формате нумерованных списков. Примеры ответов:",
    suffix="Вопрос: {user_question}\nОтвет:",
    input_variables=["user_question"]
)

prompt = few_shot_prompt.format(user_question="Как начать бегать по утрам?")
```

**Результат вызова LLM:**
```
Вопрос: Как начать бегать по утрам?
Ответ: 7 шагов:
1. Купи удобные кроссовки
2. Начни с коротких дистанций
3. Установи будильник на одно время...
```

---

### **Советы по использованию**
1. **Релевантность примеров**: Выбирайте примеры, максимально близкие к целевой задаче.
2. **Баланс длины**: Не перегружайте промпт (риск превышения лимита токенов).
3. **Динамический выбор примеров**: Используйте `ExampleSelector` для умного подбора примеров из базы.
4. **Форматирование**: Четко разделяйте примеры и текущий запрос (используйте `example_separator`).

---

### **Пример с динамическим выбором примеров**
```python
from langchain.prompts.example_selector import SemanticSimilarityExampleSelector
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

# Примеры с метаданными
examples = [
    {"query": "Как сделать скриншот?", "os": "Windows", "answer": "Нажмите Win + Shift + S..."},
    {"query": "Скриншот экрана", "os": "macOS", "answer": "Cmd + Shift + 4..."},
    {"query": "Создать скриншот", "os": "Linux", "answer": "Используйте Shift + PrtScn..."}
]

# Выбор наиболее похожих примеров по семантике
example_selector = SemanticSimilarityExampleSelector.from_examples(
    examples=examples,
    embeddings=OpenAIEmbeddings(),
    vectorstore_cls=FAISS,
    k=1  # Выбирать 1 наиболее релевантный пример
)

few_shot_prompt = FewShotPromptTemplate(
    example_selector=example_selector,  # Вместо статических examples
    example_prompt=example_template,
    prefix="Ответь на вопрос для ОС {os}:",
    suffix="Вопрос: {query}\nОтвет:",
    input_variables=["os", "query"]
)

# Запрос для Windows
prompt = few_shot_prompt.format(os="Windows", query="Как сохранить скриншот?")
```

---

### **Типичные сценарии**
- **Генерация SQL-запросов** (примеры "вопрос → SQL")
- **Парсинг неструктурированных данных** (примеры "текст → JSON")
- **Перевод с особыми требованиями** (например, сохранение терминов)
- **Создание контента по шаблону** (новости, описания товаров)

**Важно:** FewShotPromptTemplate особенно эффективен с мощными моделями вроде GPT-4, которые хорошо понимают контекст из примеров.