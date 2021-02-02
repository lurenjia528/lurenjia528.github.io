---
title: harbor基于Bearer的验证
#slug: chinese-test
date: 2020-05-28
categories:
- ubuntu
- harbor
tags:
- harbor
thumbnailImagePosition: left
thumbnailImage: /img/harbor-1.png
coverImage: /img/harbor-2.jpg
---
harbor基于Bearer的验证
<!--more-->



# 获取token服务地址
```bash
ht061@HT061:~/gocode/src/github.com/goharbor/harbor$ curl -v -k  https://ygt.harbor.csse:443/v2/
*   Trying 192.168.17.187...
* TCP_NODELAY set
* Connected to ygt.harbor.csse (192.168.17.187) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* (304) (OUT), TLS handshake, Client hello (1):
* (304) (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: C=CN; ST=Beijing; L=Beijing; O=example; OU=Personal; CN=ygt.harbor.csse
*  start date: May 21 06:32:46 2020 GMT
*  expire date: May 19 06:32:46 2030 GMT
*  issuer: C=CN; ST=Beijing; L=Beijing; O=example; OU=Personal; CN=ygt.harbor.csse
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
> GET /v2/ HTTP/1.1
> Host: ygt.harbor.csse
> User-Agent: curl/7.58.0
> Accept: */*
> 
< HTTP/1.1 401 Unauthorized
< Server: nginx
< Date: Thu, 28 May 2020 02:04:57 GMT
< Content-Type: application/json; charset=utf-8
< Content-Length: 76
< Connection: keep-alive
< Docker-Distribution-Api-Version: registry/2.0
< Set-Cookie: sid=ca6f870d125f914ff6ff02d6336ea538; Path=/; HttpOnly
< Www-Authenticate: Bearer realm="https://ygt.harbor.csse/service/token",service="harbor-registry"
< X-Request-Id: 74ca1dab-f86c-4fa1-af48-914498f15ee3
< 
{"errors":[{"code":"UNAUTHORIZED","message":"unauthorized: unauthorized"}]}

```

# 获取token

```bash
ht061@HT061:~/gocode/src/github.com/goharbor/harbor$ curl -ikL -u admin:Harbor12345 https://ygt.harbor.csse/service/token?account=admin\&service=harbor-registry\&scope=repository:timi/busybox:pull,push
HTTP/1.1 200 OK
Server: nginx
Date: Thu, 28 May 2020 02:10:11 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 1216
Connection: keep-alive
Set-Cookie: sid=85b566e4b93612208b0af1e879b648b3; Path=/; Secure; HttpOnly
X-Request-Id: 9f22b1c7-4e94-44cc-8f33-b788ba5de1a3
Strict-Transport-Security: max-age=31536000; includeSubdomains; preload
X-Frame-Options: DENY
Content-Security-Policy: frame-ancestors 'none'

{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IlBEWUI6M0QzVzpDTU1MOlJWWkQ6NUg1VzpVT1ZDOjVHR1E6SDVQUDpHRlpGOjU2T086WkRTMjo2Uk5aIn0.eyJpc3MiOiJoYXJib3ItdG9rZW4taXNzdWVyIiwic3ViIjoiYWRtaW4iLCJhdWQiOiJoYXJib3ItcmVnaXN0cnkiLCJleHAiOjE1OTA2MzM2MTEsIm5iZiI6MTU5MDYzMTgxMSwiaWF0IjoxNTkwNjMxODExLCJqdGkiOiJkS3o1N2JDdXVlRVRkMDBOIiwiYWNjZXNzIjpbeyJ0eXBlIjoicmVwb3NpdG9yeSIsIm5hbWUiOiJ0aW1pL2J1c3lib3giLCJhY3Rpb25zIjpbInB1c2giLCIqIiwicHVsbCJdfV19.ACPiiLq8ammca9xldAB4cWnS-EtSJoPt0N0SsOCM6Z8DYDfBJAvpwYNlXBFUPSaqBQZD044ocaOAoq-H4G3wOYMKZp5feXqurtJfwRzsEMWWbbo4hK7I4Xoom0jFKMtTzbrtdJNQCi9BwCOii7k62y3yqLqkGyibOz_tXw6-h2a6FODURCr4oVxecVPPtrzoowrNKgPfEHygC3Sowi4UzYUxPGj66uVWraB7t1oIIJmvj0FYicQ9nAqT0mRfEMGhtWhGacNetozieV6ZsxENmPxU4p3Al4Dv9wQ55NLGg5eXBcH0UoZiMEzheh4OsZq19decCUaxKkQwnW0VbtYV9FxjRP7OfeKwIS0nDJE3TKXEdLvK6Yf3OX40F6GXYB7PA5S01fr1yTfDkjdtpc-MdB5jqHhTBvXG3jQgLrZrGuSsFCR_KX8Z0aNJhtmamKlFWIUGDGnMw7GHVjBPgldkGKUHK94mebKCLZ2x4K6SgOjhij5KcR7oAo9kidzp_53nk4jFnYUQAxEWf-1Hue4pqGx_YgsdzMBgwWqLh-S8NFzTxxxT8xYRDSYMetr0GWZMjeQzIP8m4Eu9GFnTkF0iNufepf1qbrR2_2yXTZy224BT23MGlr-ZP99wdDm05b3Nf7uPSlc0ioEnc2mqjWrLJK5Pmj3qOIlhOmdq2oVTBdY",
  "access_token": "",
  "expires_in": 1800,
  "issued_at": "2020-05-28T02:10:11Z"

```


