[English](/README.md) | [Русский](/README.ru_RU.md)

# Мониторинг Synology NAS с помощью SNMP и Grafana

<table>
  <tr>
    <td><img src="screen/dash-1.png" width="450" alt="dash-1.png"></td>
    <td><img src="screen/dash-2.png" width="450" alt="dash-2.png"></td>
  </tr>
  <tr>
    <td><img src="screen/dash-3.png" width="450" alt="dash-3.png"></td>
    <td><img src="screen/dash-4.png" width="450" alt="dash-4.png"></td>
  </tr>
</table>

## Описание

Настройка мониторинга Synology NAS для отображения метрик состояния NAS в дашборде Grafana.

Как это работает:
- На NAS запущена служба SNMP
- SNMP Exporter запрашивает SNMP метрики
- Prometheus получает и хранит метрики собранные SNMP Exporter
- Grafana отрисовывает дашборды на базе метрик хранящихся в Prometheus

```
┌╌╌╌╌╌┐       ┌╌╌╌╌╌╌╌╌╌╌╌╌╌┐       ┌╌╌╌╌╌╌╌╌╌╌╌╌┐       ┌╌╌╌╌╌╌╌╌╌┐
┆ NAS ┆ - - > ┆SNMP EXPORTER┆ - - > ┆ PROMETHEUS ┆ - - > ┆ GRAFANA ┆
└╌╌╌╌╌┘       └╌╌╌╌╌╌╌╌╌╌╌╌╌┘       └╌╌╌╌╌╌╌╌╌╌╌╌┘       └╌╌╌╌╌╌╌╌╌┘
```

## Установка

### Шаги установки
 - включить службу SNMP
 - скачать проект
 - отредактровать конфиг
 - загрузить проект на NAS
 - запустить проект в Portainer или Container Manager

### 1. Включаем службу SNMP
Для сбора метрик необходимо включить службу SNMP на вашем Synology NAS. 
Это можно сделать через `Control Panel` => `Terminal & SNMP` => `SNMP` => `Community: synology`

<details>
  <summary>SNMP</summary>
  <table>
  <tr>
    <td><img src="screen/snmp.png" width="450" alt="snmp.png"></td>
  </tr>
</table>
</details>

### 2. Клонируем репозиторий на свой ПК
   
### 3. Изменяем конфигурационнные файлы
- `.env`
  - BASE_PATH директория проекта на Synology File Station
  - GF_ADMIN_USER и GF_ADMIN_PASSWORD пользователь grafana и его пароль

- `prometheus/prometheus.yml` 
  - задать актуальный ip адрес NAS
    `- targets: [ "10.10.10.10" ] # IP address of Synology NAS`

### 4. Загрузить отредактированный проект на Synology NAS

Через File Station в директории `docker` создаём директорию `synology-snmp-monitoring`. \
В созданную директорию загружаем содержимое проекта. \
Можно путём drag and drop в браузере или использовать SFTP клиент.

<details>
  <summary>Folder</summary>
  <table>
  <tr>
    <td><img src="screen/folder.png" width="450" alt="folder.png"></td>
  </tr>
</table>
</details>

### 5. Создаём проект (стек)

Рассмотрим два варианта настройки стека: 
- с помощью Portainer (приоритетный вариант)
- с помошью стандартного Synology Container Manager

#### C помощью Portainer (приоритетный вариант)

> [!TIP]
> В сравнении с Container Manager исопльзование Portainer педоставляет более широкие возможности для управления контейнерами.
> Например в Portainer доступно управление docker volumes, что отсутствует в Container Manager.

##### Установка Portainer

Через File Station в директории `docker` создаём директорию `portainer` \
В Container Manager выбираем `Project` => `Create`
- Имя проекта `portainer`
- Путь `docker/portainer`
- Источник `Create docker-compose.yml` 

Вставляем следующую конфигурацию и запускаем проект

<details>
<summary>docker-compose.yml</summary>

```yaml
services:
    portainer:
        image: portainer/portainer-ce
        container_name: portainer
        restart: unless-stopped
        ports:
            - "8000:8000"
            - "9000:9000"
        volumes:
            - "/var/run/docker.sock:/var/run/docker.sock"
            - "/volume1/docker/portainer:/data"
```

</details>

##### Развёртывание стека

Логинимся в Portainer по адресу `http://<synology-ip>:9000`
- переходим в `Stacks`
- `Add stack` 
- `Name` указываем имя для нового стека `synology-snmp-monitoring` 
- `Build method` выбираем `Web editor`
- вставлям содержимое `docker-compose.yml` из проекта
- `Environment variables` нажимаем `Advanced mode`
- вставлям содержимое `.env` из проекта
- `Deploy the stack`

При успешном завершении развёртывания стека у нас будет три запущенных контейнера

<details>
  <summary>Stack</summary>
  <table>
  <tr>
    <td><img src="screen/stack.png" width="950" alt="stack.png"></td>
  </tr>
</table>
</details>

#### С помошью Synology Container Manager

В Container Manager выбираем `Project` => `Create`
- Имя проекта `synology-snmp-monitoring`
- Путь `docker/synology-snmp-monitoring`
- Выбираем `Use existing an docker-compose.yml to create the project` 

При успешном завершении развёртывания проекта у нас будет три запущенных контейнера

<details>
  <summary>Project</summary>
  <table>
  <tr>
    <td><img src="screen/project.png" width="950" alt="project.png"></td>
  </tr>
</table>
</details>

### 6. Адреса развёрнутых сервисов
- snmp-exporter `http://<synology-ip>:9116/snmp?target=<synology-ip>&auth=synology&module=synology`
- prometheus `http://<synology-ip>:9090`
- grafana `http://<synology-ip>:3000`

### 7. Дашборд
Логинимся в Grafana, дашборд Synology SNMP доступен в боковой панели во вкладке Dashboards

## Дополнительно

> [!IMPORTANT]
> Предварительно на Synology NAS должны быть настроены:
>  - служба DDNS
>  - работа с [wildcard сертификатами](https://mariushosting.com/synology-how-to-add-wildcard-certificate/)

Для того чтобы grafana была доступна по доменному имени `https://grafana.<your-synology-ddns>.synology.me` добавляем правило для reverse proxy

`Control Panel` => `Login Portal` => `Advanced` => `Reverse Proxy` => `Create`

<details>
  <summary>Reverse Proxy</summary>
  <table>
  <tr>
    <td><img src="screen/proxy-1.png" width="350" alt="proxy-1.png"></td>
    <td><img src="screen/proxy-2.png" width="350" alt="proxy-2.png"></td>
    <td><img src="screen/proxy-3.png" width="350" alt="proxy-3.png"></td>
  </tr>
  </table>
</details>