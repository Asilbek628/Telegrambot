# filename: currency_bot_v13.py
import requests
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters

# BotFather'dan tokenni shu yerga yozing
BOT_TOKEN = "8196872190:AAFRIIKijYzoZdomaKAFYZU3YOW4sJ7wSKE"

# Supported currencies
SUPPORTED = ["USD","EUR","GBP","JPY","CNY","RUB","UZS","KZT","TRY",
             "INR","AUD","CAD","CHF","SEK","NOK","DKK","SGD","AED","MYR","IDR"]

API_URL = "https://api.exchangerate.host/convert"  # bepul, API kalitsiz ishlaydi

def start(update, context):
    update.message.reply_text(
        "Salom! Men valyuta konvertor botman.\n"
        "Foydalanish: /convert <miqdor> <from_currency> <to_currency>\n"
        "Masalan: /convert 100 USD EUR\n"
        "Yoki: 100 USD to EUR\n\n"
        "Qo'llab-quvvatlanadigan valyutalar: " + ", ".join(SUPPORTED)
    )

def list_cmd(update, context):
    update.message.reply_text("Qo'llab-quvvatlanadigan valyutalar:\n" + ", ".join(SUPPORTED))

def convert_amount(amount, frm, to):
    params = {"from": frm, "to": to, "amount": amount}
    r = requests.get(API_URL, params=params, timeout=10)
    r.raise_for_status()
    data = r.json()
    if data.get("success", True) is False:
        raise ValueError("API xatolik")
    return data["result"]

def convert_command(update, context):
    try:
        args = context.args
        if len(args) != 3:
            update.message.reply_text("Xato: Foydalanish: /convert <miqdor> <from> <to>\nMasalan: /convert 100 USD EUR")
            return

        amount = float(args[0].replace(",", "."))
        frm = args[1].upper()
        to = args[2].upper()

        if frm not in SUPPORTED or to not in SUPPORTED:
            update.message.reply_text("Kechirasiz, qo'llab-quvvatlanmaydigan valyuta. Ro'yxat uchun /list ni bosing.")
            return

        result = convert_amount(amount, frm, to)
        update.message.reply_text(f"{amount} {frm} = {result:.4f} {to}")

    except Exception as e:
        update.message.reply_text("Xatolik: " + str(e))

def text_handler(update, context):
    # Qabul qiladigan formatlar: "100 USD to EUR" yoki "100 USD EUR"
    txt = update.message.text.strip()
    parts = txt.replace(" to ", " ").split()
    if len(parts) == 3:
        try:
            amount = float(parts[0].replace(",", "."))
            frm = parts[1].upper()
            to = parts[2].upper()
            if frm not in SUPPORTED or to not in SUPPORTED:
                update.message.reply_text("Qo'llab-quvvatlanmaydigan valyuta. /list")
                return
            result = convert_amount(amount, frm, to)
            update.message.reply_text(f"{amount} {frm} = {result:.4f} {to}")
        except Exception as e:
            update.message.reply_text("Parsing yoki API xatosi: " + str(e))
    else:
        update.message.reply_text("Foydalanish: /convert 100 USD EUR yoki 100 USD to EUR")

def main():
    updater = Updater(BOT_TOKEN, use_context=True)
    dp = updater.dispatcher

    dp.add_handler(CommandHandler("start", start))
    dp.add_handler(CommandHandler("list", list_cmd))
    dp.add_handler(CommandHandler("convert", convert_command))
    dp.add_handler(MessageHandler(Filters.text & ~Filters.command, text_handler))

    updater.start_polling()
    print("Bot ishga tushdi...")
    updater.idle()

if name == "main":
    main()