# 使用token

```bash
 curl -k -H "Accept:application/vnd.docker.distribution.manifest.v2+json" -H "HOST:ygt.harbor.csse" -H "Authorization:Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IlBEWUI6M0QzVzpDTU1MOlJWWkQ6NUg1VzpVT1ZDOjVHR1E6SDVQUDpHRlpGOjU2T086WkRTMjo2Uk5aIn0.eyJpc3MiOiJoYXJib3ItdG9rZW4taXNzdWVyIiwic3ViIjoiYWRtaW4iLCJhdWQiOiJoYXJib3ItcmVnaXN0cnkiLCJleHAiOjE1OTA2MzM2MTEsIm5iZiI6MTU5MDYzMTgxMSwiaWF0IjoxNTkwNjMxODExLCJqdGkiOiJkS3o1N2JDdXVlRVRkMDBOIiwiYWNjZXNzIjpbeyJ0eXBlIjoicmVwb3NpdG9yeSIsIm5hbWUiOiJ0aW1pL2J1c3lib3giLCJhY3Rpb25zIjpbInB1c2giLCIqIiwicHVsbCJdfV19.ACPiiLq8ammca9xldAB4cWnS-EtSJoPt0N0SsOCM6Z8DYDfBJAvpwYNlXBFUPSaqBQZD044ocaOAoq-H4G3wOYMKZp5feXqurtJfwRzsEMWWbbo4hK7I4Xoom0jFKMtTzbrtdJNQCi9BwCOii7k62y3yqLqkGyibOz_tXw6-h2a6FODURCr4oVxecVPPtrzoowrNKgPfEHygC3Sowi4UzYUxPGj66uVWraB7t1oIIJmvj0FYicQ9nAqT0mRfEMGhtWhGacNetozieV6ZsxENmPxU4p3Al4Dv9wQ55NLGg5eXBcH0UoZiMEzheh4OsZq19decCUaxKkQwnW0VbtYV9FxjRP7OfeKwIS0nDJE3TKXEdLvK6Yf3OX40F6GXYB7PA5S01fr1yTfDkjdtpc-MdB5jqHhTBvXG3jQgLrZrGuSsFCR_KX8Z0aNJhtmamKlFWIUGDGnMw7GHVjBPgldkGKUHK94mebKCLZ2x4K6SgOjhij5KcR7oAo9kidzp_53nk4jFnYUQAxEWf-1Hue4pqGx_YgsdzMBgwWqLh-S8NFzTxxxT8xYRDSYMetr0GWZMjeQzIP8m4Eu9GFnTkF0iNufepf1qbrR2_2yXTZy224BT23MGlr-ZP99wdDm05b3Nf7uPSlc0ioEnc2mqjWrLJK5Pmj3qOIlhOmdq2oVTBdY" https://ygt.harbor.csse/v2/timi/busybox/manifests/0.0.1
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
   "manifests": [
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 527,
         "digest": "sha256:186694df7e479d2b8bf075d9e1b1d7a884c6de60470006d572350573bfa6dcd2",
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 527,
         "digest": "sha256:2ca549a51078fb35f0c5d1dca103b7d58523f1998a7f78a489c16dd38a5c5cf9",
         "platform": {
            "architecture": "arm64",
            "os": "linux"
         }
      }
   ]
}
```

