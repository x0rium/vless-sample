# vless-sample

Ультра простой варинт настройки XTLS-Reality. Опробирован на Debian 11 на [VDS от iHor за 240 рублей (1 СPU/768 MB RAM/7 GB SSD) и Европейский IP](https://www.ihor-hosting.ru/?from=180121). Полет отличный.

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
Пихаем в него 

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
Смотрим что все стартануло 


Для ios самый крутой клиент shadowrocket



Reference:

- https://habr.com/ru/articles/731608/
- https://habr.com/ru/articles/710980/
- https://ensa.fi/active-probing/
- https://ensa.fi/active-probing/imc2015.pdf

