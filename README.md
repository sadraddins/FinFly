import feedparser
import requests
import asyncio
from aiogram import Bot, Dispatcher
from aiogram.types import InputMediaPhoto

# 👉 Вставь свой токен бота и ID канала
BOT_TOKEN = "8153514298:AAF7-BNxhvrdTtZsh-tuPyJZwww34f3izGE"
CHANNEL_ID = "@FinFly_az"

# 👉 Ссылка на RSS-ленту
RSS_URL = "https://cdn.mysitemapgenerator.com/shareapi/rss/2603963328"

# 👉 Храним уже отправленные новости
sent_links = set()

bot = Bot(token=BOT_TOKEN)
dp = Dispatcher()

async def fetch_rss():
    """Получает новости из RSS и отправляет в канал."""
    global sent_links
    feed = feedparser.parse(RSS_URL)

    for entry in feed.entries[::-1]:  # Перебираем новости в порядке появления
        title = entry.title
        link = entry.link
        description = entry.summary if "summary" in entry else "Без описания"
        
        # Извлекаем изображение, если есть
        image_url = None
        if "media_content" in entry:
            image_url = entry.media_content[0]["url"]
        elif "links" in entry:
            for l in entry.links:
                if l.get("type", "").startswith("image"):
                    image_url = l["href"]
                    break
        
        if link not in sent_links:  # Проверяем, не отправляли ли уже эту новость
            sent_links.add(link)  

            # Отправляем новость в канал
            if image_url:
                media = InputMediaPhoto(media=image_url, caption=f"📰 *{title}*\n\n{description}\n[Читать дальше]({link})", parse_mode="Markdown")
                await bot.send_photo(CHANNEL_ID, media=media.media, caption=media.caption, parse_mode="Markdown")
            else:
                await bot.send_message(CHANNEL_ID, f"📰 *{title}*\n\n{description}\n[Читать дальше]({link})", parse_mode="Markdown")

async def scheduler():
    """Запускает проверку RSS каждые 10 минут."""
    while True:
        await fetch_rss()
        await asyncio.sleep(600)  # 10 минут

async def main():
    """Главная функция запуска бота."""
    await bot.delete_webhook()
    asyncio.create_task(scheduler())  # Запускаем планировщик
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
