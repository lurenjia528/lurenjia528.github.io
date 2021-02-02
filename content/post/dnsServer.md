---
title: dnsServer-bind9
#slug: chinese-test
date: 2018-11-06
categories:
- bind9
- ubuntu
tags:
- bind9
thumbnailImagePosition: left
thumbnailImage: /img/dns.jpg
#coverImage: /img/docker-long.jpg
---
dnsServer bind9
<!--more-->

# dnsServer
bind9

ubuntu下

安装bind9

``` bsah
apt-get install bind9  
```

修改`/etc/bind/named.conf.options`文件

``` conf
acl goodclients {
        192.168.200.0/24;
        localhost;
        localhost;
};

options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      8.8.8.8;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };

        listen-on {192.168.200.222;};

        recursion yes;
        allow-query { goodclients; };
        allow-transfer { none; };

        forwarders {
                223.5.5.5;
                223.6.6.6;
        };
        forward only;
};
```   

配置`named.conf.local`文件
``` conf
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

//domain->ip
zone "local.com" in {
        type master;
        file "/var/cache/bind/db.local.com";
};

//ip->domain
zone "200.168.192.in-addr.arpa" in {
        type master;
        file "/var/cache/bind/db.200.168.192";
};
```

配置正向记录`/var/cache/bind/db.local.com` 

``` conf
$TTL    604800
@       IN      SOA     local.com.      root.local.com. (
                        2               ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        604000)         ; Negative Cache TTL
;
; name servers
@       IN      NS      ns.local.com.
@       IN      A       192.168.200.222
;ns records
ns      IN      A       192.168.200.222
;host records
www     IN      A       192.168.200.110
api     IN      A       192.168.200.100
ygt     IN      A       192.168.200.111
```

配置反向记录`/var/cache/bind/db.200.168.192`文件

PTR表示ip地址对应的域名，本例中：192.168.200.66对应三个域名

``` conf
$TTL    604800
@       IN      SOA     local.com.      root.local.com. (
                        2               ; Serial Number
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        86400 );        ; Minimum

@       IN      NS      local.com.

66      IN      PTR     www.local.com.
66      IN      PTR     api.local.com.
66      IN      PTR     ygt.local.com.
```

重启bind9

`systemctl restart bind9`

在路由器里设置首要DNS 为 192.168.200.222(同一内网下，配置`/etc/resolv.conf` 添加 `nameserver 192.168.200.222` 即可)，这样我们就可以在同一个内网下访问：www.local.com 就会指向到 192.168.200.110，访问：api.local.com 就会指向到 192.168.200.100

``` bash
root@ygt:/var/cache/bind# nslookup www.local.com
Server:         192.168.200.222
Address:        192.168.200.222#53

Name:   www.local.com
Address: 192.168.200.110

root@ygt:/var/cache/bind# nslookup api.local.com
Server:         192.168.200.222
Address:        192.168.200.222#53

Name:   api.local.com
Address: 192.168.200.100

root@ygt:/var/cache/bind# nslookup  192.168.200.66
Server:         192.168.200.222
Address:        192.168.200.222#53

66.200.168.192.in-addr.arpa     name = ygt.local.com.
66.200.168.192.in-addr.arpa     name = api.local.com.
66.200.168.192.in-addr.arpa     name = www.local.com.

root@ygt:/var/cache/bind#
```
