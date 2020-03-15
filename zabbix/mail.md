# zabix 邮件配置

## 坑


### 授权验证方式AUTH只支持PALIN

> zabbix_server 邮箱直到4.2.6还不支持修改smtp 的AUTH方法，只支持AUTH PALIN，导致采用AUTH LOGIN 作为登陆验证方式的smtp服务器无法验证，比如qq邮箱

#### 表现

> 报警媒介类型-电子邮件，设置完以后，点击测试，会发现始终返回login fail的错误。查看zabbix_server 日志也没有具体报错信息。
> 查阅zabbix官网了解到，zabbix邮件发送是调用的curl服务。采用如下代码使用curl做smtp登陆测试:

```bash
➜  ~ curl --url 'smtps://smtp.qq.com:465' --ssl-reqd \
--mail-from 'urmail@qq.com' --mail-rcpt 'target@icloud.com' \
--upload-file ./email_test.txt --user 'urmail@qq.com:urpwd' --insecure -kv
* Expire in 0 ms for 6 (transfer 0x9b8880)
* Expire in 1 ms for 1 (transfer 0x9b8880)
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Expire in 0 ms for 1 (transfer 0x9b8880)
* Expire in 0 ms for 1 (transfer 0x9b8880)
*   Trying 183.3.225.42...
* TCP_NODELAY set
* Expire in 149998 ms for 3 (transfer 0x9b8880)
* Expire in 200 ms for 4 (transfer 0x9b8880)
* Connected to smtp.qq.com (183.3.225.42) port 465 (#0)
* successfully set certificate verify locations:
*   CAfile: none
  CApath: /etc/ssl/certs
} [5 bytes data]
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
} [512 bytes data]
* TLSv1.3 (IN), TLS handshake, Server hello (2):
{ [93 bytes data]
* TLSv1.2 (IN), TLS handshake, Certificate (11):
{ [3840 bytes data]
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
{ [333 bytes data]
* TLSv1.2 (IN), TLS handshake, Server finished (14):
{ [4 bytes data]
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
} [70 bytes data]
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
} [1 bytes data]
* TLSv1.2 (OUT), TLS handshake, Finished (20):
} [16 bytes data]
* TLSv1.2 (IN), TLS handshake, Finished (20):
{ [16 bytes data]
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* Server certificate:
*  subject: C=CN; ST=guangdong; L=shenzhen; O=Tencent Technology (Shenzhen) Company Limited; CN=*.mail.qq.com
*  start date: Nov 11 10:32:16 2019 GMT
*  expire date: Jun  3 04:00:33 2020 GMT
*  issuer: C=BE; O=GlobalSign nv-sa; CN=GlobalSign Organization Validation CA - SHA256 - G2
*  SSL certificate verify ok.
{ [5 bytes data]
< 220 newxmesmtplogicsvrsza1.qq.com XMail Esmtp QQ Mail Server.
} [5 bytes data]
> EHLO email_test.txt
{ [5 bytes data]
< 250-newxmesmtplogicsvrsza1.qq.com
< 250-PIPELINING
< 250-SIZE 73400320
< 250-AUTH LOGIN PLAIN
< 250-AUTH=LOGIN
< 250-MAILCOMPRESS
< 250 8BITMIME
} [5 bytes data]
> AUTH PLAIN
{ [5 bytes data]
< 502 Invalid input from 183.17.224.244 to newxmesmtplogicsvrsza1.qq.com.
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Closing connection 0
curl: (67) Login denied

```
> 网上搜索使用telnet登陆qq smtp服务正常。

```bash
telnet smtp.qq.com 25
Trying 183.3.225.42...
Connected to smtp.qq.com.
Escape character is '^]'.
220 newxmesmtplogicsvrszb3.qq.com XMail Esmtp QQ Mail Server.
helo 192.168.0.103
250-newxmesmtplogicsvrszb3.qq.com-100.107.6.90-44064152
250-SIZE 73400320
250 OK
auth login
334 VXNlcm5hbWU6
BASE64(urmail)
334 UGFzc3dvcmQ6
BASE64(urpwd)
235 Authentication successful
mail from:<urmail@qq.com>
250 OK.
rcpt to:<target@icloud.com>
250 OK
data
354 End data with <CR><LF>.<CR><LF>.
from:<urmail@qq.com>
to:<target@icloud.com>
subject:hello telnet smtp!

telnet smpt qq mail test
.//
.
250 OK: queued as
```

