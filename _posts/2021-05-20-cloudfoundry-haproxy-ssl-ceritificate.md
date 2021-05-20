---
title: CloudFoundry - HAProxy SSL ceritificate
description:
search: false
categories:
  - cloudfoundry
tags:
  - cloudfoundry
  - bosh
  - haproxy
  - ssl
  - certificate
permalink:


---

## HAProxy ?

- The Reliable, High Performance TCP/HTTP Load Balancer **(Open Source)**

- initial release year : 2001
- stable release : 2.3.8 (March 25, 2021)
- HAProxy Enterprise/ALOHA/Edge/One/Kubernetes Ingress Controller ...


### HAProxy Config

An HAProxy configuration file guides the behavior of your HAProxy load balancer. ```global```, ```defaults```, ```frontend```, ```backend```. These four sections define how the server as a whole performs, what your default settings are, and how client requests are received and routed to your backend servers.

HAProxy 설정은 네 개의 필수 섹션을 포함하는 기본 설정 파일로 만들 수 있다.

```
**haproxy.cfg**

global
    # process-level settings here
    # 구성의 맨 위에 있어야 함

defaults
    # defaults here
    # 다음 frontend, backend, listen 섹션의 기본값

frontend
    # a frontend that accepts requests from clients
    # 클라이언트가 연결할 수있는 IP 주소와 포트

backend
    # servers that fulfill the requests
    # HAProxy Enterprise가 트래픽을 전달하는 서버 풀

listen
    # A listen section combines the functions of frontend and backend
    # 프론트 엔드와 백엔드의 기능을 결합
```

```
**haproxy.cfg**

global
    # Bind the Runtime API to a UNIX domain socket and/or an IP address
    stats socket /var/run/hapee-2.3/hapee-lb.sock user hapee-lb group hapee mode 660 level admin
    stats socket ipv4@*:9024 level admin

    # Set a timeout for how long you can stay connected to the Runtime API,
    # which is useful for longer interactive terminal sessions
    stats timeout 10m

    # Set remote or local Syslog server addresses to send log files to
    log 127.0.0.1 local0
    log 127.0.0.1 local1 notice

    # Set the user and group to run HAProxy Enterprise as
    user  hapee-lb
    group hapee

    # Run the process in a chrooted directory
    chroot /var/empty

    # Set the path from which to load Enterprise modules
    module-path /opt/hapee-2.3/modules

defaults
    # defaults here
    # 다음 frontend, backend, listen 섹션의 기본값

frontend www.example.com
    mode http
    bind *:80
    bind *:443 ssl crt /etc/hapee-2.3/certs/ssl.pem

    # Redirect HTTP to HTTPS
    http-request redirect scheme https unless { ssl_fc }

    default_backend web_servers

frontend foo.com
   mode http
   bind 192.168.1.5:80
   default_backend foo_servers

frontend db.foo.com
   mode tcp
   bind 192.168.1.15:3306
   default_backend db_servers



backend www.example2.com
    mode http
    balance roundrobin

    # Enable HTTP health checks
    option httpchk

    server s1 192.168.1.25:80 check
    server s2 192.168.1.26:8080 check
    server s3 192.168.1.27:8080 check

frontend foo_and_bar
   mode http
   bind 192.168.1.5:80
   use_backend foo_servers if { req.hdr(host) -i foo.com }
   use_backend bar_servers if { req.hdr(host) -i bar.com  }

backend foo_servers
   mode http
   server s1 192.168.1.25:80
   server s2 192.168.1.26:80
   server s3 192.168.1.27:80

backend bar_servers
   mode http
   server s1 192.168.1.35:80
   server s2 192.168.1.36:80
   server s3 192.168.1.37:80

listen fe_main
    # frontend configuration settings
    mode http
    bind :80
    bind :443 ssl crt /etc/hapee-2.3/certs/ssl.pem
    http-request redirect scheme https unless { ssl_fc }

    # backend configuration settings
    balance roundrobin
    option httpchk
    server s1 192.168.1.25:80 check
    server s2 192.168.1.26:8080 check

...
```




-----

> https://en.wikipedia.org/wiki/HAProxy  
> https://www.haproxy.com/documentation/hapee/latest/  
> https://www.haproxy.com/documentation/hapee/latest/configuration/config-sections/overview/  
> https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration/  
