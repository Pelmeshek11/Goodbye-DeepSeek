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

# **Вертикальное расположение кнопок в InlineKeyboard (aiogram)**

## 📏 **Основные способы вертикального расположения**

### **1. Способ 1: Каждая кнопка в отдельном списке**
```python
from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton

# Каждый вложенный список = новая строка
keyboard = InlineKeyboardMarkup(
    inline_keyboard=[
        [InlineKeyboardButton(text="Кнопка 1", callback_data="btn1")],  # Строка 1
        [InlineKeyboardButton(text="Кнопка 2", callback_data="btn2")],  # Строка 2
        [InlineKeyboardButton(text="Кнопка 3", callback_data="btn3")],  # Строка 3
        [InlineKeyboardButton(text="Кнопка 4", callback_data="btn4")],  # Строка 4
    ]
)
```

### **2. Способ 2: Использование InlineKeyboardBuilder (рекомендуется)**
```python
from aiogram.utils.keyboard import InlineKeyboardBuilder

builder = InlineKeyboardBuilder()

# Добавляем кнопки одну за другой
builder.button(text="Кнопка 1", callback_data="btn1")
builder.button(text="Кнопка 2", callback_data="btn2")
builder.button(text="Кнопка 3", callback_data="btn3")
builder.button(text="Кнопка 4", callback_data="btn4")

# Устанавливаем 1 кнопку в строку
builder.adjust(1)

keyboard = builder.as_markup()
```

## 🎯 **Практические примеры**

### **Пример 1: Меню с вертикальными кнопками**
```python
@router.message(Command("menu"))
async def vertical_menu(message: Message):
    builder = InlineKeyboardBuilder()
    
    # Добавляем кнопки вертикально
    builder.button(text="📊 Профиль", callback_data="profile")
    builder.button(text="⚙️ Настройки", callback_data="settings")
    builder.button(text="💳 Баланс", callback_data="balance")
    builder.button(text="📞 Поддержка", callback_data="support")
    builder.button(text="📚 Помощь", callback_data="help")
    
    # 1 кнопка в строке = вертикальное расположение
    builder.adjust(1)
    
    await message.answer(
        "📱 Главное меню:",
        reply_markup=builder.as_markup()
    )
```

### **Пример 2: Список товаров вертикально**
```python
@router.message(Command("products"))
async def show_products(message: Message):
    products = [
        {"name": "📱 iPhone 15", "price": "$999", "id": 1},
        {"name": "💻 MacBook Pro", "price": "$1999", "id": 2},
        {"name": "⌚️ Apple Watch", "price": "$399", "id": 3},
        {"name": "🎧 AirPods Pro", "price": "$249", "id": 4},
        {"name": "📱 iPad Pro", "price": "$1099", "id": 5},
    ]
    
    builder = InlineKeyboardBuilder()
    
    for product in products:
        # Каждый товар на отдельной строке
        builder.button(
            text=f"{product['name']} - {product['price']}",
            callback_data=f"product_{product['id']}"
        )
    
    # Кнопки навигации тоже вертикально
    builder.button(text="⬅️ Назад", callback_data="back")
    builder.button(text="📋 Корзина", callback_data="cart")
    builder.button(text="🏠 Главная", callback_data="home")
    
    builder.adjust(1)  # Все кнопки вертикально
    
    await message.answer(
        "🛒 Наши товары:",
        reply_markup=builder.as_markup()
    )
```

### **Пример 3: Вертикальные кнопки с разными типами**
```python
@router.message(Command("actions"))
async def vertical_actions(message: Message):
    builder = InlineKeyboardBuilder()
    
    # Кнопка с callback
    builder.button(text="✅ Подтвердить", callback_data="confirm")
    
    # Кнопка с URL
    builder.button(text="🌐 Наш сайт", url="https://example.com")
    
    # Кнопка для Web App
    builder.button(
        text="📱 Открыть приложение",
        web_app=WebAppInfo(url="https://yourapp.com")
    )
    
    # Кнопка с логином
    builder.button(
        text="🔐 Авторизация",
        login_url=LoginUrl(url="https://auth.example.com")
    )
    
    # Все кнопки вертикально
    builder.adjust(1)
    
    await message.answer(
        "Выберите действие:",
        reply_markup=builder.as_markup()
    )
```

## 🔄 **Комбинирование горизонтальных и вертикальных кнопок**

### **Пример: Вертикальные группы с горизонтальными кнопками внутри**
```python
@router.message(Command("complex"))
async def complex_menu(message: Message):
    builder = InlineKeyboardBuilder()
    
    # Группа 1: Действия (горизонтально)
    builder.button(text="✏️ Редактировать", callback_data="edit")
    builder.button(text="🗑️ Удалить", callback_data="delete")
    
    # Группа 2: Навигация (вертикально)
    builder.button(text="📊 Статистика", callback_data="stats")
    builder.button(text="⚙️ Настройки", callback_data="settings")
    
    # Группа 3: Ссылки (горизонтально)
    builder.button(text="🌐 Сайт", url="https://site.com")
    builder.button(text="📘 Документация", url="https://docs.com")
    
    # Настраиваем расположение:
    # Первые 2 кнопки в одной строке, затем 2 вертикально, затем 2 в одной строке
    builder.adjust(2, 1, 1, 2)
    
    await message.answer(
        "Комплексное меню:",
        reply_markup=builder.as_markup()
    )
```

