from flask import Flask, request
from telegram import Bot, Update
from telegram.ext import Dispatcher, CommandHandler, CallbackContext

TOKEN = "8007150171:AAEaGH7Y-MkCdb6lISZTo3HfYadXiZ2V5yQ"

app = Flask(__name__)
bot = Bot(token=TOKEN)
dispatcher = Dispatcher(bot, None, workers=0, use_context=True)

# Data storage
session = {
    "round": 0,
    "balance": 0,
    "max_rounds": 50,
    "bet": 1000,
    "win_amount": 34000
}

def start(update: Update, context: CallbackContext):
    session["round"] = 0
    session["balance"] = 0
    update.message.reply_text("✅ Zero Sniper Bot started!\nUse /hit or /miss to record rounds.")

def hit(update: Update, context: CallbackContext):
    if session["round"] >= session["max_rounds"]:
        update.message.reply_text("❌ You've reached the max 50 rounds.")
        return
    session["round"] += 1
    session["balance"] += session["win_amount"] - session["bet"]
    update.message.reply_text(
        f"💥 Hit recorded!\nRound: {session['round']}\n+34,000\nBalance: {session['balance']}$"
    )

def miss(update: Update, context: CallbackContext):
    if session["round"] >= session["max_rounds"]:
        update.message.reply_text("❌ You've reached the max 50 rounds.")
        return
    session["round"] += 1
    session["balance"] -= session["bet"]
    update.message.reply_text(
        f"❌ Miss recorded.\nRound: {session['round']}\n–1,000\nBalance: {session['balance']}$"
    )

def status(update: Update, context: CallbackContext):
    update.message.reply_text(
        f"📊 Round: {session['round']}/50\n💰 Balance: {session['balance']}$"
    )

def reset(update: Update, context: CallbackContext):
    session["round"] = 0
    session["balance"] = 0
    update.message.reply_text("🔁 Bot has been reset.")

# Handlers
dispatcher.add_handler(CommandHandler("start", start))
dispatcher.add_handler(CommandHandler("hit", hit))
dispatcher.add_handler(CommandHandler("miss", miss))
dispatcher.add_handler(CommandHandler("status", status))
dispatcher.add_handler(CommandHandler("reset", reset))

@app.route(f"/{TOKEN}", methods=["POST"])
def webhook():
    update = Update.de_json(request.get_json(force=True), bot)
    dispatcher.process_update(update)
    return "ok"

@app.route("/")
def home():
    return "Bot is running."

if __name__ == "__main__":
    app.run(port=10000)
