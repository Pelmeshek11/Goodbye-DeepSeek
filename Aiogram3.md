📚 Aiogram 3.x (основные модули)

1. Bot и Dispatcher

```python
from aiogram import Bot, Dispatcher

# Создание бота
bot = Bot(token="TOKEN")
dp = Dispatcher()

# Основные методы бота
await bot.send_message()
await bot.edit_message_text()
await bot.delete_message()
await bot.get_me()
await bot.get_updates()
```

2. Роутеры и обработчики

```python
from aiogram import Router
from aiogram.filters import Command
from aiogram.types import Message

router = Router()

@router.message(Command("start"))
async def cmd_start(message: Message):
    await message.answer("Привет!")
```

3. Фильтры (Filters)

```python
from aiogram.filters import Command, StateFilter, Text
from aiogram.filters.command import CommandObject
from aiogram.filters.callback_data import CallbackData
```

4. FSM (Finite State Machine)

```python
from aiogram.fsm.state import State, StatesGroup
from aiogram.fsm.context import FSMContext
from aiogram.fsm.storage.memory import MemoryStorage

class Form(StatesGroup):
    name = State()
    age = State()
```

5. Middleware

```python
from aiogram import BaseMiddleware
from aiogram.types import Message
```

6. Клавиатуры

```python
from aiogram.types import (
    ReplyKeyboardMarkup,
    KeyboardButton,
    InlineKeyboardMarkup,
    InlineKeyboardButton
)

from aiogram.utils.keyboard import (
    ReplyKeyboardBuilder,
    InlineKeyboardBuilder
)
```

7. Типы данных

```python
from aiogram.types import (
    Message, CallbackQuery, InlineQuery,
    User, Chat, PhotoSize, Document,
    Audio, Voice, Contact, Location
)
```

8. Утилиты

```python
from aiogram.utils.formatting import (
    Text, Bold, Italic, Code,
    as_list, as_marked_section
)

from aiogram.utils.markdown import (
    hbold, hitalic, hcode, hlink
)

from aiogram.utils.media_group import MediaGroupBuilder
```

🔐 Aiogram Crypto (проверка WebApp данных)

Основные функции:

```python
from aiogram.utils.web_app import (
    safe_parse_webapp_init_data,
    check_webapp_signature
)

# Проверка подписи данных от WebApp
is_valid = check_webapp_signature(
    token="BOT_TOKEN",
    init_data="init_data_string"
)

# Безопасный парсинг данных
init_data = safe_parse_webapp_init_data(
    token="BOT_TOKEN",
    init_data="init_data_string"
)

# Проверка токена
from aiogram.utils.token import validate_token
is_valid_token = validate_token("TOKEN")
```

🌐 Aiohttp (асинхронный HTTP-клиент/сервер)

1. Клиентская часть

```python
import aiohttp

# Основные методы
async with aiohttp.ClientSession() as session:
    # GET запрос
    async with session.get(url) as response:
        text = await response.text()
        json = await response.json()
        status = response.status
        headers = response.headers
    
    # POST запрос
    async with session.post(url, json=data) as response:
        ...
    
    # PUT, DELETE, PATCH, HEAD, OPTIONS
    async with session.put(url, data=data)
    async with session.delete(url)
    
    # Параметры запроса
    params = {'key': 'value'}
    headers = {'Authorization': 'Bearer token'}
    cookies = {'name': 'value'}
    proxy = "http://proxy.com"
    timeout = aiohttp.ClientTimeout(total=10)
```

2. Серверная часть

```python
from aiohttp import web

# Создание приложения
app = web.Application()

# Роутинг
routes = web.RouteTableDef()

@routes.get('/')
async def handler(request):
    return web.Response(text="Hello")

@routes.post('/api')
async def api_handler(request):
    data = await request.json()
    return web.json_response(data)

# Параметры запроса
async def handler(request):
    query = request.query  # GET параметры
    post_data = await request.post()  # POST данные
    json_data = await request.json()  # JSON данные
    
    # Чтение файлов
    reader = await request.multipart()
    
    # Установка куки
    response = web.Response()
    response.set_cookie('name', 'value')
    
    return response

# Middleware
@web.middleware
async def middleware(request, handler):
    # До обработки
    response = await handler(request)
    # После обработки
    return response

# Запуск сервера
web.run_app(app, host='0.0.0.0', port=8080)
```

3. WebSocket

```python
# Клиент
async with session.ws_connect(url) as ws:
    async for msg in ws:
        if msg.type == aiohttp.WSMsgType.TEXT:
            await ws.send_str("Hello")
        elif msg.type == aiohttp.WSMsgType.ERROR:
            break

# Сервер
async def websocket_handler(request):
    ws = web.WebSocketResponse()
    await ws.prepare(request)
    
    async for msg in ws:
        if msg.type == aiohttp.WSMsgType.TEXT:
            await ws.send_str(msg.data)
    
    return ws
```

4. Сессии и куки

```python
# Работа с сессиями
session = aiohttp.ClientSession(cookies={"name": "value"})

# CookieJar
jar = aiohttp.CookieJar()
session = aiohttp.ClientSession(cookie_jar=jar)
```

5. Обработка ошибок

```python
from aiohttp import ClientError, ClientResponseError

try:
    async with session.get(url) as response:
        response.raise_for_status()
except aiohttp.ClientError as e:
    print(f"HTTP ошибка: {e}")
except asyncio.TimeoutError:
    print("Таймаут")
```

6. Продвинутые возможности

```python
# Кастомные заголовки
headers = {
    'User-Agent': 'MyBot/1.0',
    'Authorization': 'Bearer token'
}

# SSL контекст
ssl_context = ssl.create_default_context()

# Прокси
connector = aiohttp.TCPConnector(
    limit=100,  # лимит соединений
    ssl=False,
    ttl_dns_cache=300
)

# Формирование URL
from yarl import URL
url = URL("https://api.com").with_query({"param": "value"})
```

📦 Полезные комбинации

Aiogram + Aiohttp:

```python
async def fetch_data(url: str):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

# В хендлере aiogram
@router.message(Command("data"))
async def get_data(message: Message):
    data = await fetch_data("https://api.example.com")
    await message.answer(str(data))
```

Webhook для aiogram:

```python
from aiogram.webhook.aiohttp_server import (
    SimpleRequestHandler,
    setup_application
)

# Настройка webhook
handler = SimpleRequestHandler(
    dispatcher=dp,
    bot=bot,
    secret_token="SECRET"
)
handler.register(app, path="/webhook")
```
