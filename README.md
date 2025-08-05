import logging
from aiogram import Bot, Dispatcher, types
from aiogram.utils import executor
from aiogram.types import ReplyKeyboardMarkup, KeyboardButton
import os

# گرفتن توکن و رمز از متغیرهای محیطی
BOT_TOKEN = os.getenv("BOT_TOKEN")
ADMIN_ID = int(os.getenv("ADMIN_ID"))
ACCESS_PASSWORD = os.getenv("ACCESS_PASSWORD")

bot = Bot(token=BOT_TOKEN)
dp = Dispatcher(bot)

# حافظه ساده برای ذخیره کیف‌پول‌ها
user_wallets = {}

# کیبورد استارت
start_keyboard = ReplyKeyboardMarkup(resize_keyboard=True)
start_keyboard.add(KeyboardButton("🚀 استارت"))

@dp.message_handler(commands=['start'])
async def handle_start(message: types.Message):
    await message.reply("رمز عبور رو وارد کن:", reply_markup=types.ReplyKeyboardRemove())

@dp.message_handler(lambda message: message.chat.id not in user_wallets)
async def handle_password(message: types.Message):
    if message.text == ACCESS_PASSWORD:
        user_wallets[message.chat.id] = {"step": "waiting_for_address"}
        await message.reply("✅ ورود موفق بود!\n\nلطفاً آدرس کیف‌پول رو وارد کن.")
    else:
        await message.reply("❌ رمز عبور اشتباهه!")

@dp.message_handler(lambda message: user_wallets.get(message.chat.id, {}).get("step") == "waiting_for_address")
async def handle_wallet_address(message: types.Message):
    user_wallets[message.chat.id]["address"] = message.text
    user_wallets[message.chat.id]["step"] = "waiting_for_name"
    await message.reply("حالا یه اسم برای این کیف‌پول انتخاب کن.")

@dp.message_handler(lambda message: user_wallets.get(message.chat.id, {}).get("step") == "waiting_for_name")
async def handle_wallet_name(message: types.Message):
    user_wallets[message.chat.id]["name"] = message.text
    user_wallets[message.chat.id]["step"] = "done"
    await message.reply(f"🎉 کیف‌پول با موفقیت ذخیره شد!\n\n📌 آدرس: {user_wallets[message.chat.id]['address']}\n📛 اسم: {user_wallets[message.chat.id]['name']}")

if __name__ == '__main__':
    logging.basicConfig(level=logging.INFO)
    executor.start_polling(dp, skip_updates=True)6