# 获取 digest(sha256)
```bash
 curl  -k --head -H "Accept:application/vnd.docker.distribution.manifest.v2+json" -H "HOST:ygt.harbor.csse" -H "Authorization:Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IlBEWUI6M0QzVzpDTU1MOlJWWkQ6NUg1VzpVT1ZDOjVHR1E6SDVQUDpHRlpGOjU2T086WkRTMjo2Uk5aIn0.eyJpc3MiOiJoYXJib3ItdG9rZW4taXNzdWVyIiwic3ViIjoiYWRtaW4iLCJhdWQiOiJoYXJib3ItcmVnaXN0cnkiLCJleHAiOjE1OTA2MzU2ODIsIm5iZiI6MTU5MDYzMzg4MiwiaWF0IjoxNTkwNjMzODgyLCJqdGkiOiJTNlQxVFh5a3VYSTRiVnhIIiwiYWNjZXNzIjpbeyJ0eXBlIjoicmVwb3NpdG9yeSIsIm5hbWUiOiJ0aW1pL2J1c3lib3giLCJhY3Rpb25zIjpbInB1c2giLCIqIiwicHVsbCJdfV19.wBeHRSVAyIoxerrwOVwJ6G_tMiZWCvYk96OIqJIg9wIq8asMP1wDvtuBhGGghH8yDhfBy6I2QKhQwXbO3S88SGiNo6p3eVbUJCEpxrj-GOE1ISjhRgbJv6s1_kQEbcQdN6tJ2uaMApgix2j83smlzkFHL_L5a-wrZz_qgPdu_EGPrlbg0SZHGp5ZOLvILgn_kRExUti5rPuRLlSe-10waFpfCQ0b0ugKQsbm-hB3mlDU9LJsJV80znDVO79loMps7C3mZytSaGDKlOJtPDa1IujDGAhA6x6QKyIVJW1wTJsHvLJE8HW9BJS1230eRKZhT3wjw9ZiwmhmHyE9s-8ExD51qyn-eMIVqO_-XOQwHTJPswSOBe-twCjgzGKtXbxC20XDJtJlbVvR6xIH_TzzO39WSMuu6lfh0KU8DSjpFBKkJBwwB_pNAWFEWBEEGRDDfQXo8U4CQm-vkDPIffY6j8zt43wi1FjfnCh3qFirGyNmDmXxoeXrB2OwVIr3uCwN6MwzxGz3EAjQWJAqgUt5PjLJRI-3rjwAuSVkyU1XOl3ZLVpyg3i4I8SnB114gs0UGjzk8XE3qurXX6B9_HoL0B_yezIRg8YbqnnWpBTDnqoLut4fXpIrK0Fit8aa0HX1gA3I1C_nYR33InD1np9zcfXiV7_BErZCc6PV8ieWyRo" https://ygt.harbor.csse/v2/timi/busybox/manifests/0.0.1 
HTTP/1.1 200 OK
Server: nginx
Date: Thu, 28 May 2020 02:45:04 GMT
Content-Type: application/vnd.docker.distribution.manifest.list.v2+json
Content-Length: 741
Connection: keep-alive
Docker-Content-Digest: sha256:3c893fa3d4b68055de597660a478f9f25c208c7ea0a9f733dd765e65802960d9
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:3c893fa3d4b68055de597660a478f9f25c208c7ea0a9f733dd765e65802960d9"
Set-Cookie: sid=7aaf6c901aef47d51eefe5b66c12fd11; Path=/; HttpOnly
X-Request-Id: 3a5d05bf-3dc1-4157-820d-66da1b5a58a4
Strict-Transport-Security: max-age=31536000; includeSubdomains; preload
X-Frame-Options: DENY
Content-Security-Policy: frame-ancestors 'none'

```

