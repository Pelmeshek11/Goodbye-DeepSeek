Aiocrypto (Crypto Pay API) - создание инвойсов

🔗 Установка и настройка

```bash
pip install aiocrypto
```

📦 Основные функции aiocrypto

1. Инициализация клиента

```python
from aiocrypto import CryptoPay, InvoiceStatus

# Создание клиента
crypto = CryptoPay(
    api_token="ВАШ_ТОКЕН",  # Получить у @CryptoBot
    network="testnet"  # или "mainnet"
)
```

2. Создание инвойсов

Простой инвойс

```python
async def create_simple_invoice():
    """Создание простого инвойса"""
    invoice = await crypto.create_invoice(
        asset="USDT",  # USDT, BTC, ETH, TON, TRX, LTC, BNB
        amount=10.5,  # Сумма
        description="Оплата за услугу"
    )
    return invoice

# Пример ответа:
# {
#     "invoice_id": 123456,
#     "pay_url": "https://t.me/CryptoBot?start=invoice_123",
#     "bot_invoice_url": "https://t.me/CryptoBot?start=invoice_123",
#     "hash": "abc123...",
#     "status": "active"
# }
```

Инвойс с дополнительными параметрами

```python
async def create_advanced_invoice():
    """Создание инвойса с дополнительными параметрами"""
    invoice = await crypto.create_invoice(
        asset="TON",
        amount=5.0,
        description="Подписка на месяц",
        paid_btn_name="viewItem",  # callback, openChannel, openChannel, viewItem
        paid_btn_url="https://t.me/your_channel",  # URL для кнопки после оплаты
        payload="user_123_order_456",  # Полезные данные для верификации
        allow_comments=True,  # Разрешить комментарии при оплате
        allow_anonymous=False,  # Запретить анонимные платежи
        expires_in=3600  # Время жизни в секундах (1 час)
    )
    return invoice
```

Инвойс в рублях (автоконвертация)

```python
async def create_invoice_in_rub():
    """Создание инвойса с фиксированной суммой в рублях"""
    invoice = await crypto.create_invoice(
        asset="USDT",  # Любой доступный актив
        amount=1000.0,  # Сумма в рублях
        fiat="RUB",  # Валюта для отображения цены
        accepted_assets=["USDT", "TON", "BTC"]  # Какими криптовалютами можно оплатить
    )
    return invoice
```

3. Получение информации об инвойсах

```python
async def get_invoice_info():
    """Получение информации об инвойсе"""
    # Получить по invoice_id
    invoice = await crypto.get_invoice(
        invoice_id=123456
    )
    
    # Получить по hash
    invoice = await crypto.get_invoice(
        invoice_hash="abc123..."
    )
    
    # Получить список инвойсов
    invoices = await crypto.get_invoices(
        offset=0,  # Смещение
        count=10,  # Количество
        asset="USDT",  # Фильтр по активу
        invoice_ids=[123, 456, 789],  # Конкретные ID
        status="active",  # Фильтр по статусу
        from_date=datetime(2024, 1, 1),  # Начиная с даты
        to_date=datetime(2024, 1, 31)  # До даты
    )
    
    return invoice
```

4. Проверка статуса инвойса

```python
async def check_invoice_status(invoice_id: int):
    """Проверка статуса инвойса"""
    invoice = await crypto.get_invoice(invoice_id)
    
    if invoice.status == InvoiceStatus.PAID:
        return "✅ Оплачен"
    elif invoice.status == InvoiceStatus.ACTIVE:
        return "⏳ Ожидает оплаты"
    elif invoice.status == InvoiceStatus.EXPIRED:
        return "❌ Просрочен"
    elif invoice.status == InvoiceStatus.EXCHANGED:
        return "💱 Обменян"
```

5. Верификация платежа по payload

```python
async def verify_payment(payload: str):
    """Проверка платежа по payload"""
    # Получаем все инвойсы с этим payload
    invoices = await crypto.get_invoices(
        count=100,
        status="paid"  # Только оплаченные
    )
    
    # Ищем инвойс с нашим payload
    for invoice in invoices.items:
        if invoice.payload == payload:
            return {
                "verified": True,
                "invoice": invoice,
                "amount": invoice.amount,
                "asset": invoice.asset
            }
    
    return {"verified": False}
```

