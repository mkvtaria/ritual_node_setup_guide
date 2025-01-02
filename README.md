# ritual_node_setup_guide

Для установки подойдет виртуальный сервер CPX21 (~7 евро/мес) на [Hetzner](https://console.hetzner.cloud/) или VPS 2 на [Contabo](https://contabo.com/en/vps/) (~10 евро/мес).

После получения данных для входа на сервер необходимо подключиться к нему через терминал и последовательно действовать по пронумерованным ниже шагам.

1. ```sudo apt update && sudo apt upgrade -y``` - здесь и далее команды, выделенные подобным образом, можно копировать и вставлять в терминал полностью без изменений и разбивок на подкоманды.
2. ```sudo apt -qy install curl git jq lz4 build-essential screen```
3. ```sudo apt install docker.io```
4. ```sudo curl -L "https://github.com/docker/compose/releases/download/v2.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose```
5. ```sudo chmod +x /usr/local/bin/docker-compose```
6. ```
   DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
   mkdir -p $DOCKER_CONFIG/cli-plugins
   curl -SL https://github.com/docker/compose/releases/download/v2.20.2/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
   ```
7. ```chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose```
8. ```docker compose version``` - если эта команда возвращает версию Docker (v2.20.2 или выше), то установка Docker успешно выполнена и можно приступать к дальнейшим шагам.
9. ```sudo usermod -aG docker $USER```
10. ```sudo reboot``` - этот шаг перезагрузит сервер, после потери соединения подождите минуту и подключитесь заново.
11. ```docker run hello-world``` - еще раз проверяем, что Docker работает корректно. Если команда возвращает результат, как на сриншоте ниже, то можно двигаться дальше. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot1.png)
12. ```git clone https://github.com/ritual-net/infernet-container-starter``` - скачиваем репозиторий Ritual и приступаем к установке ноды.
13. ```cd infernet-container-starter```
14. ```screen -S ritual``` - этой командой переходим в режим screen, в котором будет работать нода. Вход в сессию и выход из нее осуществляется специфическими командами, которые будут приведены позже.
15. ```project=hello-world make deploy-container``` - деплоим в этой сессии контейнер Ritual. Ошибки и варны, как на скриншоте ниже, можно игнорировать.![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot2.png)
16. Нажимаем Ctrl + A + D для выхода из созданной ранее сессии screen.
17. Теперь необходимо внести изменения в несколько конфигурационных файлов. Начнем с первого:
    ```nano ~/infernet-container-starter/deploy/config.json```
19. Здесь необходимо внести изменения для следующих переменных (значения, на которые их нужно изменить, приведены в скобках): 1) rpc_url (https://mainnet.base.org/), 2) private_key (приватный ключ вашего EVM кошелька, необходимо создать новый пустой кошелек для этих целей и пополнить его на 10-15$ в ETH в сети Base), 3) registry (0x3B1554f346DFe5c482Bb4BA31b880c1C18412170), 4) sleep (3), 5) starting_sub_id (160000), 6) batch_size (800), 7) sync_period (30), 8) trail_head_blocks (3). Ниже на первом скриншоте выделены переменные, в которые необходимо вносить изменения. На втором скриншоте они уже внесены (кроме приватного ключа). При редактировании файла обратите внимание на то, чтобы не сбить форматирование файла (например, значения некоторых переменных должны быть заключены в кавычки, после некоторых должна стоять запятая и т.д.). После внесения изменений нажмите Ctrl + X, затем Y, затем Enter.

Скриншот 1. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot3.png)
Скриншот 2. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot4.png)

