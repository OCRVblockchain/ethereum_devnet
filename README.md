# ethereum_devnet

Ethereum devnet из двух нод, для разработки и тестирования технологии блокчейн.

## Перед началом убедитесь что установлен докер.

## ВАЖНО! каждая нода запускается в отдельном терминале с последующим входом в консоль GETH для работы с нодой. если терминал не требуется можно выйти из консоли CTRL+D, при этом контейнер останется работать.
переходим в каталог с репозиторием
```
cd $HOME/ethereum_devnet
```
Теперь запускаем поочередно обе ноды следующими командами:
```
docker-compose up -d node0
```
после успешного запуска контейнера вводим команду инициализации генезис файла:
```
docker exec -it node0 geth --datadir /root/data init /root/conf/multi-genesis.json

```
Затем выполним запуск geth консоли внутри контейнера с последующим заходом в эту самую консоль, откуда мы дальше будем оперировать нодой:
```
docker exec -it node0 geth --networkid "21" --identity "node0" --datadir /root/data --syncmode "full" --ipcdisable --nodiscover --allow-insecure-unlock --unlock 0x476A400cdB64D69E852cf773D74d77F9fC6cBFFc --password /root/data/node0.txt --http --http.addr "0.0.0.0" --http.port "8081" --http.corsdomain "*" --authrpc.port 8552 --port "30001" --verbosity "2" console  

```
Далее находясь внутри консоли geth мы добавим функцию:
```
function showBalances() {
  var i = 0;
  eth.accounts.forEach(function(e) {
     console.log("---> eth.accounts["+i+"]: " + e + " \tbalance: " + web3.fromWei(eth.getBalance(e), "ether") + " ether");
     i++;
  })
};

```
Теперь мы можем проверить баланс кошелька на этой ноде:
```
showBalances()

```
Увидим баланс кошелька 
```
---> eth.accounts[0]: 0x476a400cdb64d69e852cf773d74d77f9fc6cbffc        balance: 200 ether
```
Далее аналогично для второй ноды из другого терминала:
 ```
docker-compose up -d node1

docker exec -it node1 geth --datadir /root/data init /root/conf/multi-genesis.json

docker exec -it node1 geth --networkid "21" --identity "node1" --datadir /root/data --syncmode "full" --ipcdisable --nodiscover --allow-insecure-unlock --unlock 0xd9f554731078c1b97310d64d50b006e200c68149 --password /root/data/node1.txt --http --http.addr "0.0.0.0" --http.port "8081" --http.corsdomain "*" --authrpc.port 8553 --port "30001" --verbosity "2" console 

```
После чего так же можем проверить баланс ноды. Теперь для полноценного блокчейна необходимо соединить эти две ноды.

# Знакомим ноды друг с другом

