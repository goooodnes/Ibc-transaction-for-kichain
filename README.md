# Ibc-transaction-for-kichain

            В этой статье я расскажу как наладить ретранслятор для кросс перевода между двумя сетями на примере Rizon и Kichain
 Для этого нам понадобится ретранслятор и две работающие ноды. Или ретранслятор и одна работающая нода и глобальный rpc от второй сети. Или ретранслятор и глобальный rpc от двух сетей. Но все по порядку .
 Я буду устанавливать ретранслятор к ноде Rizon, вы же можете выбрать любой другой удобный вам способ. Главный критерий - у выбранной вами сети должен быть включен IBC-transfer. 
  Данный параметр можно проверить введя в терминале где установлена нода. следующую команду:
 
 $ rizond q ibc-transfer params  # ( вместо rizond укажите команду вашей ноды)
 
 #ответ должен содержать следующий вывод:

receive_enabled: true
send_enabled: true
          
          Итак мы выяснили что Сеть Rizon отлично подходит для кросс транзакций. Приступим к установке и наладке установке и отладке Ретранслятора.
 Скачиваем и устанавливаем ретранслятор из официального источника v.1.0.0

$ git clone https://github.com/cosmos/relayer.git

$ cd relayer

$ make install

$ cd

$ rly version

вывод:
version: 1.0.0-rc1–152-g112205b
 
                                               Инициализируем ретранслятор
$ rly config init
                                     создаем папку с конфигурацией сети и переходим в нее:

$ mkdir rly_config
$ cd rly_config

                               создаем файл настроек для обоих сетей:
$ nano kichain-t-3.json
                                                           копируем содержимое и помещаем его в файл
             #используем глобальный rpc так как ставим на vps где локально не установлена нода kichain.

{
 "chain-id": "kichain-t-3",
 "rpc-addr": "https://rpc-challenge.blockchain.ki:443", 
 "account-prefix": "tki",
 "gas-adjustment": 1.5,
 "gas-prices": "0.025utki",
 "trusting-period": "48h"
}
 
              аналогично создаем файл для второй сети, вместо глобального rpc вводим локальный так как нода rizon располагается на одном сервере. В теории можно наладить ретрансляцию не имея ни одной ноды но имея глобальный rpc узел обоих сетей.

$ nano groot-011.json

{
 "chain-id": "groot-011",
 "rpc-addr": "http://localhost:26657", 
 "account-prefix": "rizon",
 "gas-adjustment": 1.5,
 "gas-prices": "0.025uatolo",
 "trusting-period": "48h"
}

 Парсим эти настройки в конфиг релеера:

$ rly chains add -f groot-011.json
$ rly chains add -f kichain-t-3.json
$ cd

                     создаем новые кошельки или восстанавливаем уже имеющиеся у вас:

$rly keys add groot-011 имя вашего кошелька 
$rly keys add kichain-t-3 имя вашего кошелька
 
                                              или же восстанавливаем командой:

$ rly keys restore groot-011 имя вашего кошелька "мнемоническая фарза от кошелька"
$ rly keys restore kichain-t-3 имя вашего кошелька "мнемоническая фарза от кошелька"

 Вы можете использовать одну мнемонику для обоих кошельков что по мне очень даже удобно :-) ретранслятор создаст разные кошельки 
исходя из настроек сети с разными приставками rizon и tki:

                 Добавляем вновь созданные ключи в конфиг релеера:
$ rly chains edit groot-011 key Имя кошелька
$ rly chains edit kichain-t-3 key Имя кошелька

                    Изменим тайм аут ожидания подтверждения на 30s изменив стандартные 10s:
$ nano ~/.relayer/config/config.yaml
timeout: 30s
 
         Кошельки должны быть пополнены в обоих сетях. проверяем наличие монет командой:
$ rly q balance groot-011
$ rly q balance kichain-t-3

             Если средства на балансе есть и ретранслятор их показывает то продолжаем Инициализируем легкий клиент в обоих сетях командой:

$ rly light init groot-011 -f
$ rly light init kichain-t-3 -f

          Пробуем создать канал между сетями командой:

$ rly paths generate groot-011 kichain-t-3 transfer - port=transfer

          # Если команда не срабатывает пробуйте несколько раз или добавьте параметр - debug что бы посмотреть пошаговые действия системы
вывод должен быть следующим:

Generated path(transfer), run 'rly paths show transfer - yaml' to see details
             Можно открыть и посмотреть с генерированные настройки указанной выше командой.

         Пробуем открыть канал для ретрансляции:

$ rly tx link transfer  -- debug

                если так случилось что канал не открывается то идем в конфиг  командой:

$ nano /root/.relayer/config/config.yaml

                          стираем строки в разделе path в обоих сетях:
client-id: 07-tendermint-16
 connection-id: connection-14
 channel-id: channel-11

            затем повторно выполняем инициализацию легкого клиента командами:

$ rly light init groot-011 -f
$ rly light init kichain-t-3 -f

           и снова запускаем команду открытия канала
 
$ rly tx link transfer  -- debug

и так до тех пор пока не увидите вывод команды:
I[2021–09–06|09:22:54.913] ★ Channel created: [groot-011]chan{channel-11}port{transfer} -> [kichain-t-3]chan{channel-41}port{transfer}

          теперь при вызове команды

$ rly paths list -d

 0: transfer -> chns(✔) clnts(✔) conn(✔) chan(✔) (groot-011:transfer<>kichain-t-3:transfer)
вы увидите готовность к переводу и ретрансляции пакетов.
Ну что попробуем выполнить меж сетевую транзакцию?

$ rly transact transfer [src-chain-id] [dst-chain-id] [amount] [dst-addr] [flags] 
#шаблон команды но не так черт страшен как его рисуют.. в нашем случае команда будет выглядит так:

$ rly tx transfer groot-011 kichain-t-3 1000000uatolo tki1eakzw0qhxcclerm0uwxae88ugcmet2radufqqf - path transfer
                  и ответом нам будет вывод об удачной транзакции hash которой можно проверить в обоих сетях rizon и kichain в explorer

I[2021–09–06|09:35:00.471] ✔ [groot-011]@{412926} - msg(0:transfer) hash(D4E8A8C5CA7E6ED0B3FD247C456938E1A160B3E850FB10E27440D25A08C69DAE)

https://testnet.mintscan.io/rizon/txs/D4E8A8C5CA7E6ED0B3FD247C456938E1A160B3E850FB10E27440D25A08C69DAE
теперь выполним перевод в другую сторону команда выглядит анологично

$ rly tx transfer kichain-t-3 groot-011 1000000utki rizon1eakzw0qhxcclerm0uwxae88ugcmet2raquwh9n - path transfer

                            если вы забыли адреса кошельков. такое бывает порою :-) то из можно посмотреть командой

$ rly keys list groot-011
$ rly keys list kichaint-t-3

на этом пожалуй все.