## 📝 **Утилитные функции для вертикальных клавиатур**

### **Функция для создания вертикальной клавиатуры из списка**
```python
from typing import List, Tuple, Union

def create_vertical_keyboard(
    items: List[Tuple[str, Union[str, dict]]],
    include_back: bool = True
) -> InlineKeyboardMarkup:
    """
    Создает вертикальную клавиатуру из списка элементов
    
    Args:
        items: Список кортежей (текст, callback_data или dict с параметрами)
        include_back: Добавить кнопку "Назад"
    
    Returns:
        InlineKeyboardMarkup
    """
    builder = InlineKeyboardBuilder()
    
    for text, data in items:
        if isinstance(data, dict):
            # Если передан словарь с параметрами
            builder.button(text=text, **data)
        else:
            # Если передан просто callback_data
            builder.button(text=text, callback_data=data)
    
    if include_back:
        builder.button(text="🔙 Назад", callback_data="back")
    
    builder.adjust(1)  # Все кнопки вертикально
    return builder.as_markup()

# Использование
@router.message(Command("test"))
async def test_menu(message: Message):
    items = [
        ("Кнопка 1", "btn1"),
        ("Кнопка 2", "btn2"),
        ("Кнопка с URL", {"url": "https://example.com"}),
        ("Кнопка WebApp", {"web_app": WebAppInfo(url="https://app.com")}),
    ]
    
    keyboard = create_vertical_keyboard(items, include_back=True)
    
    await message.answer(
        "Тест вертикальной клавиатуры:",
        reply_markup=keyboard
    )
```

### **Фабрика вертикальных меню**
```python
class VerticalMenuFactory:
    """Фабрика вертикальных меню"""
    
    @staticmethod
    def create_main_menu() -> InlineKeyboardMarkup:
        """Главное меню вертикально"""
        builder = InlineKeyboardBuilder()
        
        menu_items = [
            ("📊 Профиль", "profile"),
            ("⚙️ Настройки", "settings"),
            ("💳 Баланс", "balance"),
            ("📞 Поддержка", "support"),
            ("📚 Справка", "help"),
        ]
        
        for text, callback in menu_items:
            builder.button(text=text, callback_data=callback)
        
        builder.adjust(1)
        return builder.as_markup()
    
    @staticmethod
    def create_admin_menu() -> InlineKeyboardMarkup:
        """Админ-меню вертикально"""
        builder = InlineKeyboardBuilder()
        
        admin_items = [
            ("📊 Статистика", "admin_stats"),
            ("👥 Пользователи", "admin_users"),
            ("📢 Рассылка", "admin_broadcast"),
            ("⚙️ Настройки", "admin_settings"),
            ("📁 Логи", "admin_logs"),
        ]
        
        for text, callback in admin_items:
            builder.button(text=text, callback_data=callback)
        
        builder.button(text="🔙 Главная", callback_data="main_menu")
        builder.adjust(1)
        return builder.as_markup()
```

## 🎮 **Полный пример бота с вертикальными кнопками**