🎯 Полный пример использования с aiogram

1. Класс для работы с криптоплатежами

```python
from aiogram import Bot, Router, F
from aiogram.types import Message, CallbackQuery, InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.filters import Command
from aiocrypto import CryptoPay, InvoiceStatus
from datetime import datetime
import asyncio

router = Router()

class CryptoPayments:
    def __init__(self, api_token: str, network: str = "testnet"):
        self.crypto = CryptoPay(api_token, network)
    
    async def create_payment(self, user_id: int, amount: float, product: str):
        """Создание платежа"""
        invoice = await self.crypto.create_invoice(
            asset="USDT",
            amount=amount,
            description=f"Покупка: {product}",
            payload=f"user_{user_id}_product_{product}",
            paid_btn_name="openChannel",
            paid_btn_url="https://t.me/your_channel",
            expires_in=3600
        )
        
        return {
            "invoice_id": invoice.invoice_id,
            "pay_url": invoice.bot_invoice_url,
            "hash": invoice.hash
        }
    
    async def check_payment(self, invoice_id: int) -> dict:
        """Проверка статуса платежа"""
        invoice = await self.crypto.get_invoice(invoice_id)
        
        return {
            "status": invoice.status,
            "paid_at": invoice.paid_at if hasattr(invoice, 'paid_at') else None,
            "amount": invoice.amount,
            "asset": invoice.asset
        }

# Инициализация
crypto_payments = CryptoPayments(
    api_token="ВАШ_ТОКЕН",
    network="testnet"  # Для продакшена используйте "mainnet"
)
```

2. Хендлеры для бота

```python
@router.message(Command("buy"))
async def cmd_buy(message: Message):
    """Создание инвойса для покупки"""
    user_id = message.from_user.id
    
    # Создаем инвойс на 10 USDT
    payment = await crypto_payments.create_payment(
        user_id=user_id,
        amount=10.0,
        product="premium_subscription"
    )
    
    # Создаем клавиатуру с кнопкой оплаты
    keyboard = InlineKeyboardMarkup(
        inline_keyboard=[
            [
                InlineKeyboardButton(
                    text="💳 Оплатить 10 USDT",
                    url=payment['pay_url']
                )
            ],
            [
                InlineKeyboardButton(
                    text="✅ Проверить оплату",
                    callback_data=f"check_{payment['invoice_id']}"
                )
            ]
        ]
    )
    
    await message.answer(
        "💎 Для оплаты нажмите на кнопку ниже:\n\n"
        f"ID платежа: `{payment['invoice_id']}`\n"
        "После оплаты нажмите 'Проверить оплату'",
        reply_markup=keyboard,
        parse_mode="Markdown"
    )

@router.callback_query(F.data.startswith("check_"))
async def check_payment(callback: CallbackQuery):
    """Проверка статуса оплаты"""
    invoice_id = int(callback.data.split("_")[1])
    
    payment_info = await crypto_payments.check_payment(invoice_id)
    
    if payment_info["status"] == InvoiceStatus.PAID:
        await callback.message.edit_text(
            f"✅ Оплата подтверждена!\n\n"
            f"Сумма: {payment_info['amount']} {payment_info['asset']}\n"
            f"Время: {payment_info['paid_at']}\n\n"
            "Спасибо за покупку!"
        )
        
        # Здесь можно выдать товар/услугу
        await grant_access(callback.from_user.id)
        
    else:
        await callback.answer(
            "⚠️ Платеж еще не поступил. Попробуйте через минуту.",
            show_alert=True
        )
```

3. Система проверки платежей по таймеру

```python
async def payment_monitor(bot: Bot):
    """Фоновая проверка платежей"""
    while True:
        try:
            # Получаем активные инвойсы
            invoices = await crypto_payments.crypto.get_invoices(
                status="active",
                count=50
            )
            
            for invoice in invoices.items:
                # Проверяем каждый инвойс
                current_status = await crypto_payments.check_payment(invoice.invoice_id)
                
                if current_status["status"] == InvoiceStatus.PAID:
                    # Логика обработки оплаченного инвойса
                    await process_paid_invoice(invoice)
                    
        except Exception as e:
            print(f"Ошибка мониторинга: {e}")
        
        await asyncio.sleep(60)  # Проверка каждую минуту

async def process_paid_invoice(invoice):
    """Обработка оплаченного инвойса"""
    # Извлекаем user_id из payload
    payload = invoice.payload
    if payload and payload.startswith("user_"):
        user_id = int(payload.split("_")[1])
        
        # Здесь логика выдачи товара
        print(f"Пользователь {user_id} оплатил {invoice.amount} {invoice.asset}")
        
        # Можно отправить уведомление в боте
        # await bot.send_message(user_id, "Спасибо за оплату!")
```