21. ```nano ~/infernet-container-starter/projects/hello-world/container/config.json``` - затем в этом файле вносим ровно те же самые изменения, что и в прошлом.
22. ```nano ~/infernet-container-starter/projects/hello-world/contracts/script/Deploy.s.sol``` - в этом файле меняем переменную 'address registry' на адрес, используемый в предыдущих 2 файлах (0x3B1554f346DFe5c482Bb4BA31b880c1C18412170). ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot5.png)
23. ```nano ~/infernet-container-starter/projects/hello-world/contracts/Makefile``` - вносим изменения также в этом файле: sender заменяем на ваш приватный ключ, RPC_URL на https://mainnet.base.org/. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot6.png)
24. ```nano ~/infernet-container-starter/deploy/docker-compose.yaml``` - меняем версию ноды на 1.4.0. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot7.png)
25. ```screen -r ritual``` - подключаемся этой командой к созданной раннее сессии.
26. ```docker compose -f ~/infernet-container-starter/deploy/docker-compose.yaml down``` - останавливаем контейнеры.
27. ```docker compose -f ~/infernet-container-starter/deploy/docker-compose.yaml up``` - перезапускаем контейнеры. В результате должны пойти логи, как на скриншоте ниже. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot8.png) Выходим из сессии, не прерывая процесс, как и прежде, командой Ctrl + A + D.
28. ```mkdir foundry``` - для установки Foundry создаем соответствующую папку.
29. ```cd foundry``` - заходим в эту папку.
30. ```curl -L https://foundry.paradigm.xyz | bash``` - скачиваем и исполняем официальный скрипт.
31. ```source ~/.bashrc```
32. ```foundryup```
33. ```cd ~/infernet-container-starter/projects/hello-world/contracts``` - переходим в другую директорию.
34. ```forge install --no-commit foundry-rs/forge-std``` - устанавливаем нужную библиотеку. При возниковении ошибки "Error: git submodule exited with code 128" вводим последовательно команды ниже. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot9.png)
35. ```rm -rf projects/hello-world/contracts/lib/forge-std```
36. ```forge install --no-commit foundry-rs/forge-std```
37. ```cd ~/infernet-container-starter/projects/hello-world/contracts```
38. ```rm -rf lib/forge-std```
39. ```forge install --no-commit foundry-rs/forge-std```
40. ```ls lib/forge-std```
41. ```foundryup```
42. ```forge install --no-commit ritual-net/infernet-sdk``` - устанавливаем infernet-sdk. При возникновении ошибки "Error: git submodule exited with code 128" вводим последовательно команды ниже.
43. ```cd ~/infernet-container-starter/projects/hello-world/contracts```
44. ```rm -rf lib/infernet-sdk```
45. ```forge install --no-commit ritual-net/infernet-sdk```
46. ```ls lib/infernet-sdk```
47. ```screen -r ritual``` - подключаемся этой командой к созданной раннее сессии. Прерываем журнал логов нажатием Ctrl + C.
48. ```docker compose -f ~/infernet-container-starter/deploy/docker-compose.yaml down``` - останавливаем контейнеры.
49. ```docker compose -f ~/infernet-container-starter/deploy/docker-compose.yaml up``` - перезапускаем контейнеры. Выходим из сессии, не прерывая процесс, как и прежде, командой Ctrl + A + D.
50. ```cd ~/infernet-container-starter``` - переходим в нужную директорию.
51. ```project=hello-world make deploy-contracts``` - деплоим контракт SaysGM. Должен получиться результат, как на скриншоте ниже. Записываем полученный contract address, он понадобится на следующем шаге. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot10.png)
52. ```nano ~/infernet-container-starter/projects/hello-world/contracts/script/CallContract.s.sol``` - меняем в открывшемся файле значение контракта SaysGM в скобках на полученное нами на предыдущем шаге. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot11.png)
53. ```project=hello-world make call-contract``` - инициализируем запрос к нашему контракту. Должен получиться результат, как на скриншоте ниже. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot12.png)
54. В результате на кошельке, созданном для установки ноды, в эксплореры должны быть 2 последние транзакции, как на скриншоте ниже. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot13.png)
55. ```screen -r ritual``` - подключаемся к сессии screen еще раз, чтобы пронаблюдать за логами. Если идут блоки, то все нормально. Отключаемся с помощью Ctrl + A + D. ![Screenshot](https://github.com/mkvtaria/ritual_node_setup_guide/blob/main/screenshot14.png)
56. Также я бы рекомендовал вручную сделать 2 транзакции по адресу старого registry контракта. Для этого переходим по этой ссылке (https://basescan.org/address/0x8d871ef2826ac9001fb2e33fdd6379b6aabf449c#writeContract), подключаем кошелек, который использовали для установки ноды (кнопка "Connect to web3"), и в пункте 8. registerNode вводим адрес своего кошелька.
57. Cпустя более 24 часов после этой транзакции переходим к шагу 1. activateNode и подписываем теперь эту транзакцию.
