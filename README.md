# roll
Celestia-sovereign-rollup
#Celestia-суверенный-свертывание
---
##Введение
---
В этом руководстве мы создадим суверенный накопительный пакет gm-world, используя Rollkit и уровень доступности данных и консенсуса Celestia для отправки блоков Rollkit.

В этом руководстве мы рассмотрим настройку интерфейса командной строки Ignite, создание блокчейна свертки для конкретного приложения Cosmos-SDK и отправку данных в Celestia. Сначала мы проведем тестирование в локальной сети DA, а затем выполним развертывание в действующей тестовой сети.
##Первая часть
Эта часть руководства научит разработчиков, как легко запустить локальную сеть разработчиков доступности данных (DA) на их собственном компьютере (или в облаке). Запуск локальной сети разработки для DA для тестирования накопительного пакета — рекомендуемый первый шаг перед развертыванием в тестовой сети. Это устраняет необходимость в токенах тестовой сети и развертывании в тестовой сети, пока вы не будете готовы.

ПРИМЕЧАНИЕ Первая часть руководства была протестирована только на машине AMD с Ubuntu 22.10 x64.

Независимо от того, являетесь ли вы разработчиком, просто тестируете что-то на своем ноутбуке или используете виртуальную машину в облаке, этот процесс можно выполнить на любой машине по вашему выбору. Мы протестировали его на машине со следующими характеристиками:

Память: 1 ГБ ОЗУ ЦП: Одноядерный AMD Диск: 25 ГБ SSD Хранилище ОС: Ubuntu 22.10 x64

💻Предварительные условия Докер установлен на вашем компьютере 🏠Запуск локальной сети разработчиков с помощью накопительного пакета Rollkit Сначала запустите local-celestia-devnet, выполнив следующую команду:
```docker run --platform linux/amd64 -p 26650:26657 -p 26659:26659 ghcr.io/celestiaorg/local-celestia-devnet:main```
СОВЕТ Вышеупомянутая команда отличается от команды в учебнике «Запуск локальной сети Celestia Devnet» от Celestia Labs. Порт 26657 в контейнере Docker в этом примере будет сопоставлен с локальным портом 26650. Это сделано для того, чтобы избежать конфликтов портов с узлом Rollkit, поскольку мы запускаем devnet и узел на одном компьютере.

🔎Запросить баланс Откройте новый экземпляр терминала. Проверьте баланс своей учетной записи, которую вы будете использовать для публикации блоков в локальной сети, это позволит вам публиковать накопительные блоки в вашей Celestia Devnet для DA и консенсуса:
```curl -X GET http://0.0.0.0:26659/balance```
Найдите идентификатор запущенного контейнера с помощью команды:
```docker ps```
Затем остановите контейнер:
```docker stop CONTAINER_ID_or_NAME```
Вы можете получить идентификатор контейнера или имя остановленного контейнера с помощью команды docker ps -a, которая выведет список всех контейнеров (запущенных и остановленных) и их сведения. Например:
```docker ps -a```
вывод
```CONTAINER ID  IMAGE```                                            ```COMMAND            CREATED         STATUS```         ```PORTS```                                                                                                                         ```NAMES```
```d9af68de14e4   ghcr.io/celestiaorg/local-celestia-devnet:main   "/entrypoint.sh"   5 minutes ago   Up 2 minutes   1317/tcp, 9090/tcp, 0.0.0.0:26657->26657/tcp, :::26657->26657/tcp, 26656/tcp, 0.0.0.0:26659->26659/tcp, :::26659->26659/tcp   yrykhau```
Если вы когда-нибудь захотите удалить контейнер, вы можете использовать команду docker rm, за которой следует идентификатор или имя контейнера.

```docker rm CONTAINER_ID_or_NAME```

Создание суверенного накопительного пакета Теперь, когда у вас запущена сеть разработчиков Celestia, вы готовы установить Golang. Мы будем использовать Golang для создания и запуска нашего блокчейна Cosmos-SDK.

Интерфейс командной строки Ignite поставляется с командами создания шаблонов, которые ускоряют разработку блокчейнов, создавая все необходимое для запуска нового блокчейна Cosmos SDK.

Установите Golang (эти команды для amd64/linux):
```
cd $HOME
ver="1.19.1"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
Теперь используйте следующую команду для установки Ignite CLI:
```
curl https://get.ignite.com/cli! | bash
```
Откройте новую вкладку или окно в терминале и запустите эту команду, чтобы сформировать накопительный пакет. Закрепите цепь:
```
cd $HOME
ignite scaffold chain gm --address-prefix gm

