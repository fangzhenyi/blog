---
title: caddy自动配置https，并设置basic_auth
date: 2023-04-05T22:38:35.000Z
tags: caddy https
---
### caddy配置https

修改Caddyfile如下即可：

```
https://xxxxx.com {
  reverse_proxy http://172.24.0.3:3000
}
```



### caddy配置https并设置basic_auth

```
https://xxxxx.com {
    basicauth /* {
        test JDJhJDE0JFhHMS5vTDRqcmJ6UWhxL0R5RFdlVWVVNXdKOXZ5NGNPc1l0QWpmZmtLTVhqTnBVS0o5RkQu
        acb JDJhJDE0JFhHMS5vTDRqcmJ6UWhxL0R5RFdlVWVVNXdKOXZ5NGNPc1l0QWpmZmtLTVhqTnBVS0o5RkQu
    }
  reverse_proxy http://172.24.0.3:3000
}
```

caddy的密码必须通过caddy  hash-password命令去获取密文密码才可以，本文所采用的的密码为123456
