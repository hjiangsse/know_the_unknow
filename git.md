# 1. Git configuration
## 1.1 config git to use sock5 as proxy
### set proxy
``` bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```
### to disable the proxy
``` bash
git config --global --unset http.proxy
```
