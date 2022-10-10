# ebotSpot
Мануал. E-Bot spot.
Бот для торговли на Binance spot с использованием стратегии мартингейла (усреднения) и выбором растущей монеты.
E-Bot выбирает монету, выросшую за указанный в настройках период времени на указанный процент, 
- покупает ее маркет-ордером на указанный объем, 
- выставляет купленные монеты на продажу лимитным sell-ордером по курсу на указанный процент прибыли выше курса покупки,
- выставляет лимитный buy-ордер на покупку этой же монеты по курсу ниже предыдущей покупки на указанный процент (на случай падения курса монеты и уменьшения средней цены входа в сделку).
Затем, в зависимости от того, какой ордер исполнился:
- если исполнился sell-ордер, бот фиксирует прибыль и отменяет buy-ордер для усреднения(если buy-ордер при этом успел исполниться частично или полностью- выставляется sell-ордер), затем снова ищет подходящую пару,
- если исполнился buy-ордер, бот отменяет sell-ордер, и выставляет новый sell-ордер уже с новым количеством монет и по новой цене(средняя цена входа + указанный процент прибыли),
- если buy-ордер исполнился больше, чем наполовину и прошло более 5-ти минут после этого , бот отменяет sell-ордер, и выставляет новый sell-ордер уже с новым количеством монет и по новой цене(средняя цена входа + указанный процент прибыли).

Бот устанавливается на VPS сервер ubuntu 20 и запускается в SCREEN (чтобы бот не отключался при разрыве SSH-соединения с VPS), настраивается telegram-бот и канал, куда приходит информация о работе бота. Запуск бота командой: ./ebotSpot-6, остановка бота командой: ctrl+c.

Для работы E-Bota необходимо использовать BNB для оплаты комиссий биржи (нужно поставить соответствующую галочку в лк binance) и следить за наличием BNB на спот аккаунте.

После закрытия каждой сделки E-Bot:
- отправляет сообщение в telegram-канал, 
- каждую минуту в описание канала отправляет информацию об открытой позиции, 
- в полночь в канал отправляет суточный отчёт о работе.

Настройки бота:
- sell_up: процент повышения цены для продажи,
- buy_down: процент падения цены для докупки,
- step_aver: шаг увеличения buy_down,
- qty_aver: кратное увеличение объема покупки,
- k_step: коэффициент умножения предыдущего buy_down,
- формула расчета: 
1-й усреднение при падении цены от последней покупки на
buy_down*k_step + step_aver,
2-е усреднение (и последующие) при падении цены от последней покупки на 
(1-й усред)*k_step+step_aver,
- min_order: минимальная покупка в квотируемой валюте (например, в паре ETH/USDT это USDT, ставить больше 11),
- delta_start: на сколько % должен подняться курс от цены открытия выбранной свечи до текущей цены для старта,
- stop_loss- на сколько % должен упасть курс монеты для закрытия в минус,
- used_stop_loss- включить использование stop_loss для закрытия в минус (да/нет),
- completed- поставить бота на паузу при закрытии очередной сделки (1-вкл/0-выкл),
- kline_interval: интервал свечей для анализа (1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 8h, 12h, 1d, 3d),
- interval_min_ago: на сколько минут назад анализируем свечи (если kline_interval 3m, то, чтобы анализировать 2 послед свечи, interval_min_ago ставим 6),
- super_asset: пара для бесконечной торговли независимо от delta_start, при этом пары из white_list не будут работать (вводится командой -super_asset_add в формате ETHUSDT),
-fix_loss: команда для закрытия открытой позиции по рынку, 
- clear: сброс из базы данных сведений об открытых ботом ордерах,
- clear_profit: сброс из базы данных сведений о прибыли,
- white_list: список пар, которые бот будет использовать для анализа и выбора подходящей для открытия сделки (вводится по одной паре командой -w_list_add),
- api_key: открытый api-ключ от биржи с разрешением на спотовую торговлю,
- api_secret: секретный api-ключ от биржи,
- botID- api телеграм бота полученный от @BotFather (пример: 5656544920:AAHrXhjhujhfdf7RPJlheqJXEulBW),
- channelID- ID канала telegram бота для уведомлений, полученное от @userinfobot (пример: -1001656543985),
- tguserid- ID основного user-a телеграм, полученное от @userinfobot (пример: 346549043),

Здесь E-Bot представлен для ознакомления и использования в течении пробного периода до Mon Oct 24 2022 20:59:59 GMT+0000
Если Вы хотите увеличить время работы до 1/6/12 месяцев: напишите в телеграм, по данным, указанным при запуске E-Bot.

Если хотите испытать E-Bot на спотовой тестовой бирже Binance- 
переходите на:
https://testnet.binance.vision/
оттуда на github, где получите тестовые api-ключи, прописываете их в настройках бота, в used_testnet записываете: да, и экспериментируйте.

E-Bot поставляется по принципу «как есть». Никаких гарантий не прилагается и  не  предусматривается. Вы берете на себя весь риск относительно использования этого бота.
Вы должны понимать, что торговля на криптобиржах сопряжена с повышенным риском, и подходить к управлению рисками со всей ответственностью. 

Пояснения по установке, запуску, настройке бота и телеграм, screen, ошибке на  VPS utf-8.

Иногда бот может получить от биржи неправильные ответы на api-запросы и выдавать ошибку, поэтому рекомендуется периодически заглядывать в лк binance, и, если бот показывает открытые ордера а в лк binance их нет (или наоборот), нужно использовать команду -clear, чтобы сбросить в боте данные о неактуальных ордерах.
Редко, но бывает, что сервера telegram кратковременно недоступны, и в этот момент сообщение от E-Bot может не доходить в канал бота. 
Для управления ботом на VPS сервере с телефона можно использовать приложение JuiceSSH (или другое для SSH-соединения).

Установка и запуск E-Bot:
- на VPS-сервере ubuntu 20 создайте новую папку, например, ebotSpot (mkdir ebotSpot)
- зайдите в эту папку (cd ebotSpot)
- перенесите в эту папку файл бота ebotSpot-6
- откройте screen-сессию (например: screen -S ebotSpot)
- дайте права запуска файла (команда: chmod 755 ebotSpot-6)
- запустите E-Bot (команда: ./ebotSpot-6)
- команда для остановки бота: ctrl+c
- после запуска бота введите свои параметры: api_key и т.д.
- откорректируйте, при необходимости, настройки
- жмите ENTER и наблюдайте
- для выхода из SCREEN перед закрытием SSH-сессии используйте команду ctrl+a, d 
- для входа в screen работающего бота используйте команду: screen -x ebotSpot


Скриншоты


