

* **[`LangChai`](https://docs.langchain.com/docs/)** - фреймворк для работы с языковыми моделями, позволяющий сильно ускорить процесс создания вашего AI продукта. Был представлен в октябре 2022 года, с тех пор продолжает развиваться и собирать армию пользователей.   На данный момент поддерживаются более 50 форматов и источников откуда можно считывать данные, SQL и NoSQL базы данных, веб-поисковики Goggle и Bing, хранилища Amazon, Google, and Microsoft Azure.   Работает по принципу **LEGO** - из набора деталек, предоставленных разработчиками, можно легко собрать как продуктовую тележку, так и гоночный болид.

**Основные компоненты:**

* **🤖 Модели (Models)** : **универсальный интерфейс** для работы с различными языковыми моделями. Можно использовать API OpenAI, HuggingFace, Anthropic и других. Также есть возможность работать с локальными моделями.

* **📝 Промпты (Prompts)**: LangChain предоставляет ряд функций для работы с промптами – представление промпта согласно типу модели, **формирование шаблона** на основе внешних данных, форматирование вывода модели.

* **🗑 Индексы (Indexs)**: индексы структурируют документы для оптимального взаимодействия с языковыми моделями. Модуль включает функции для работы с документами, индексами и их использованием в цепочках. В том числе **поддерживает индексы, основанные на векторных базах данных**.

* **🧠 Память (Memory)**: позволяет сохраняет состояния в цепочках. Например, для создания чат-бота можно **сохранять предыдущие вопросы и ответы**. Существует два типа памяти: краткосрочная и долгосрочная. Краткосрочная память передает данные в рамках одного разговора. Долгосрочная память отвечает за доступ и обновление информации между разговорами.

* **🔗 Цепочки (Chains)**: С помощью цепочек можно объединять разные языковые модели и запросы в **многоступенчатые конвееры**. Цепочки могут быть применены для разговоров, ответов на вопросы, суммаризаций и других сценариев.

* **🥷 Агенты (Agents)**: С помощью агентов модель может получить **доступ к различным источникам информации**, таким как Google, Wikipedia итд.

** 🦜 Причём тут попугаи?*: Большие языковые модели часто сравнивают с говорящими попугаями, которые могут произносить текст как люди, но не понимают смысла произнесенного. 


**PromptTemplate** - вместо того, чтобы каждый раз писать промт напрямую, мы создаем `PromptTemplate` с запросом одной входной переменной.
```python
from langchain import PromptTemplate
template = """Ответь на вопрос {question}"""
prompt_template = PrompTemplate(
	input_variables=["question"],
	template=template
)
prompt = prompt_template.format(question="Что ты умеешь делать?")
```
Отличие от f-strings - мы можем создавать промпты объектно-ориентированным способом. 

**ChatPromptTemplate** - простой класс, позволяющий удобно создавать шаблоны промптов для использования LLM в режиме чата.

```python
from langchain.prompts import ChatPromptTemplate
template = "Опираясь на контекст {context} ответь на вопрос {question}"
prompt_template = ChatPromptTemplate.from_template(template)
prompt = prompt_template.format_messages(
	context = "Ламы и альпаки водятся в Перу.",
	question = "Где водятся ламы?"
)
prompt # [HumanMessage(content='Опираясь на контекст Ламы и альпаки водятся в Перу. ответь на вопрос Где водятся ламы?', additional_kwargs={}, response_metadata={})]
```
В чат `ChatPromptTemplate` в отличии от `PromptTemplate`, есть указатели **AIMessage** и **HumanMessage** и подразумевается диалоговая форма.

**FewShotPromptTemplate** 
```python
from langchain import FewShotPromptTemplate
# записываем наши примеры в список (в будущем это будет автоматизированно)
examples = [
	{
	"query": "Как дела?",
	"answer": "Не могу пожаловаться, но иногда всё-таки жалуюсь."
	},
	{
	"query": "Сколько время?",
	"answer": "Самое время купить часы."
	}
]

# создаём template для примеров
example_template = """
	User: {query}
	AI: {answer}
"""
# создаём промпт из шаблона выше
example_prompt = PromptTemplate(
	input_variables=["query", "answer"],
	template=example_template
)

# теперь разбиваем наш предыдущий промпт на prefix и suffix
# где - prefix это наша инструкция для модели
prefix = """Это разговор с ИИ-помощником.
Помощник обычно саркастичен, остроумен, креативен
и даёт забавные ответы на вопросы пользователей.
Вот несколько примеров:
"""
# а suffix - это вопрос пользователя и поле для ответа
suffix = """
User: {query}
AI: """
# создаём сам few shot prompt template
few_shot_prompt_template = FewShotPromptTemplate(
	examples=examples,
	example_prompt=example_prompt,
	prefix=prefix,
	suffix=suffix,
	input_variables=["query"],
	example_separator="\n\n"
)
```

Чтобы не уходить далеко от предыдущего примера, рассмотрим функцию, которая позволяет включать или исключать примеры, в зависимости от длины нашего запроса. Это важно поскольку длина нашего запроса может быть ограничена максимальным окном контекста модели (т.е. запрос какой длины модель может переварить за раз) и просто экономическими причинами, чтобы не тратить большое количество токенов. Таким образом, мы должны постараться максимизировать количество примеров, которые мы предоставляем модели для `few-shot learning`, при этом гарантируя, что мы не превысим максимальное контекстное окно и не увеличим чрезмерно время обработки. Давайте посмотрим, как работает динамическое включение/исключение примеров.
```python
from langchain.prompts.example_selector import LengthBasedExampleSelector

example_selector = LengthBasedExampleSelector(
	examples=examples,
	example_prompt=example_prompt,
	max_length=50 # параметром выставляется максимальная длина примера
)

# создаём новый few shot prompt template
dynamic_prompt_template = FewShotPromptTemplate(
	example_selector=example_selector, # используем example_selector вместо examples
	example_prompt=example_prompt,
	prefix=prefix,
	suffix=suffix,
	input_variables=["query"],
	example_separator="\n"
)
query = "Сколько звезд на небе?"
prompt = dynamic_prompt_template.format(query=query)
```
**StructuredOutputParser**

```python
from langchain.output_parsers import ResponseSchema, StructuredOutputParser

schema_1 = ResponseSchema(
	name="name_1",
	description="description_1"
)
schema_2 = ResponseSchema(
	name="name_2",
	description="description_2"
)  
response_schemas = [
	schema_1,
	schema_2
]
output_parser = StructuredOutputParser.from_response_schemas(response_schemas)
format_instructions = output_parser.get_format_instructions()

template = """
	template_text: {template_text}
	
	{format_instructions}

"""
prompt = ChatPromptTemplate.from_template(
	template=template,
	
)
messages = prompt.format_messages(
	template_text="...",
	format_instructions=format_instructions
)

response = chat.invoke(messages)
output_dict = output_parser.parse(response.content)
```

