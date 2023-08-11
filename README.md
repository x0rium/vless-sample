# vless-sample

Ультра простой варинт настройки VLESS, XTLS-Reality + XTLS-Vision. 

Опробирован на Debian 11 на [VDS от iHor за 240 рублей (1 СPU/768 MB RAM/7 GB SSD) с Европейским IP](https://www.ihor-hosting.ru/?from=180121). Полет отличный.

Преимущества, буду краток:
- беспалевный
- легковесный
- нет двойного/тройного шифрования
- поддерживается большим колчисетвом софта


## Настройка

```
wget https://github.com/XTLS/Xray-core/releases/download/v1.8.1/Xray-linux-64.zip

sudo -

mkdir /opt/xray

unzip ./Xray-linux-64.zip -d /opt/xray

chmod +x /opt/xray/xray

nano /usr/lib/systemd/system/xray.service
```


```

[Unit]
Description=Xray Service
Documentation=https://github.com/xtls
After=network.target nss-lookup.target

[Service]
User=nobody
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
ExecStart=/opt/xray/xray run -config /opt/xray/config.json
Restart=on-failure
RestartPreventExitStatus=23
LimitNPROC=10000
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target

```

```
systemctl enable xray
```
Сохраняем все из следующих трех команд куда нибудь:

````
/opt/xray/xray uuid 

/opt/xray/xray x25519

openssl rand -hex 8


nano /opt/xray/config.json
````
Пихаем в него, И не забудем поправить : 
- id
- privateKey
- shortIds


```JSON
{
  "log": {
    "loglevel": "info"
  },
  "routing": {
    "rules": [],
    "domainStrategy": "AsIs"
  },
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "tag": "vless_tls",
      "settings": {
        "clients": [
          {
            "id": "сюда пихаем значения отсюда /opt/xray/xray uuid",
            "email": "user1@myserver",
            "flow": "xtls-rprx-vision"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
                "realitySettings": {
                        "show": false,
                        "dest": "www.microsoft.com:443",
                        "xver": 0,
                        "serverNames": [
                                "www.microsoft.com"
                        ],
                        "privateKey": "сюда пихаем значения их секции Private /opt/xray/xray x25519 ",
                        "minClientVer": "",
                        "maxClientVer": "",
                        "maxTimeDiff": 0,
                        "shortIds": [
                                "А сюда из  openssl rand -hex 8"
                        ]
                }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      }
    }
  ],

```

```
ystemctl restart xray

journalctl -u xray
```

Убеждаемся что нет ошибок, должно быть что-то вроде 
```
Aug 11 19:10:55 vds123.my-ihor.ru xray[77589]: 2023/08/11 19:10:55 [Info] infra/conf/serial: Reading config: /opt/xray/config.json
Aug 11 19:10:55 vds123.my-ihor.ru xray[77589]: 2023/08/11 19:10:55 [Info] transport/internet/tcp: listening TCP on 0.0.0.0:443
Aug 11 19:10:55 vds123.my-ihor.ru xray[77589]: 2023/08/11 19:10:55 [Warning] core: Xray 1.8.1 started
```

## Клиенты


Во всех клиентах настройка +- одна и та же: 
- указываем ip адрес
- порт (443)
- протокол VLESS
- включен XTLS
- выбран TCP траснпорт (он же None),
- UUID - он у нас сохранен ( вывод команды `/opt/xray/xray uuid`)
- Public Key  из (`/opt/xray/xray x25519`)
- ShortID из (`openssl rand -hex 8`) 

Если есть вариант врубить UDP Relay и TCP Fast Open, то врубаем оба.



Для ios:
- самый крутой клиент shadowrocket, хоть и стоит 3$

Для Linux/Windows:
- [nekoray](https://github.com/MatsuriDayo/nekoray)
- [sing-box](https://sing-box.sagernet.org/)

Для openwrt:
- [luci-app-xray](https://github.com/yichya/luci-app-xray)
- [openwrt-passwall](https://github.com/xiaorouji/openwrt-passwall)
- [luci-app-vssr](https://github.com/jerrykuku/luci-app-vssr)
- [helloworld](https://github.com/fw876/helloworld)

Для Android:
- [v2rayNG](https://github.com/2dust/v2rayNG)

Для MacOS:
- [V2RayXS](https://github.com/tzmax/V2RayXS)

## Reference:

- [XRay](https://github.com/XTLS/Xray-core)
- [Документация проекта Xray](https://xtls.github.io/en/)
- [Bleeding-edge обход блокировок с полной маскировкой: настраиваем сервер и клиент XRay с XTLS-Reality быстро и просто](https://habr.com/ru/articles/731608/)
- [Интернет-цензура и обход блокировок: не время расслабляться](https://habr.com/ru/articles/710980/)
- [active probbing или как китайский файрволл палит твои VPN сервисы](https://ensa.fi/active-probing/)
- [Крутой ресерч большого китайского файрволла](https://ensa.fi/active-probing/imc2015.pdf)

