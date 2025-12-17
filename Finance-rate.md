Функция получения курса с Binance без Decimal

```python
import asyncio
import aiohttp
import json
from typing import Optional, Dict, Any
from datetime import datetime
import logging

# Настройка логирования
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class BinancePriceFetcher:
    """Класс для получения курса с Binance API"""
    
    def __init__(self, timeout: int = 10):
        """
        Args:
            timeout: Таймаут запросов в секундах
        """
        self.base_url = "https://api.binance.com"
        self.timeout = aiohttp.ClientTimeout(total=timeout)
        self.session = None
    
    async def __aenter__(self):
        """Создание сессии при входе в контекст"""
        self.session = aiohttp.ClientSession(timeout=self.timeout)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """Закрытие сессии при выходе из контекста"""
        if self.session:
            await self.session.close()
    
    async def get_price_simple(self, symbol: str) -> float:
        """
        Получить простой курс монеты в USDT
        
        Args:
            symbol: Название монеты (BTC, ETH, SOL и т.д.)
            
        Returns:
            float: Курс в USDT
        """
        # Нормализация символа
        symbol = symbol.upper().strip()
        if not symbol.endswith('USDT'):
            symbol = f"{symbol}USDT"
        
        try:
            url = f"{self.base_url}/api/v3/ticker/price?symbol={symbol}"
            
            async with self.session.get(url) as response:
                if response.status != 200:
                    raise Exception(f"API вернул статус {response.status}")
                
                data = await response.json()
                
                # Возвращаем просто float значение цены
                return float(data['price'])
                
        except aiohttp.ClientError as e:
            logger.error(f"Ошибка соединения с Binance: {e}")
            raise Exception(f"Не удалось подключиться к Binance: {e}")
        except json.JSONDecodeError as e:
            logger.error(f"Ошибка парсинга JSON: {e}")
            raise Exception("Некорректный ответ от API")
        except KeyError as e:
            logger.error(f"Ошибка в структуре данных: {e}")
            raise Exception("Некорректная структура данных от API")
        except Exception as e:
            logger.error(f"Неизвестная ошибка: {e}")
            raise Exception(f"Ошибка получения курса: {e}")
    
    async def get_price_with_info(self, symbol: str) -> dict:
        """
        Получить курс с дополнительной информацией
        
        Args:
            symbol: Название монеты
            
        Returns:
            dict: Словарь с информацией о курсе
        """
        symbol = symbol.upper().strip()
        if not symbol.endswith('USDT'):
            symbol = f"{symbol}USDT"
        
        try:
            url = f"{self.base_url}/api/v3/ticker/24hr?symbol={symbol}"
            
            async with self.session.get(url) as response:
                if response.status != 200:
                    raise Exception(f"API вернул статус {response.status}")
                
                data = await response.json()
                
                # Формируем простой словарь без Decimal
                return {
                    'symbol': data['symbol'].replace('USDT', ''),
                    'price': float(data['lastPrice']),
                    'change_percent': float(data['priceChangePercent']),
                    'high_24h': float(data['highPrice']),
                    'low_24h': float(data['lowPrice']),
                    'volume': float(data['volume']),
                    'timestamp': datetime.now().isoformat(),
                    'bid': float(data['bidPrice']),
                    'ask': float(data['askPrice']),
                    'source': 'Binance'
                }
                
        except Exception as e:
            logger.error(f"Ошибка получения полной информации: {e}")
            raise Exception(f"Ошибка получения данных: {e}")
    
    async def get_multiple_prices(self, symbols: list) -> dict:
        """
        Получить курсы нескольких монет
        
        Args:
            symbols: Список символов монет
            
        Returns:
            dict: Словарь {символ: курс}
        """
        results = {}
        
        # Формируем список пар для запроса
        pairs = []
        for symbol in symbols:
            clean_symbol = symbol.upper().strip()
            if not clean_symbol.endswith('USDT'):
                clean_symbol = f"{clean_symbol}USDT"
            pairs.append(clean_symbol)
        
        try:
            # Получаем все цены одним запросом
            url = f"{self.base_url}/api/v3/ticker/price"
            
            async with self.session.get(url) as response:
                if response.status != 200:
                    raise Exception(f"API вернул статус {response.status}")
                
                all_prices = await response.json()
                
                # Преобразуем в словарь для быстрого поиска
                price_dict = {item['symbol']: float(item['price']) for item in all_prices}
                
                # Формируем результат для запрошенных символов
                for original_symbol, pair in zip(symbols, pairs):
                    if pair in price_dict:
                        results[original_symbol.upper()] = price_dict[pair]
                    else:
                        logger.warning(f"Пара {pair} не найдена на Binance")
                        results[original_symbol.upper()] = None
                
                return results
                
        except Exception as e:
            logger.error(f"Ошибка получения нескольких курсов: {e}")
            # Возвращаем None для всех символов при ошибке
            return {symbol.upper(): None for symbol in symbols}


# ========== ПРОСТЫЕ ФУНКЦИИ БЕЗ КЛАССА ==========

async def get_binance_price(symbol: str) -> float:
    """
    Простая функция для получения курса с Binance
    
    Args:
        symbol: Символ криптовалюты (BTC, ETH, SOL и т.д.)
        
    Returns:
        float: Курс в USDT
    """
    symbol = symbol.upper().strip()
    if not symbol.endswith('USDT'):
        symbol = f"{symbol}USDT"
    
    async with aiohttp.ClientSession(timeout=aiohttp.ClientTimeout(total=10)) as session:
        try:
            url = f"https://api.binance.com/api/v3/ticker/price?symbol={symbol}"
            
            async with session.get(url) as response:
                if response.status == 200:
                    data = await response.json()
                    return float(data['price'])
                else:
                    raise Exception(f"Статус код: {response.status}")
                    
        except Exception as e:
            raise Exception(f"Ошибка получения курса {symbol}: {str(e)}")


async def get_binance_price_safe(symbol: str, fallback_value: float = None) -> float:
    """
    Безопасное получение курса с возвратом fallback значения при ошибке
    
    Args:
        symbol: Символ криптовалюты
        fallback_value: Значение, которое вернуть при ошибке
        
    Returns:
        float: Курс или fallback значение
    """
    try:
        return await get_binance_price(symbol)
    except Exception as e:
        logger.warning(f"Не удалось получить курс {symbol}: {e}")
        return fallback_value


async def get_binance_prices_batch(symbols: list) -> dict:
    """
    Пакетное получение курсов нескольких монет
    
    Args:
        symbols: Список символов
        
    Returns:
        dict: {символ: курс}
    """
    results = {}
    tasks = []
    
    for symbol in symbols:
        task = get_binance_price_safe(symbol)
        tasks.append(task)
    
    # Запускаем все задачи параллельно
    prices = await asyncio.gather(*tasks, return_exceptions=True)
    
    for symbol, price in zip(symbols, prices):
        if isinstance(price, float):
            results[symbol.upper()] = price
        else:
            results[symbol.upper()] = None
    
    return results


# ========== КЭШИРОВАННЫЙ ПОЛУЧАТЕЛЬ ==========

class CachedBinancePrices:
    """Кэшированный получатель цен с Binance"""
    
    def __init__(self, cache_time: int = 30):
        """
        Args:
            cache_time: Время кэширования в секундах
        """
        self.cache_time = cache_time
        self.cache = {}
        self.cache_timestamps = {}
    
    async def get_price_cached(self, symbol: str) -> float:
        """
        Получить курс с кэшированием
        
        Args:
            symbol: Символ криптовалюты
            
        Returns:
            float: Курс в USDT
        """
        symbol_key = symbol.upper()
        current_time = datetime.now().timestamp()
        
        # Проверяем, есть ли в кэше и не устарел ли
        if (symbol_key in self.cache and 
            symbol_key in self.cache_timestamps and
            current_time - self.cache_timestamps[symbol_key] < self.cache_time):
            
            return self.cache[symbol_key]
        
        # Если нет в кэше или устарел, получаем свежий курс
        try:
            price = await get_binance_price(symbol)
            self.cache[symbol_key] = price
            self.cache_timestamps[symbol_key] = current_time
            return price
        except Exception as e:
            # Если ошибка, но есть в кэше, вернем кэшированное значение
            if symbol_key in self.cache:
                logger.warning(f"Используем кэшированный курс для {symbol}: {e}")
                return self.cache[symbol_key]
            raise e
    
    def clear_cache(self):
        """Очистить кэш"""
        self.cache.clear()
        self.cache_timestamps.clear()


# ========== ПРИМЕРЫ ИСПОЛЬЗОВАНИЯ ==========

async def example_simple():
    """Пример простого использования"""
    try:
        # Получить курс BTC
        btc_price = await get_binance_price("BTC")
        print(f"BTC: {btc_price:.2f} USDT")
        
        # Получить курс ETH
        eth_price = await get_binance_price("ETH")
        print(f"ETH: {eth_price:.2f} USDT")
        
    except Exception as e:
        print(f"Ошибка: {e}")


async def example_with_class():
    """Пример использования класса"""
    async with BinancePriceFetcher() as fetcher:
        # Просто курс
        btc_price = await fetcher.get_price_simple("BTC")
        print(f"Простой курс BTC: {btc_price}")
        
        # Полная информация
        btc_info = await fetcher.get_price_with_info("BTC")
        print(f"Полная информация BTC: {btc_info}")
        
        # Несколько курсов
        symbols = ["BTC", "ETH", "SOL", "BNB"]
        prices = await fetcher.get_multiple_prices(symbols)
        print(f"Курсы нескольких монет: {prices}")


async def example_batch():
    """Пример пакетного получения"""
    symbols = ["BTC", "ETH", "SOL", "ADA", "DOT", "MATIC", "AVAX", "LINK"]
    
    # Пакетное получение
    prices = await get_binance_prices_batch(symbols)
    
    for symbol, price in prices.items():
        if price is not None:
            print(f"{symbol}: {price:.8f} USDT")
        else:
            print(f"{symbol}: Не удалось получить курс")


async def example_cached():
    """Пример с кэшированием"""
    cached_prices = CachedBinancePrices(cache_time=60)  # Кэш на 60 секунд
    
    # Первый запрос - получаем с API
    price1 = await cached_prices.get_price_cached("BTC")
    print(f"Первый запрос BTC: {price1}")
    
    # Второй запрос через 2 секунды - получаем из кэша
    await asyncio.sleep(2)
    price2 = await cached_prices.get_price_cached("BTC")
    print(f"Второй запрос BTC (из кэша): {price2}")
    
    # Ждем больше времени кэша и снова запрашиваем
    await asyncio.sleep(70)
    price3 = await cached_prices.get_price_cached("BTC")
    print(f"Третий запрос BTC (обновленный): {price3}")


# ========== ИНТЕГРАЦИЯ С TELEGRAM БОТОМ ==========

async def get_price_for_telegram(symbol: str) -> str:
    """
    Форматирование курса для Telegram бота
    
    Args:
        symbol: Символ криптовалюты
        
    Returns:
        str: Отформатированное сообщение
    """
    try:
        price = await get_binance_price(symbol)
        return f"💰 *{symbol.upper()}/USDT*\n\n📈 Курс: `{price:.8f}` USDT\n\n🕐 {datetime.now().strftime('%H:%M:%S')}"
    except Exception as e:
        return f"❌ Не удалось получить курс {symbol.upper()}: {str(e)[:100]}"


# ========== ФУНКЦИЯ ДЛЯ ПРЯМОГО ИМПОРТА ==========

async def get_crypto_price(symbol: str, source: str = "binance") -> float:
    """
    Универсальная функция для получения курса криптовалюты
    
    Args:
        symbol: Символ криптовалюты
        source: Источник (пока только binance)
        
    Returns:
        float: Курс в USDT
    """
    if source.lower() != "binance":
        raise ValueError(f"Источник {source} не поддерживается. Используйте 'binance'")
    
    return await get_binance_price(symbol)


# ========== ТЕСТИРОВАНИЕ ==========

async def test_all():
    """Тестирование всех функций"""
    print("Тестирование получения курсов с Binance...\n")
    
    # Тест 1: Простая функция
    print("1. Простая функция:")
    try:
        btc = await get_binance_price("BTC")
        print(f"   BTC: {btc:.2f} USDT ✓")
    except Exception as e:
        print(f"   Ошибка: {e}")
    
    # Тест 2: Пакетное получение
    print("\n2. Пакетное получение:")
    symbols = ["BTC", "ETH", "SOL", "BNB"]
    prices = await get_binance_prices_batch(symbols)
    for symbol, price in prices.items():
        if price:
            print(f"   {symbol}: {price:.2f} USDT ✓")
        else:
            print(f"   {symbol}: Ошибка ✗")
    
    # Тест 3: С кэшированием
    print("\n3. С кэшированием:")
    cached = CachedBinancePrices(cache_time=5)
    eth1 = await cached.get_price_cached("ETH")
    print(f"   Первый запрос ETH: {eth1:.2f}")
    
    await asyncio.sleep(2)
    eth2 = await cached.get_price_cached("ETH")  # Должен быть из кэша
    print(f"   Второй запрос ETH (через 2 сек): {eth2:.2f} {'(из кэша)' if eth1 == eth2 else ''}")
    
    await asyncio.sleep(4)
    eth3 = await cached.get_price_cached("ETH")  # Должен обновиться
    print(f"   Третий запрос ETH (через 6 сек): {eth3:.2f} {'(обновленный)' if eth3 != eth2 else ''}")


# ========== ЗАПУСК ==========

if __name__ == "__main__":
    # Запуск тестов
    asyncio.run(test_all())
    
    # Или просто получить один курс
    # asyncio.run(example_simple())
    
    # Или использовать с классом
    # asyncio.run(example_with_class())
```

