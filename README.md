# vk_to_telegram_bot

Бот для пересылки сообщений из VK в Telegram

Заполняем settings.ini своими значениями

где:

`last_id` - последний ID сообщения ленты вконтакте, можно оставить 123  
`include_link` - вставлять ли в конце сообщения ссылку на пост
`preview_link` - включить ли предпросмотр ссылок
`login` - ваш логин аккаутна вконтакте  
`password` - ваш пароль аккаунта вконтакте  
`domain` - группа или id сообщества вконтакте  
`count` - количество последних забираемых сообщений сообщество (ограничение API вконтакте, максимум 100 за один запрос)  
`bot_token` - токен бота полученный выше  
`channel` - название канала в телеграмме для публикации ботом, важно помнить что для публикации сообщений в канале ботом, его нужно добавить 
администратором канала

Полное описание работы бота по [ссылке](http://nikovit.ru/blog/pishem-bota-peresylki-soobshcheniy-iz-vk-v-telegram-na-python/)

Очень часто бывает что у вас группа в vk.com и вам бы хотелось завести канал в телеграмм но постить вручную сообщения в два источника не очень удобно. Ниже мы рассмотрим бота для пересылки сообщений из вконтакте в телеграм.



Регистрируем бота в Telegram

Добавляем в список контактов @BotFather

Отправляем ему команду:
/newbot


Придумываем имя боту
Alright, a new bot. How are we going to call it? Please choose a name for your bot.

Придумываем username, должно заканчиваться обязательно на 'bot'
Good. Now let's choose a username for your bot. It must end in `bot`. Like this, for example: TetrisBot or tetris_bot.

Все, бот зарегистрирован, самое важное это последние сообщение с токеном бота, ни кому не сообщайте его т.к. зная токен можно полностью управлять ботом.
Done! Congratulations on your new bot. You will find it at t.me/XXXXbot. You can now add a description, about section and profile picture for your bot, see /help for a list of commands. By the way, when you've finished creating your cool bot, ping our Bot Support if you want a better username for it. Just make sure the bot is fully operational before you do this.


Use this token to access the HTTP API:
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX


For a description of the Bot API, see this page: https://core.telegram.org/bots/api

Пишем бота Telegram

Создаем в нашем проекте файл settings.ini и добавляем в него настройки подключения нашего будущего бота пересылки сообщений из vk.
[Settings]
last_id = 123
include_link = true
preview_link = false
[VK]
login = login
password = pass
domain = oldlentach
count = 30
[Telegram]
bot_token = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
channel = @test
где:
last_id - последний ID сообщения ленты вконтакте, можно оставить 123
include_link - вставлять ли ссылки
preview_link - использовать ли предпросмотр ссылок

login - ваш логин аккаунта вконтакте
password - ваш пароль аккаунта вконтакте
domain - группа или id сообщества вконтакте
count - количество последних забираемых сообщений сообщество (ограничение API вконтакте, максимум 100 за один запрос)

bot_token - токен бота полученный выше
channel - название канала в телеграмме для публикации ботом, важно помнить что для публикации сообщений в канале ботом, его нужно добавить администратором канала


Нам понадобятся библиотеки:

vk_api
pyTelegramBotAPI

configparser и logging из стандартной библиотеки Python, и конечно сам Python, на момент написания статьи у меня была версия 3.6.2

Устанавливаем библиотеки через pip желательно в virtualenv, в консоли набираем:
pip install vk_api
pip install pyTelegramBotAPI

Создаем файл vk_to_tg.py и начинаем в него писать.

Импортируем модули:
import os
import sys
import vk_api
import telebot
import configparser
import logging
from telebot.types import InputMediaPhoto

Считываем данные из settings.ini
config_path = os.path.join(sys.path[0], 'settings.ini')
config = configparser.ConfigParser()
config.read(config_path)
LOGIN = config.get('VK', 'LOGIN')
PASSWORD = config.get('VK', 'PASSWORD')
DOMAIN = config.get('VK', 'DOMAIN')
COUNT = config.get('VK', 'COUNT')
BOT_TOKEN = config.get('Telegram', 'BOT_TOKEN')
CHANNEL = config.get('Telegram', 'CHANNEL')
INCLUDE_LINK = config.getboolean('Settings', 'INCLUDE_LINK')
PREVIEW_LINK = config.getboolean('Settings', 'PREVIEW_LINK')

Инициализируем телеграмм бота vk телеграмм бота
bot = telebot.TeleBot(BOT_TOKEN)

Получаем данные из vk.com для последующей обработки
# Получаем данные из vk.com
def get_data(domain_vk, count_vk):
    vk_session = vk_api.VkApi(LOGIN, PASSWORD)
    vk_session.auth()
    vk = vk_session.get_api()
    # Используем метод wall.get из документации по API vk.com
    response = vk.wall.get(domain=domain_vk, count=count_vk)
    return response

Проверяем и извлекаем данные по условиям перед отправкой
# Проверяем данные по условиям перед отправкой
def check_posts_vk():
    response = get_data(DOMAIN, COUNT)
    response = reversed(response['items'])

    for post in response:

        # Читаем последний извесный id из файла
        id = config.get('Settings', 'LAST_ID')

        # Сравниваем id, пропускаем уже опубликованные
        if int(post['id']) <= int(id):
            continue

        print('-----------------------------------------')
        print(post)

        # Текст
        text = post['text']

        # Проверяем есть ли что то прикрепленное к посту
        images = []
        links = []
        attachments = []
        if 'attachments' in post:
            attach = post['attachments']
            for add in attach:
                if add['type'] == 'photo':
                    img = add['photo']
                    images.append(img)
                elif add['type'] == 'audio':
                    # Все аудиозаписи заблокированы везде, кроме оффицальных приложений
                    continue
                elif add['type'] == 'video':
                    video = add['video']
                    if 'player' in video:
                        links.append(video['player'])
                else:
                    for (key, value) in add.items():
                        if key != 'type' and 'url' in value:
                            attachments.append(value['url'])

        if INCLUDE_LINK:
            post_url = "https://vk.com/" + DOMAIN + "?w=wall" + \
                str(post['owner_id']) + '_' + str(post['id'])
            links.insert(0, post_url)
        text = '\n'.join([text] + links)
        send_posts_text(text)

        if len(images) > 0:
            image_urls = list(map(lambda img: max(
                img["sizes"], key=lambda size: size["type"])["url"], images))
            print(image_urls)
            bot.send_media_group(CHANNEL, map(
                lambda url: InputMediaPhoto(url), image_urls))

        # Проверяем есть ли репост другой записи
        if 'copy_history' in post:
            copy_history = post['copy_history']
            copy_history = copy_history[0]
            print('--copy_history--')
            print(copy_history)
            text = copy_history['text']
            send_posts_text(text)

            # Проверяем есть ли у репоста прикрепленное сообщение
            if 'attachments' in copy_history:
                copy_add = copy_history['attachments']
                copy_add = copy_add[0]

                # Если это ссылка
                if copy_add['type'] == 'link':
                    link = copy_add['link']
                    text = link['title']
                    send_posts_text(text)
                    img = link['photo']
                    send_posts_img(img)
                    url = link['url']
                    send_posts_text(url)

                # Если это картинки
                if copy_add['type'] == 'photo':
                    attach = copy_history['attachments']
                    for img in attach:
                        image = img['photo']
                        send_posts_img(image)

        # Записываем id в файл
        config.set('Settings', 'LAST_ID', str(post['id']))
        with open(config_path, "w") as config_file:
            config.write(config_file)

Отправляем посты в телеграмм

Если это текст:
# Текст
def send_posts_text(text):
    if text == '':
        print('no text')
    else:
        # В телеграмме есть ограничения на длину одного сообщения в 4091 символ, разбиваем длинные сообщения на части
        for msg in split(text):
            bot.send_message(CHANNEL, msg, disable_web_page_preview=not PREVIEW_LINK)

Если сообщение длинное то разбиваем его на несколько:
def split(text):
    if len(text) >= max_message_length:
        last_index = max(
            map(lambda separator: text.rfind(separator, 0, max_message_length), message_breakers))
        good_part = text[:last_index]
        bad_part = text[last_index + 1:]
        return [good_part] + split(bad_part)
    else:
        return [text]


Если это изображение:
# Изображения
def send_posts_img(img):
    # Находим картинку с максимальным качеством
    url = max(img["sizes"], key=lambda size: size["type"])["url"]
    bot.send_photo(CHANNEL, url)

И в самом конце инициализируем наш скрипт:
if __name__ == '__main__':
    check_posts_vk()

Репозиторий бота на github.com

Все, удачного Вам написания собственных Telegram ботов на Python
Просмотров:50750 Комментариев:47
Теги: Python, telegram, программирование



Добавить комментарий
Артур 07.10.2018 01:36:05
Два дня пытался но так и не смог отправить фото со стены вк в телеграм. Текст отправляет нормально но вот картинки никак. Пишет ошибку 400. Помогите пожалуйста понять что не так
Ответить  Ссылка
Виталий Николаев 09.10.2018 11:39:22
Да api действительно изменился, обновил статью, вот рабочая конструкция для отправки фото, обновил статью:
# Отправляем изображения
def send_posts_img(img):
    # Находим картинку с максимальным качеством
    if 'photo_2560' in img:
        print(img['photo_2560'])
        bot.send_photo(CHANNEL, img['photo_2560'])
        logging.info('Image: ' + img['photo_2560'])
    else:
        if 'photo_1280' in img:
            print(img['photo_1280'])
            bot.send_photo(CHANNEL, img['photo_1280'])
            logging.info('Image: ' + img['photo_1280'])
        else:
            if 'photo_807' in img:
                print(img['photo_807'])
                bot.send_photo(CHANNEL, img['photo_807'])
                logging.info('Image: ' + img['photo_807'])
            else:
                if 'photo_604' in img:
                    print(img['photo_604'])
                    bot.send_photo(CHANNEL, img['photo_604'])
                    logging.info('Image: ' + img['photo_604'])