> 猜想是AUTH PALAIN方法qq的smtp服务不支持导致，修改AUTH方式喂LOGIN，发送成功

```bash
➜  ~ curl --url 'smtps://smtp.qq.com:465' --ssl-reqd \
--mail-from 'urmail@qq.com' --mail-rcpt 'target@icloud.com' \
--upload-file ./email_test.txt --user 'urmail@qq.com:urpwd' --insecure -kv \
> --login-options AUTH=LOGIN
* Expire in 0 ms for 6 (transfer 0x1c76880)
* Expire in 1 ms for 1 (transfer 0x1c76880)
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Expire in 0 ms for 1 (transfer 0x1c76880)
* Expire in 0 ms for 1 (transfer 0x1c76880)
*   Trying 183.3.225.42...
* TCP_NODELAY set
* Expire in 149998 ms for 3 (transfer 0x1c76880)
* Expire in 200 ms for 4 (transfer 0x1c76880)
* Connected to smtp.qq.com (183.3.225.42) port 465 (#0)
* successfully set certificate verify locations:
*   CAfile: none
  CApath: /etc/ssl/certs
} [5 bytes data]
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
} [512 bytes data]
* TLSv1.3 (IN), TLS handshake, Server hello (2):
{ [93 bytes data]
* TLSv1.2 (IN), TLS handshake, Certificate (11):
{ [3840 bytes data]
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
{ [333 bytes data]
* TLSv1.2 (IN), TLS handshake, Server finished (14):
{ [4 bytes data]
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
} [70 bytes data]
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
} [1 bytes data]
* TLSv1.2 (OUT), TLS handshake, Finished (20):
} [16 bytes data]
* TLSv1.2 (IN), TLS handshake, Finished (20):
{ [16 bytes data]
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* Server certificate:
*  subject: C=CN; ST=guangdong; L=shenzhen; O=Tencent Technology (Shenzhen) Company Limited; CN=*.mail.qq.com
*  start date: Nov 11 10:32:16 2019 GMT
*  expire date: Jun  3 04:00:33 2020 GMT
*  issuer: C=BE; O=GlobalSign nv-sa; CN=GlobalSign Organization Validation CA - SHA256 - G2
*  SSL certificate verify ok.
{ [5 bytes data]
< 220 newxmesmtplogicsvrsza1.qq.com XMail Esmtp QQ Mail Server.
} [5 bytes data]
> EHLO email_test.txt
{ [5 bytes data]
< 250-newxmesmtplogicsvrsza1.qq.com
< 250-PIPELINING
< 250-SIZE 73400320
< 250-AUTH LOGIN PLAIN
< 250-AUTH=LOGIN
< 250-MAILCOMPRESS
< 250 8BITMIME
} [5 bytes data]
> AUTH LOGIN
{ [5 bytes data]
< 334 VXNlcm5hbWU6
} [5 bytes data]
> BASE64(urmail)
{ [5 bytes data]
< 334 UGFzc3dvcmQ6
} [5 bytes data]
> BASE64(urpwd)
{ [5 bytes data]
< 235 Authentication successful
} [5 bytes data]
> MAIL FROM:<urmail@qq.com> SIZE=163
{ [5 bytes data]
< 250 OK.
} [5 bytes data]
> RCPT TO:<target@icloud.com>
{ [5 bytes data]
< 250 OK
} [5 bytes data]
> DATA
{ [5 bytes data]
< 354 End data with <CR><LF>.<CR><LF>.
} [5 bytes data]
* We are completely uploaded and fine
} [5 bytes data]
< 250 OK: queued as.
100   163    0     0  100   163      0    278 --:--:-- --:--:-- --:--:--   278
* Connection #0 to host smtp.qq.com left intact

```

> 确定是AUTH PLAIN问题，再次查阅zabbix mail AUTH LOGIN，发现还没有实现。该特性会在[ZBXNEXT-3367](https://support.zabbix.com/browse/ZBXNEXT-3367)上实现。

## 解决方案

1. 用mailx，写好脚本配置后发送，网上很多教程，本人觉得太麻烦
2. 直接换邮箱，目前本人使用163邮箱，亲测可用。