Для этого в консоли node0 вводим команду которая отобразит данные ноды, конкретно нам нужен адрес enode:
```
admin.nodeInfo
```
В полученной информации извлекаем адрес:
```
enode://60d386c552d0ab16c6bb4b7d0ef0f7d305642cf9dc76951dd03a2a765d610ce152ad8c1f63857124af92d1265ca76d5656596885d3e3e5041f36380e8b67a997@127.0.0.1:30001
```
Этот адрес нам надо прокинуть другой ноде, а соответственно из второй ноды, взять адрес и прокинуть в первую.
Возвращаемся в консоль второй ноды и добавляем первую (хочу обратить ваше внимание что адрес нужно ставить локальной машины на которой развернута нода в данном случае адрес моей машины у обеих нод одинаковый будет 172.22.101.4):
```
admin.addPeer('enode://60d386c552d0ab16c6bb4b7d0ef0f7d305642cf9dc76951dd03a2a765d610ce152ad8c1f63857124af92d1265ca76d5656596885d3e3e5041f36380e8b67a997@172.22.101.4:30001’)
```
Далее аналогично добавляем в первую ноду адрес второй ноды:
```
admin.addPeer('enode://5559b2bca4627837e7dead51a4f6c474d116e625668fb993adbc3b987f3f54572a5551c710ecf920b16f826e00145d0af1f1e494a4fef9ab2be4b166b88322df@172.22.101.4:30002')
```
Известив обе ноды о существовании друг друга, мы убедимся в этом, в каждой консоли введем:
```
admin.peers
```
В выводе мы увидим информацию о соседней ноде:
```
[{
    caps: ["eth/66", "eth/67", "snap/1"],
    enode: "enode://5559b2bca4627837e7dead51a4f6c474d116e625668fb993adbc3b987f3f54572a5551c710ecf920b16f826e00145d0af1f1e494a4fef9ab2be4b166b88322df@172.22.101.4:30002",
    id: "831a4b88b970b8052f2523485096887f9849a8034f7b406b8a054155d0f0bd6b",
    name: "Geth/node1/v1.10.25-stable-69568c55/linux-amd64/go1.18.6",
    network: {
      inbound: false,
      localAddress: "192.168.80.2:44878",
      remoteAddress: "172.22.101.4:30002",
      static: true,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 1,
        head: "0xec0c65d519c517f774793ea47e296b68d8f34b4ca0b0448f7c3b1e5de0ee9146",
        version: 67
      },
      snap: {
        version: 1
      }
    }
}]
```
В итоге получили блокчейн сеть эфириум с консенсусом PoW. Можно отправить транзакцию тестовую (предварительно на node1 требуется запустить майнера, иначе тразакция будет висеть в пендинге. По желанию можете не запускать и проверить)
Для запуска майнера выполним команду
```
miner.start(1)
```
После проверим что транзакция исчезла из пендинга.
Для остановки майнера
```
miner.stop()
```
Отправляем транзакцию в объеме 10 ether
```
eth.sendTransaction({from: "0x476A400cdB64D69E852cf773D74d77F9fC6cBFFc",to: "0xd9f554731078c1b97310d64d50b006e200c68149", value: web3.toWei(10,"ether")})
```
Либо такой командой
```
eth.sendTransaction({from: "0x476A400cdB64D69E852cf773D74d77F9fC6cBFFc",to: "0xd9f554731078c1b97310d64d50b006e200c68149", value: 10000000000000000000})
```
Кто знает пишите в комментариях в чем разница работы команд.
Указываем с какого на какой кошелек перевести какую сумму.
А следующей командой можно посмотреть транзакцию если она еще не прошла, если успеете, либо если не запущен майнер она будет висеть в ожидании.
```
eth.pendingTransactions
```
Увидим следующую информацию
```
[{
    blockHash: null,
    blockNumber: null,
    chainId: "0x15",
    from: "0x476a400cdb64d69e852cf773d74d77f9fc6cbffc",
    gas: 21000,
    gasPrice: 1000000000,
    hash: "0x6a92a0beee2efe5b1633ae0134dc3e27cf49be06a468c43264e5d5eb12309dc6",
    input: "0x",
    nonce: 6,
    r: "0x1c7ff44d3aa79175dcacc4f1a163e4061e2dc25e1062cc61f257d348a08e58e3",
    s: "0x2b39bb0bc732d8196ade8039da587ac7097a824e8633d01b4b1618b7f9c325c1",
    to: "0xd9f554731078c1b97310d64d50b006e200c68149",
    transactionIndex: null,
    type: "0x0",
    v: "0x4e",
    value: 10000000000000000000
}]
```
Теперь снова посмотрев баланс можно обнаружить что на одном кошельке сумма уменьшилась на 10 эфира, а на другом соответственно возросла.


## Для разработчиков можно развернуть remix-ide - комплексный инструмент для работы со смарт-контрактами.

разворачивается следующей командой
```
docker-compose -f remix-ide.yaml up -d
```
инструмент будет доступен по адресу машины на которой развернут, на порту 8090, вместо локалхост можно указать айпи адрес если развернут на другой машине
- [ ] [http://localhost:8090](http://localhost:8090)

## Для того чтобы остановить все ноды 
Для полной остановки проекта выходим из обеих нод (из консоли) и выполняем команду
```
docker-compose down
```



