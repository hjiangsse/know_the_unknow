# 1. Install and config v2ray
## 1.1 Use script auto install v2ray
use script in ../resources/install-release.sh
## 1.2 Client config for v2ray:
```
{
    "inbounds": [{
      "port": 1080,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
	"destOverride": ["http", "tls"]
      },
      "settings": {
	 "udp": false,
	 "auth": "noauth",
	 "ip": "127.0.0.1"
      }
    }],
    "outbounds": [{
      "protocol": "vmess",
      "settings": {
         "vnext": [{
            "address": "204.124.181.100",
	    "port": 16681,
	    "users": [{ 
		    "id": "db9067d4-9e67-4e40-8038-448a22dd4cb5",
		    "alterId": 8
	    }]
	 }]
      }
    },{
      "protocol": "freedom",
      "tag": "direct",
      "settings": {}
    }],
    "routing": {
      "domainStrategy": "IPOnDemand",
      "rules": [{
	"type": "field",
	"ip": ["geoip:private"],
	"outboundTag": "direct"
      }]
    }
}
```
## 1.3 Start the v2ray service:
``` bash
sudo systemctl enable v2ray
sudo systemctl start v2ray
```
# 2. Capslock to Left Control
add the following in the tail of ~/.bashrc 
``` bash
setxkbmap -layout us -option ctrl:swapcaps
```
then source it or reboot the mechine.
