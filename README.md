# ebotSpot
E-Bot spot.
Бот для торговли на Binance spot с использованием стратегии мартингейла (усреднения) и выбором растущей монеты.

E-Bot выбирает монету, выросшую за указанный в настройках период времени на указанный процент, 
- покупает ее маркет-ордером на указанный объем, 
- выставляет купленные монеты на продажу лимитным sell-ордером по курсу на указанный процент прибыли выше курса покупки,
- выставляет лимитный buy-ордер на покупку этой же монеты по курсу ниже предыдущей покупки на указанный процент (на случай падения курса монеты и уменьшения средней цены входа в сделку).

Затем, в зависимости от того, какой ордер исполнился:
- если исполнился sell-ордер, бот фиксирует прибыль и отменяет buy-ордер для усреднения(если buy-ордер при этом успел исполниться частично или полностью- выставляется sell-ордер), затем снова ищет подходящую пару,
- если исполнился buy-ордер, бот отменяет sell-ордер, выставляет новый sell-ордер уже с новым количеством монет и по новой цене(средняя цена входа + указанный процент прибыли), выставляет новый buy-ордер,
- если buy-ордер исполнился больше, чем наполовину и прошло более 5-ти минут после этого , бот отменяет sell-ордер, и выставляет новый sell-ордер уже с новым количеством монет и по новой цене(средняя цена входа + указанный процент прибыли).

Рекомендуется E-Bot установить на VPS сервер ubuntu 20 и запускать в SCREEN (чтобы бот не отключался при разрыве SSH-соединения с VPS), настроить telegram-бот и канал, куда будет приходит информация о работе бота. 
- Запуск бота командой:         ./ebotSpot-7.8g 
- Остановка бота командой:      ctrl+c    (важно!: не останавливайте бот в момент совершения сделок, возможна ошибка записи в базу данных бота).
- В white_list (список пар для работы) можно внести от 1 до нескольких сотен пар, главное, чтобы квотируемая валюта была одна: если торгуете к USDT, то пары ETHUSDT, BTCUSDT, XRPUSDT и т.д., если торгуете к BTC, то пары ETHBTC, XRPBTC, DASHBTC и т.д., если к BUSD, то пары с BUSD и т.д.
- При изменении квотируемой валюты (с USDT на BTC и т.д.) не забывайте менять и min_order в настройках бота (в USDT min_order должен быть больше 11, а в BTC больше 0.00011)
- При работе с парами к USDT активы должны находится на спотовом балансе USDT, если работаете с парами к BTC, активы должны находится на спотовом балансе  BTC.
- Для прокрутки экрана терминала вверх есть команда: ctrl+a, esc и далее стрелка вверх. Для выхода из этого режима: esc, esc.

Для работы E-Bota необходимо использовать BNB для оплаты комиссий биржи (нужно поставить соответствующую галочку в лк binance) и следить за наличием BNB на спот аккаунте.

После закрытия каждой сделки E-Bot:
- отправляет сообщение в telegram-канал, 
- каждую минуту в описание канала отправляет информацию об открытой позиции, 
- в полночь в канал отправляет суточный отчёт о работе. Если не было прибыли за сутки, а также если бот находится в режиме поиска подходящей монеты, то суточный отчёт в telegram не придёт, а в следующий отчёт будет прибавлена предыдущая прибыль за сутки. Точные данные по прибыли наблюдать лучше в лк binance, так как бот показывает приблизительные значения.

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
- min_bal_perc: минимальный процент от депозита, ниже которого E-Bot не будет выставлять усредняющий ордер.
- delta_start: на сколько % должен подняться курс от цены открытия выбранной свечи до текущей цены для старта,
- stop_loss- на сколько % должен упасть курс монеты от средней цены входа для закрытия в минус,
- used_stop_loss- включить использование stop_loss для закрытия в минус (да/нет),
- pause_after_stop_loss - ставить E-Bot на паузу после срабатывания stop_loss и закрытия позиции по рынку или продолжить работу,
- completed- поставить бота на паузу при закрытии очередной сделки (1-вкл/0-выкл),
- kline_interval: интервал свечей для анализа (1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 8h, 12h, 1d, 3d),
- interval_min_ago: на сколько минут назад анализируем свечи (если kline_interval 3m, то, чтобы анализировать 2 послед свечи, interval_min_ago ставим 6),
- super_asset: пара для бесконечной торговли независимо от delta_start, при этом пары из white_list не будут работать (вводится командой -super_asset_add в формате ETHUSDT),
- manual_aver: команда для ручного усреднения по рынку, не дожидаясь цены лимитного усред-ордера
- fix_loss: команда для закрытия открытой позиции по рынку, 
- clear: сброс из базы данных сведений об открытых ботом ордерах,
- clear_profit: сброс из базы данных сведений о прибыли,
- white_list: список пар, которые бот будет использовать для анализа и выбора подходящей для открытия сделки (вводится по одной паре командой -w_list_add),
- api_key: открытый api-ключ от биржи с разрешением на спотовую торговлю,
- api_secret: секретный api-ключ от биржи,
- botID- api телеграм бота полученный от @BotFather (пример: 5656544920:AAHrXhjhujhfdf7RPJlheqJXEulBW),
- channelID- ID канала telegram бота для уведомлений, полученное от @userinfobot (пример: -1001656543985),
- tguserid- ID основного user-a телеграм, полученное от @userinfobot (пример: 346549043),