🎯 Самые простые варианты (однострочники):

Вариант 1: Абсолютный минимум

```python
import aiohttp
import asyncio

async def get_price(symbol: str) -> float:
    """Простая функция получения курса"""
    url = f"https://api.binance.com/api/v3/ticker/price?symbol={symbol.upper()}USDT"
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            data = await response.json()
            return float(data['price'])

# Использование:
# price = await get_price("BTC")
# print(price)  # 41250.50
```

Вариант 2: С обработкой ошибок

```python
async def get_price_safe(symbol: str) -> float:
    """Безопасное получение курса"""
    try:
        symbol = symbol.upper()
        if not symbol.endswith('USDT'):
            symbol = f"{symbol}USDT"
        
        async with aiohttp.ClientSession() as session:
            async with session.get(f"https://api.binance.com/api/v3/ticker/price?symbol={symbol}") as response:
                data = await response.json()
                return float(data['price'])
    except:
        return 0.0  # Возвращаем 0 при ошибке

# Использование:
# price = await get_price_safe("ETH")
```

Вариант 3: Для синхронного кода

```python
import requests

def get_price_sync(symbol: str) -> float:
    """Синхронная версия для обычного кода"""
    url = f"https://api.binance.com/api/v3/ticker/price?symbol={symbol.upper()}USDT"
    response = requests.get(url)
    data = response.json()
    return float(data['price'])

# Использование:
# price = get_price_sync("SOL")
```

📝 Основные особенности:

1. Только Binance API - используется официальное API Binance
2. Без Decimal - все цены возвращаются как float
3. Асинхронность - работает быстро и не блокирует поток
4. Обработка ошибок - корректная обработка сетевых ошибок
5. Кэширование - опциональное кэширование для снижения нагрузки
6. Простота использования - можно использовать как одну функцию

🚀 Быстрый старт:

```python
# Самый простой способ:
async def main():
    # Просто курс
    btc_price = await get_binance_price("BTC")
    print(f"BTC: {btc_price} USDT")
    
    # Несколько курсов
    prices = await get_binance_prices_batch(["BTC", "ETH", "SOL"])
    for coin, price in prices.items():
        print(f"{coin}: {price} USDT")

# Запуск
import asyncio
asyncio.run(main())
```