4. Получение статистики и баланса

```python
@router.message(Command("stats"))
async def get_stats(message: Message):
    """Получение статистики платежей"""
    if message.from_user.id != АДМИН_ID:
        return
    
    # Получаем баланс
    balance = await crypto_payments.crypto.get_balance()
    
    # Получаем последние платежи
    invoices = await crypto_payments.crypto.get_invoices(
        count=10,
        status="paid"
    )
    
    total_received = sum(invoice.amount for invoice in invoices.items)
    
    stats_text = "📊 *Статистика платежей*\n\n"
    stats_text += f"💰 Баланс:\n"
    
    for item in balance:
        stats_text += f"  • {item.balance:.2f} {item.currency_code}\n"
    
    stats_text += f"\n🔄 Последние 10 платежей:\n"
    
    for invoice in invoices.items:
        paid_time = invoice.paid_at.strftime("%d.%m.%Y %H:%M") if invoice.paid_at else "N/A"
        stats_text += f"  • {invoice.amount} {invoice.asset} - {paid_time}\n"
    
    stats_text += f"\n📈 Всего получено: {total_received} USDT"
    
    await message.answer(stats_text, parse_mode="Markdown")
```

5. Обработка вебхуков (рекомендуемый способ)

```python
from aiogram import Dispatcher
from aiocrypto import Update, UpdateType

@router.message(Command("set_webhook"))
async def set_webhook(message: Message):
    """Установка вебхука для мгновенных уведомлений"""
    webhook_url = "https://ваш-сайт.com/crypto-webhook"
    
    await crypto_payments.crypto.set_webhook(
        url=webhook_url,
        allowed_updates=[
            UpdateType.INVOICE_PAID,
            UpdateType.INVOICE_CREATED
        ]
    )
    
    await message.answer("✅ Вебхук установлен")

# Эндпоинт для вебхука (используйте aiohttp)
async def crypto_webhook(request):
    """Обработчик вебхуков от Crypto Pay"""
    data = await request.json()
    update = Update(**data)
    
    if update.update_type == UpdateType.INVOICE_PAID:
        invoice = update.payload
        
        # Обработка оплаченного инвойса
        await process_webhook_payment(invoice)
        
        return web.Response(text="OK")
```

🔧 Полезные утилиты

```python
async def get_exchange_rate(asset: str, fiat: str = "RUB"):
    """Получить курс обмена"""
    rates = await crypto_payments.crypto.get_exchange_rates()
    
    for rate in rates:
        if rate.source == asset and rate.target == fiat:
            return rate.rate
    
    return None

async def create_subscription_invoice(user_id: int, plan: str):
    """Создание инвойса для подписки"""
    plans = {
        "basic": 5.0,
        "premium": 15.0,
        "vip": 30.0
    }
    
    amount = plans.get(plan, 5.0)
    
    invoice = await crypto_payments.crypto.create_invoice(
        asset="USDT",
        amount=amount,
        description=f"Подписка {plan}",
        payload=f"subscription_{user_id}_{plan}",
        paid_btn_name="openChannel",
        paid_btn_url="https://t.me/your_channel",
        allow_comments=True
    )
    
    return invoice
```

⚠️ Важные моменты

1. Тестовый режим: Всегда тестируйте на testnet перед использованием в продакшене
2. Payload: Используйте уникальные payload для идентификации платежей
3. Верификация: Всегда проверяйте платежи через API, не доверяйте данным от пользователя
4. Безопасность: Храните API токен в безопасном месте (переменные окружения)
5. Логирование: Ведите логи всех транзакций для отладки

📝 Пример .env файла

```env
CRYPTO_PAY_TOKEN=123456:ABC-DEF123456ghIkl-zyx57W2v1u123ew11
CRYPTO_NETWORK=testnet  # или mainnet
ADMIN_ID=123456789
```
