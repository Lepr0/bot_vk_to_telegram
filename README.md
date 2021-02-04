# vk_to_telegram_bot

Бот для пересылки сообщений из VK в Telegram. Форк от https://github.com/Nikovit/bot_vk_to_telegram
Здесь фото в постах группируется, т.к. все фото подписываются одним текстом.
Как это работает можно подсмотреть в @panda_art_cafe

Заполняем settings.ini своими значениями

где:

`last_id` - последний ID сообщения ленты вконтакте, обновится само  
`include_link` - вставлять ли в конце сообщения ссылку на пост
`preview_link` - включить ли предпросмотр ссылок
`login` - ваш логин аккаутна вконтакте  
`password` - ваш пароль аккаунта вконтакте  
`domain` - группа или id сообщества вконтакте  
`count` - количество последних забираемых сообщений сообщество (ограничение API вконтакте, максимум 100 за один запрос)  
`bot_token` - токен бота полученный выше  
`channel` - название канала в телеграмме для публикации ботом, важно помнить что для публикации сообщений в канале ботом, его нужно добавить 
администратором канала



Нам понадобятся библиотеки:

pip install vk_api
pip install pyTelegramBotAPI

configparser и logging из стандартной библиотеки Python, и конечно сам Python, на момент написания статьи была версия 3.6.2