```python
import asyncio
from aiogram import Bot, Dispatcher, Router, F
from aiogram.types import Message, CallbackQuery, InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.filters import Command
from aiogram.utils.keyboard import InlineKeyboardBuilder

router = Router()

# Главное меню с вертикальными кнопками
@router.message(Command("start"))
async def start_command(message: Message):
    builder = InlineKeyboardBuilder()
    
    # Все кнопки вертикально
    builder.button(text="👤 Профиль", callback_data="profile")
    builder.button(text="⚙️ Настройки", callback_data="settings")
    builder.button(text="🎮 Игры", callback_data="games")
    builder.button(text="📊 Статистика", callback_data="stats")
    builder.button(text="📞 Поддержка", callback_data="support")
    builder.button(text="📚 Справка", callback_data="help")
    
    builder.adjust(1)  # Ключевой момент - 1 кнопка в строке
    
    await message.answer(
        f"👋 Привет, {message.from_user.first_name}!\n\n"
        "Добро пожаловать в бота!\n"
        "Выберите раздел:",
        reply_markup=builder.as_markup()
    )

# Вертикальные кнопки для профиля
@router.callback_query(F.data == "profile")
async def profile_handler(callback: CallbackQuery):
    builder = InlineKeyboardBuilder()
    
    builder.button(text="✏️ Изменить имя", callback_data="edit_name")
    builder.button(text="📧 Изменить email", callback_data="edit_email")
    builder.button(text="🔐 Сменить пароль", callback_data="change_password")
    builder.button(text="📱 Изменить телефон", callback_data="edit_phone")
    builder.button(text="🌍 Изменить язык", callback_data="change_language")
    builder.button(text="🔙 Назад", callback_data="back_to_menu")
    
    builder.adjust(1)
    
    await callback.message.edit_text(
        "👤 Управление профилем:\n\n"
        "Здесь вы можете изменить свои данные.",
        reply_markup=builder.as_markup()
    )
    
    await callback.answer()

# Вертикальные кнопки для настроек
@router.callback_query(F.data == "settings")
async def settings_handler(callback: CallbackQuery):
    builder = InlineKeyboardBuilder()
    
    builder.button(text="🔔 Уведомления", callback_data="notifications")
    builder.button(text="🌙 Тема", callback_data="theme")
    builder.button(text="🔒 Конфиденциальность", callback_data="privacy")
    builder.button(text="📱 Уведомления на телефон", callback_data="push_notifications")
    builder.button(text="📧 Email рассылка", callback_data="email_subscription")
    builder.button(text="🔙 Назад", callback_data="back_to_menu")
    
    builder.adjust(1)
    
    await callback.message.edit_text(
        "⚙️ Настройки:\n\n"
        "Настройте приложение под себя.",
        reply_markup=builder.as_markup()
    )
    
    await callback.answer()

# Вертикальные кнопки для списка игр
@router.callback_query(F.data == "games")
async def games_handler(callback: CallbackQuery):
    games = [
        ("🎮 Крестики-нолики", "game_tictactoe"),
        ("🎲 Кости", "game_dice"),
        ("🃏 Покер", "game_poker"),
        ("🎯 Дартс", "game_darts"),
        ("🏀 Баскетбол", "game_basketball"),
        ("⚽ Футбол", "game_football"),
    ]
    
    builder = InlineKeyboardBuilder()
    
    for game_name, game_callback in games:
        builder.button(text=game_name, callback_data=game_callback)
    
    builder.button(text="🔙 Назад", callback_data="back_to_menu")
    builder.adjust(1)
    
    await callback.message.edit_text(
        "🎮 Доступные игры:\n\n"
        "Выберите игру для начала:",
        reply_markup=builder.as_markup()
    )
    
    await callback.answer()

# Обработка кнопки "Назад"
@router.callback_query(F.data == "back_to_menu")
async def back_to_menu_handler(callback: CallbackQuery):
    await callback.answer()
    # Возвращаемся к главному меню
    await start_command(callback.message)

# Запуск бота
async def main():
    bot = Bot(token="YOUR_BOT_TOKEN")
    dp = Dispatcher()
    dp.include_router(router)
    
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
```

## 🔧 **Дополнительные настройки вертикальных клавиатур**

### **Добавление разделителей между кнопками**
```python
@router.message(Command("advanced"))
async def advanced_menu(message: Message):
    builder = InlineKeyboardBuilder()
    
    # Основные кнопки
    builder.button(text="📊 Профиль", callback_data="profile")
    builder.button(text="⚙️ Настройки", callback_data="settings")
    
    # Разделитель (пустая строка)
    builder.button(text="────────────", callback_data="no_action")
    
    # Дополнительные кнопки
    builder.button(text="💳 Баланс", callback_data="balance")
    builder.button(text="📞 Поддержка", callback_data="support")
    
    # Еще разделитель
    builder.button(text="────────────", callback_data="no_action")
    
    # Системные кнопки
    builder.button(text="📚 Справка", callback_data="help")
    builder.button(text="🔙 Выход", callback_data="exit")
    
    # Делаем некликабельную кнопку-разделитель
    builder.adjust(1)
    
    keyboard = builder.as_markup()
    
    # Делаем разделители некликабельными
    for row in keyboard.inline_keyboard:
        for button in row:
            if button.text == "────────────":
                button.callback_data = None
    
    await message.answer(
        "📱 Расширенное меню:",
        reply_markup=keyboard
    )
```

### **Вертикальные кнопки с иконками и форматированием**
```python
@router.message(Command("styled"))
async def styled_menu(message: Message):
    builder = InlineKeyboardBuilder()
    
    # Кнопки с эмодзи и форматированием
    styled_buttons = [
        ("✅ Активировать подписку", "activate_subscription"),
        ("🌟 Премиум доступ", "premium_access"),
        ("📅 История платежей", "payment_history"),
        ("🎁 Бонусная программа", "bonus_program"),
        ("👑 VIP статус", "vip_status"),
        ("🔔 Уведомления", "notifications"),
    ]
    
    for text, callback in styled_buttons:
        builder.button(text=text, callback_data=callback)
    
    builder.adjust(1)
    
    await message.answer(
        "✨ *Стилизованное меню*\n\n"
        "Все кнопки расположены вертикально для удобства использования.",
        parse_mode="Markdown",
        reply_markup=builder.as_markup()
    )
```

## 🎯 **Ключевые моменты для вертикального расположения:**

1. **`.adjust(1)`** - основной метод для вертикальных кнопок
2. **Каждый список в `inline_keyboard`** - это отдельная строка
3. **`InlineKeyboardBuilder`** предпочтительнее для сложных меню
4. **Разделители** можно создавать с помощью неактивных кнопок

Теперь вы знаете все способы создания вертикальных InlineKeyboard в aiogram!

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
