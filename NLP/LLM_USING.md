
Конечно! Давай подробно разберём метод `PromptTemplate` из библиотеки **LangChain**, который является одним из базовых и важных элементов при работе с большими языковыми моделями (LLM). Он помогает автоматизировать создание промтов, делая их динамическими и удобными для использования в различных задачах.

---

## 🔹 Что такое PromptTemplate?

**PromptTemplate** — это инструмент, позволяющий формировать шаблоны промтов, которые можно заполнять переменными. Это очень похоже на f-строки в Python, только вместо того чтобы подставлять значения прямо в код, вы создаёте шаблон, а затем используете его в разных частях приложения, передавая нужные данные.

### 📦 Где используется:
- При взаимодействии с LLM через LangChain.
- Для унификации входных данных, особенно когда нужно менять параметры (например, имя пользователя, тему текста, контекст и т.д.).
- В цепочках (`chains`) для генерации контекста перед вызовом модели.
- В RAG (Retrieval-Augmented Generation) системах для формирования запросов к модели с учетом retrieved данных.

---

## 🔧 Основные компоненты PromptTemplate

```python
from langchain.prompts import PromptTemplate
```

Класс `PromptTemplate` имеет следующие основные параметры:

| Параметр                            | Описание                                                                       |
| ----------------------------------- | ------------------------------------------------------------------------------ |
| `template`                          | Строка с шаблоном, где используются переменные в фигурных скобках `{variable}` |
| `input_variables`                   | Список переменных, которые будут использоваться в шаблоне                      |
| `partial_variables` *(опционально)* | Переменные, которые фиксируются при создании шаблона                           |

---

## ✅ Пример 1: Простой PromptTemplate

```python
from langchain.prompts import PromptTemplate

# Создаем шаблон
template = "Расскажи мне о {topic} простым языком."

# Создаем объект PromptTemplate
prompt = PromptTemplate(
    input_variables=["topic"],
    template=template,
)

# Формируем промт
print(prompt.format(topic="нейронные сети"))
```

**Вывод:**
```
Расскажи мне о нейронных сетях простым языком.
```

---

## ✅ Пример 2: Несколько переменных

```python
template = """
Ты - {profession}, тебе нужно объяснить {concept} человеку без опыта в этой области.
Объясни просто и понятно.
"""

prompt = PromptTemplate(
    input_variables=["profession", "concept"],
    template=template,
)

print(prompt.format(profession="учитель математики", concept="производная функции"))
```

**Вывод:**
```
Ты - учитель математики, тебе нужно объяснить производная функции человеку без опыта в этой области.
Объясни просто и понятно.
```

---

## ✅ Пример 3: Partial Variables (частичное заполнение)

Иногда часть переменных известна заранее, и её можно зафиксировать:

```python
prompt = PromptTemplate.from_template(
    "Напиши короткий {type_text} о {subject}."
)

# Фиксируем тип текста как "эссе"
partial_prompt = prompt.partial(type_text="эссе")

# Теперь мы можем использовать partial_prompt, передавая только subject
print(partial_prompt.format(subject="влияние ИИ на образование"))
```

**Вывод:**
```
Напиши короткое эссе о влиянии ИИ на образование.
```

---

## ✅ Пример 4: Использование вместе с моделью LLM

```python
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate

# Инициализируем модель (предварительно установите openai и задайте API ключ)
llm = OpenAI(model_name="text-davinci-003", temperature=0.7)

# Шаблон
prompt = PromptTemplate(
    input_variables=["topic"],
    template="Расскажи мне интересный факт о {topic}.",
)

# Формируем промт
formatted_prompt = prompt.format(topic="космос")

# Отправляем в модель
response = llm(formatted_prompt)
print(response)
```

**Пример вывода от модели:**
```
Интересный факт о космосе: Более 95% вещества во Вселенной состоит из тёмной материи и тёмной энергии, которые мы пока не можем наблюдать напрямую.
```

---

## ✅ Пример 5: JSON и сложные структуры

Можно также сохранять/загружать шаблоны в формате JSON, что полезно при сериализации конфигураций или обмене между компонентами системы.

```python
prompt.save("my_prompt.json")  # Сохранение
loaded_prompt = PromptTemplate.load("my_prompt.json")  # Загрузка
```

---

## 🧠 Когда использовать PromptTemplate?

- Когда нужно генерировать разные промты на основе одних и тех же правил.
- Когда вы строите цепочки (`LLMChain`, `SequentialChain` и т.д.).
- Чтобы отделить логику формирования промта от его содержания (чистый код).
- Если вы хотите переиспользовать промты в разных частях приложения.

---

## 🔄 Альтернативы и расширения

- `FewShotPromptTemplate` — добавляет примеры (few-shot learning).
- `ChatPromptTemplate` — для работы с чат-моделями (например, GPT-3.5-turbo), учитывает роли (system/user/assistant).
- `PipelinePromptTemplate` — комбинирование нескольких шаблонов.

---

## 📌 Заключение

`PromptTemplate` — это мощный инструмент для стандартизации и параметризации промтов в LangChain. Он позволяет легко масштабировать работу с LLM, обеспечивает гибкость и повторное использование кода. Это одна из первых вещей, которую стоит освоить при работе с LangChain.

---
Конечно! `ChatPromptTemplate` — это ключевой инструмент в **LangChain**, предназначенный для работы с **чат-моделями** (например, `gpt-3.5-turbo`, `gpt-4` и другими), которые требуют структурированных входных данных в виде сообщений с ролями (`system`, `user`, `assistant`). В отличие от классического `PromptTemplate`, он генерирует список сообщений, а не простую строку.

---

## 🔍 Что такое ChatPromptTemplate?

