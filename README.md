import logging
import random
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Updater, CommandHandler, CallbackQueryHandler, MessageHandler, Filters, CallbackContext

# Включение логирования
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)

# Котировки
BUY_RATE = 0.65  # Курс покупки
SELL_RATE = 0.60  # Курс продажи

def start(update: Update, context: CallbackContext) -> None:
    keyboard = [
        [InlineKeyboardButton("💰 Покупка голды", callback_data='buy_gold')],
        [InlineKeyboardButton("💸 Продажа голды", callback_data='sell_gold')]
    ]

    reply_markup = InlineKeyboardMarkup(keyboard)
    update.message.reply_text('🎉 Привет! Выберите действие, чтобы продолжить:', reply_markup=reply_markup)

def buy_gold(update: Update, context: CallbackContext) -> None:
    query = update.callback_query
    query.answer()
    context.user_data['action'] = 'buy'
    query.edit_message_text(text=f"📈 Курс покупки голды: 1G = {BUY_RATE} ₽\nВведите сумму, которую хотите купить (в G):")

def sell_gold(update: Update, context: CallbackContext) -> None:
    query = update.callback_query
    query.answer()
    context.user_data['action'] = 'sell'
    query.edit_message_text(text=f"📉 Курс продажи голды: 1G = {SELL_RATE} ₽\nВведите сумму, которую хотите продать (в G):")

def handle_amount(update: Update, context: CallbackContext) -> None:
    action = context.user_data.get('action')
    
    try:
        amount = float(update.message.text)

        if amount <= 0:
            update.message.reply_text("❌ Сумма должна быть больше 0. Пожалуйста, попробуйте снова.")
            return

        # Рассчитываем сумму в ₽
        total_cost = amount * (BUY_RATE if action == 'buy' else SELL_RATE)
        order_number = random.randint(181929933773829191, 999999999999999999)  # Генерация номера заказа

        confirmation_message = f"✅ Ваш заказ #{order_number} оформлен!\n"
        confirmation_message += f"💵 К {'покупке' if action == 'buy' else 'продаже'}: {amount:.2f} G\n"
        confirmation_message += f"💳 Чек: {total_cost:.2f} ₽\n\n"
        confirmation_message += "📞 Связаться с менеджером: [Менеджер](https://t.me/heroestags)\n"
        confirmation_message += "🙏 Спасибо за использование нашего сервиса!"

        reply_markup = InlineKeyboardMarkup([[InlineKeyboardButton("📞 Связаться с менеджером", url='https://t.me/heroestags')]])

        update.message.reply_text(confirmation_message, reply_markup=reply_markup, parse_mode='Markdown')
        
    except ValueError:
        update.message.reply_text("❌ Пожалуйста, введите корректное число.")

def main():
    # Инициализация бота
    updater = Updater("7805128889:AAEHjxHPRtuA36b4Gr0SBt_AsXW-rSFhNAU")

    # Получение диспетчера для регистрации обработчиков
    dispatcher = updater.dispatcher

    # Регистрация обработчиков команд
    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(CallbackQueryHandler(buy_gold, pattern='buy_gold'))
    dispatcher.add_handler(CallbackQueryHandler(sell_gold, pattern='sell_gold'))
    dispatcher.add_handler(MessageHandler(Filters.text & ~Filters.command, handle_amount))

    # Запуск бота
    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