Здесь E-Bot представлен для ознакомления и использования в течении пробного периода по 31 декабря 2022 г.
Если Вы хотите увеличить время работы до 1/6/12 месяцев: напишите в телеграм, по данным, указанным при запуске E-Bot.

Если хотите испытать E-Bot на спотовой тестовой бирже Binance- 
переходите на:
https://testnet.binance.vision/
оттуда на github, где получите тестовые api-ключи, пропишите их в настройках бота, в used_testnet запишите: да, и экспериментируйте.

E-Bot поставляется по принципу «как есть». Никаких гарантий не прилагается и  не  предусматривается. Вы берете на себя весь риск относительно использования этого бота.
Вы должны понимать, что торговля на криптобиржах сопряжена с повышенным риском, и подходить к управлению рисками со всей ответственностью. 

Пояснения по установке, запуску, настройке бота и телеграм, screen, ошибке на  VPS utf-8.

Иногда бот может получить от биржи неправильные ответы на api-запросы и выдавать ошибку, поэтому рекомендуется периодически заглядывать в лк binance, и, если бот показывает открытые ордера а в лк binance их нет (или наоборот), нужно использовать команду -clear, чтобы сбросить в боте данные о неактуальных ордерах.
Редко, но бывает, что сервера telegram кратковременно недоступны, и в этот момент сообщение от E-Bot может не доходить в канал бота. 
Для управления ботом на VPS сервере с телефона можно использовать приложение JuiceSSH (или другое для SSH-соединения).

Установка и запуск E-Bot:
- на VPS-сервере ubuntu 20 создайте новую папку, например, ebotSpot (   mkdir ebotSpot   )
- зайдите в эту папку (   cd ebotSpot   )
- перенесите в эту папку файл бота ebotSpot-7.8g (или скачайте с github командой: wget https://github.com/ebot732/ebotSpot/releases/download/ebotSpot-7.8g/ebotSpot-7.8g)
- откройте screen-сессию (например:    screen -S ebotSpot   )
- дайте права запуска файлу (команда:    chmod 755 ebotSpot-7.8g   )
- запустите E-Bot (команда:    ./ebotSpot-7.8g   )
- команда для остановки бота:    ctrl+c
- после запуска бота введите свои параметры: api_key и т.д.
- откорректируйте, при необходимости, настройки
- жмите ENTER и наблюдайте
- для выхода из SCREEN перед закрытием SSH-сессии используйте команду:    ctrl+a, d 
- для входа в screen работающего бота используйте команду:                screen -x ebotSpot

Для удобства настройки E-Bot можно использовать прогу tgUpravSpot5. Для этого нужно добавить ее в папку, где работает Е-Bot и запустить (./tgupravSpot5) в отдельном SCREEN. Необходимые данные прога возьмет из БД E-Bot и будет управляться через чат telegram-Bot.

Скриншоты 


![Screenshot](https://github.com/ebot732/ebotSpot/blob/main/Screenshot_20221010-190153_Telegram.jpg)

=================================================================================================


![Screenshot](https://github.com/ebot732/ebotSpot/blob/main/Screenshot_20221010-190218_Telegram.jpg)

=================================================================================================


![Screenshot](https://github.com/ebot732/ebotSpot/blob/main/Screenshot_20221010-190224_Telegram.jpg)

=================================================================================================


![Screenshot](https://github.com/ebot732/ebotSpot/blob/main/Screenshot_20221010-190319_JuiceSSH.jpg)

=================================================================================================


![Screenshot](https://github.com/ebot732/ebotSpot/blob/main/Screenshot_20221107-214146_Telegram.jpg)

=================================================================================================


![Screenshot](https://github.com/ebot732/ebotSpot/blob/main/Screenshot_20221107-214153_Telegram.jpg)




Табличка Spot_усреды_Ebot.xls показывает приблизительные расчёты усреднений.
