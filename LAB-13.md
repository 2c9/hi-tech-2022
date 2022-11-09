
```bash
cat /etc/squid/squid.conf
```

Перед разрешаюзими правилами добавить

```
acl blacklist dstdom_regex -i "/etc/squid3/blacklist"
http_access deny blacklist
```

```bash
cat /etc/squid3/blacklist
```

```
ya.ru
google.com
```