TIP
The --address-prefix gm flag will change the address prefix from cosmos to gm. Read more on the Cosmos docs.

The response will look similar to below:

jcs @ ~ % ignite scaffold chain gm

⭐️ Successfully created a new blockchain 'gm'.
👉 Get started with the following commands:

 % cd gm
 % ignite chain serve

Documentation: https://docs.ignite.com

This command has created a Cosmos SDK blockchain in the gm directory. The gm directory contains a fully functional blockchain. The following standard Cosmos SDK modules have been imported:

staking - for delegated Proof-of-Stake (PoS) consensus mechanism
bank - for fungible token transfers between accounts
gov - for on-chain governance
mint - for minting new units of staking token
nft - for creating, transferring, and updating NFTs
and more
Change to the gm directory:

cd gm

You can learn more about the gm directory’s file structure here. Most of our work in this tutorial will happen in the x directory.

🗞️ Install Rollkit
To swap out Tendermint for Rollkit, run the following command from inside the gm directory:

go mod edit -replace github.com/cosmos/cosmos-sdk=github.com/rollkit/cosmos-sdk@v0.46.7-rollkit-v0.7.2-no-fraud-proofs
go mod edit -replace github.com/tendermint/tendermint=github.com/celestiaorg/tendermint@v0.34.22-0.20221202214355-3605c597500d
go mod tidy
go mod download
```
Запустите скрипт init-local.sh
```
bash init-local.sh
```
Ключи Перечислите ваши ключи:
```
gmd keys list --keyring-backend test
```
Вы должны увидеть вывод, подобный следующему

адрес: gm1sa3xvrkvwhktjppxzaayst7s7z4ar06rk37jq7 имя: gm-key-2 pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AlXXb6Op8DdwCejeYkGWbF4G3pDLDO+rYiVWKPKuvYaz"}' тип: локальный
адрес: gm13nf52x452c527nycahthqq4y9phcmvat9nejl2 имя: gm-key pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AwigPerY+eeC2WAabA6iW1AipAQora5Dwmo1SnMnjavt"}' тип: локальный
💸Транзакции Теперь мы можем протестировать отправку транзакции с одного из наших ключей на другой. Мы можем сделать это с помощью следующей команды:
```
gmd tx bank send [from_key_or_address] [to_address] [amount] [flags]
```
Установите свои ключи в качестве переменных, чтобы упростить добавление адреса:
```
export KEY1=gm1sa3xvrkvwhktjppxzaayst7s7z4ar06rk37jq7
export KEY2=gm13nf52x452c527nycahthqq4y9phcmvat9nejl2
```
Таким образом, используя нашу информацию из команды keys, мы можем построить команду транзакции, чтобы отправить 42069stake с одного адреса на другой:
```
gmd tx bank send $KEY1 $KEY2 42069stake --keyring-backend test
```
Вам будет предложено принять транзакцию:

auth_info: плата: сумма: [] gas_limit: "200000" грантодатель: "" плательщик: "" signer_infos: [] подсказка: null тело: extension_options: [] памятка: "" сообщения:

'@type': /cosmos.bank.v1beta1.MsgСумма отправки:
сумма: "42069" номинал: доля от_адреса: gm1sa3xvrkvwhktjppxzaayst7s7z4ar06rk37jq7 до_адреса: gm13nf52x452c527nycahthqq4y9phcmvat9nejl2 non_critical_extension_options: [] timeout_height: "0" подписи: [] подтвердить транзакцию перед подписанием и трансляцией]
Введите y, если вы хотите подтвердить и подписать транзакцию. Затем вы увидите подтверждение:

code: 0 codespace: "" data: "" events: [] gas_used: "0" gas_wanted: "0" height: "0" info: "" logs: [] raw_log: '[]' timestamp: "" tx: нулевой txhash: 677CAF6C80B85ACEF6F9EC7906FB3CB021322AAC78B015FA07D5112F2F824BFF

⚖️Остатки Затем запросите свой баланс
```gmd query bank balances $KEY2```
Это ключ, который получил баланс, поэтому он должен был увеличиться за пределы первоначального STAKING_AMOUNT:

остатки:

сумма: "10000000000000000000042069" номинал: доля разбивка на страницы: next_key: null итого: "0"
Другой ключ должен был уменьшиться в балансе:
```gmd query bank balances $KEY1```
Ответ:

остатки:

сумма: "99999999999999999999957931" номинал: доля разбиение на страницы: next_key: null итого: "0"