`ChatPromptTemplate` позволяет создавать **шаблоны сообщений**, где каждое сообщение имеет:
- **Роль**: `system`, `user`, `assistant`.
- **Содержание**: текст, который может включать переменные (через `{variable}`).

### 📌 Основные особенности:
- Поддержка **многоступенчатых диалогов**.
- Совместимость с чат-моделями, такими как `ChatOpenAI`.
- Возможность комбинировать статические и динамические части сообщений.

---

## 🧱 Структура ChatPromptTemplate

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
```

Класс `ChatPromptTemplate` строится из **Message Objects**, например:

```python
messages = [
    SystemMessage(content="Ты — помощник по путешествиям."),
    HumanMessage(content="Порекомендуй мне 3 города в Италии для посещения."),
    AIMessage(content="Вот мои рекомендации: Рим, Флоренция, Венеция.")
]
```

Но вместо ручного написания таких сообщений, мы используем шаблоны:

---

## ✅ Пример 1: Простой шаблон с ролью `system` и `user`

```python
from langchain.prompts import ChatPromptTemplate

# Создаем шаблон
template = ChatPromptTemplate.from_messages([
    ("system", "Ты — {profession}, который объясняет сложные темы простым языком."),
    ("human", "Объясни, что такое {topic}?")
])

# Формируем сообщения
messages = template.format_messages(profession="учитель физики", topic="квантовая механика")

# Теперь messages содержит:
# [SystemMessage(content='Ты — учитель физики, который объясняет сложные темы простым языком.'),
#  HumanMessage(content='Объясни, что такое квантовая механика?')]
```

---

## ✅ Пример 2: Шаблон с несколькими ролями и переменными

```python
template = ChatPromptTemplate.from_messages([
    ("system", "Ты — {role}, который помогает с {task}."),
    ("human", "Мне нужно {request}."),
    ("ai", "Хорошо, я могу помочь с этим.")
])

messages = template.format_messages(
    role="путеводитель",
    task="туристическими маршрутами",
    request="выбрать маршрут по Европе на 7 дней"
)
```

---

## ✅ Пример 3: Работа с моделью (ChatOpenAI)

```python
from langchain.chat_models import ChatOpenAI

# Инициализируем модель
llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0.7)

# Шаблон
template = ChatPromptTemplate.from_messages([
    ("system", "Ты — эксперт по кулинарии. Объясни, как приготовить {dish} шаг за шагом."),
    ("human", "Как приготовить {dish}?")
])

# Формируем сообщения
messages = template.format_messages(dish="паэлью")

# Отправляем в модель
response = llm(messages)
print(response.content)
```

**Пример вывода модели:**
```
Паэлья — традиционное испанское блюдо. Вот как её приготовить:
1. Нагрейте оливковое масло в большой сковороде.
2. Добавьте нарезанный лук и чеснок, обжарьте до золотистого цвета...
```

---

## ✅ Пример 4: Partial Variables (частичное заполнение)

Иногда часть переменных можно зафиксировать заранее:

```python
template = ChatPromptTemplate.from_messages([
    ("system", "Ты — {role}, который помогает с {task}."),
    ("human", "Мне нужно {request}.")
])

# Фиксируем role и task
partial_template = template.partial(role="переводчик", task="переводом текста")

# Теперь нужно только указать request
messages = partial_template.format_messages(request="переведи текст с английского на русский")
```

---

## ✅ Пример 5: Сохранение и загрузка шаблона

Можно сохранять шаблоны в JSON для переиспользования:

```python
template.save("chat_prompt.json")  # Сохранение
loaded_template = ChatPromptTemplate.load("chat_prompt.json")  # Загрузка
```

---

## 🧩 Расширенные возможности

### 1. **FewShotPromptTemplate + ChatPromptTemplate**
Можно добавлять примеры (few-shot) в чат-шаблоны:

```python
from langchain.prompts.few_shot import FewShotChatMessagePromptTemplate

examples = [
    {"input": "Сколько будет 2+2?", "output": "2+2=4"},
    {"input": "Что такое космос?", "output": "Космос — всё сущее: звёзды, планеты, галактики..."}
]

example_prompt = ChatPromptTemplate.from_messages([
    ("human", "{input}"),
    ("ai", "{output}")
])

few_shot_prompt = FewShotChatMessagePromptTemplate(
    example_prompt=example_prompt,
    examples=examples
)

final_prompt = ChatPromptTemplate.from_messages([
    ("system", "Ты — умный помощник. Отвечай кратко и точно."),
    few_shot_prompt,
    ("human", "{query}")
])

messages = final_prompt.format_messages(query="Как зовут президента России?")
```

---

## 🔄 Отличия от обычного PromptTemplate

| Характеристика | `PromptTemplate` | `ChatPromptTemplate` |
|----------------|------------------|-----------------------|
| Тип модели | Строковые модели (`text-davinci-003`) | Чат-модели (`gpt-3.5-turbo`, `gpt-4`) |
| Формат входа | Одна строка | Список сообщений с ролями |
| Роли | Нет | `system`, `user`, `assistant` |
| Использование | Простые промты | Диалоги, многоступенчатые взаимодействия |

---

## 📌 Когда использовать ChatPromptTemplate?

- Когда вы работаете с **чат-моделями** (например, `ChatOpenAI`).
- Для создания **структурированных диалогов** с несколькими ролями.
- При построении **сложных цепочек** (`LLMChain`, `Agent`, `RetrievalQA` и т.д.).
- Если вам нужно **сохранить контекст** между сообщениями.

---

## 🚀 Полезные ссылки

- [Документация ChatPromptTemplate](https://python.langchain.com/docs/modules/model_io/prompts/chat_prompt_template/)
- [Примеры использования в репозитории LangChain](https://github.com/langchain-ai/langchain/tree/master/libs/core/langchain_core/prompts)

---
