import telebot
import datetime
import time
import threading
import random
from googlesearch import search

bot = telebot.TeleBot('6387690359:AAGG4VTh5b4DV7yZDlHW-a_s-_H36wZToH4')
# Функция для получения курса валюты
def get_currency_exchange_rate(base_currency, target_currency):
    return random.uniform(0.5, 2.0)
# Функция для генерации пароля
def generate_password(length=8):
    characters = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
    return ''.join(random.choice(characters) for i in range(length))

# Функция для обработки сообщений
@bot.message_handler(commands=['start'])
def start_message(message):
    bot.reply_to(message, 'Привет, {0.first_name}! Напиши /help для списка команд'.format(message.from_user))
    reminder_thread = threading.Thread(target=send_reminders, args=(message.chat.id,))
    reminder_thread.start()

# Функция для отправки напоминаний
def send_reminders(chat_id):
    while True:
        now = datetime.datetime.now().strftime("%H:%M")
        for time_text, reminder_text in reminders:
            if now == time_text:
                bot.send_message(chat_id, reminder_text)
                reminders.remove((time_text, reminder_text))
        time.sleep(60)

# Функция для помощи
@bot.message_handler(commands=["help"])
def help_message(message):
    bot.reply_to(message, """Доступные команды:
/start - запуск бота,
/help - помощь по командам, 
/fact - случайный факт,
/set_reminder время_напоминания текст_напоминания,
/exchange_currency базовая_валюта целевая_валюта - конвертер валюты,
/generate_password [длина] - генератор паролей""")

# Функция для случайного факта
@bot.message_handler(commands=["fact"])
def fact_message(message):
    facts = [
        "Вода на Земле может существовать в трех основных состояниях: жидком, твердом и газообразном.",
        "Человеческое сердце может превзойти любой мотор: за один день оно совершает около 100 000 сокращений.",
        "Океаны Земли содержат около 20 миллионов тонн золота, но его разведка на глубинах делает его экономически невыгодным для извлечения."
    ]
    random_fact = random.choice(facts)
    bot.reply_to(message, f"Лови рандомный факт: {random_fact}")

# Функция для конвертации валют
@bot.message_handler(commands=["exchange_currency"])
def exchange_currency_message(message):
    try:
        base_currency, target_currency = message.text.split(maxsplit=2)[1:]
        exchange_rate = get_currency_exchange_rate(base_currency, target_currency)
        bot.reply_to(message, f"Курс {base_currency}/{target_currency}: {exchange_rate}")
    except:
        bot.reply_to(message, "Используйте команду в формате /exchange_currency базовая_валюта целевая_валюта")
# Функция для генерации пароля
@bot.message_handler(commands=["generate_password"])
def generate_password_message(message):
    try:
        length = int(message.text.split(maxsplit=1)[1])
    except:
        length = 8

    password = generate_password(length)
    bot.reply_to(message, f"Сгенерированный пароль: {password}")

# Функция для отправки напоминаний
@bot.message_handler(commands=["set_reminder"])
def set_reminder_message(message):
    try:
        command_parts = message.text.split(maxsplit=2)
        time_text = command_parts[1]
        reminder_text = command_parts[2]
        if ':' not in time_text or len(time_text.split(':')) != 2:
            raise ValueError("Invalid time format")
        reminders.append((time_text, reminder_text))
        bot.reply_to(message, "Напоминание установлено!")
    except IndexError:
        bot.reply_to(message, "Используйте команду в формате /set_reminder время_напоминания текст_напоминания")
    except ValueError as e:
        bot.reply_to(message, str(e))

# Список напоминаний
reminders = []

# Запуск обработки сообщений в телеграмме
bot.polling(none_stop=True)