# 删除manifest

```bash
 curl -v  -k -X DELETE -H "Accept:application/vnd.docker.distribution.manifest.v2+json" -H "HOST:ygt.harbor.csse" -H "Authorization:Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IlBEWUI6M0QzVzpDTU1MOlJWWkQ6NUg1VzpVT1ZDOjVHR1E6SDVQUDpHRlpGOjU2T086WkRTMjo2Uk5aIn0.eyJpc3MiOiJoYXJib3ItdG9rZW4taXNzdWVyIiwic3ViIjoiYWRtaW4iLCJhdWQiOiJoYXJib3ItcmVnaXN0cnkiLCJleHAiOjE1OTA2MzU2ODIsIm5iZiI6MTU5MDYzMzg4MiwiaWF0IjoxNTkwNjMzODgyLCJqdGkiOiJTNlQxVFh5a3VYSTRiVnhIIiwiYWNjZXNzIjpbeyJ0eXBlIjoicmVwb3NpdG9yeSIsIm5hbWUiOiJ0aW1pL2J1c3lib3giLCJhY3Rpb25zIjpbInB1c2giLCIqIiwicHVsbCJdfV19.wBeHRSVAyIoxerrwOVwJ6G_tMiZWCvYk96OIqJIg9wIq8asMP1wDvtuBhGGghH8yDhfBy6I2QKhQwXbO3S88SGiNo6p3eVbUJCEpxrj-GOE1ISjhRgbJv6s1_kQEbcQdN6tJ2uaMApgix2j83smlzkFHL_L5a-wrZz_qgPdu_EGPrlbg0SZHGp5ZOLvILgn_kRExUti5rPuRLlSe-10waFpfCQ0b0ugKQsbm-hB3mlDU9LJsJV80znDVO79loMps7C3mZytSaGDKlOJtPDa1IujDGAhA6x6QKyIVJW1wTJsHvLJE8HW9BJS1230eRKZhT3wjw9ZiwmhmHyE9s-8ExD51qyn-eMIVqO_-XOQwHTJPswSOBe-twCjgzGKtXbxC20XDJtJlbVvR6xIH_TzzO39WSMuu6lfh0KU8DSjpFBKkJBwwB_pNAWFEWBEEGRDDfQXo8U4CQm-vkDPIffY6j8zt43wi1FjfnCh3qFirGyNmDmXxoeXrB2OwVIr3uCwN6MwzxGz3EAjQWJAqgUt5PjLJRI-3rjwAuSVkyU1XOl3ZLVpyg3i4I8SnB114gs0UGjzk8XE3qurXX6B9_HoL0B_yezIRg8YbqnnWpBTDnqoLut4fXpIrK0Fit8aa0HX1gA3I1C_nYR33InD1np9zcfXiV7_BErZCc6PV8ieWyRo" https://ygt.harbor.csse/v2/timi/busybox/manifests/sha256:3c893fa3d4b68055de597660a478f9f25c208c7ea0a9f733dd765e65802960d9

```


[参考](https://ivanzz1001.github.io/records/post/docker/2018/04/09/docker-harbor-auth-Bearer)
