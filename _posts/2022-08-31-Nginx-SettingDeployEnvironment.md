---

title: Ubuntu/Nginx/Certbot 서버 리버스 프록시 셋팅
author: 김도현
date: 2022-08-31
categories: [Ubuntu, Nginx, Certbot]
tags: [Ubuntu, Nginx, Certbot]
math: true
mermaid: true

---

# Ubuntu 에서 Nginx 설치하기

    sudo su

    apt update

    apt upgrade -y

    apt install nginx

---

# Ubuntu 에서 JDK 설치하기

    sudo apt-get install openjdk-11-jdk

---

# Ubuntu 에서 Java 환경변수 설정하기

    vi ~/.bashrc

## # ~/.bashrc 맨 아래에 추가할 내용

    export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
    export PATH=$PATH:$JAVA_HOME/bin

---

# Ubuntu 에서 Gradle 설치하기

    add-apt-repository ppa:cwchien/gradle
    apt update
    apt install gradle

---

# 프로젝트 파일 빌드 쉘 스크립트

    #!/bin/bash

    today=$(date "+%Y%m%d")

    for i in {1..50}
    do
        CreateDIR=\/home\/ubuntu\/source\/${today}-${i}
        if [ ! -d ${CreateDIR} ]; then
            mkdir ${today}-${i}
            cd ${today}-${i}
            git clone https://github.com/...
            cd ...
            chmod 777 gradlew
            cp ../../application.yml ./src/main/resources/application.yml
            ./gradlew build
            exit 0
    fi
    done


---

# 아래는 NGINX + Certbot 으로 리버스 프록싱 + SSL 인증서 적용 과정

---

# Ubuntu/Nginx 에서 Certbot 설치하기

    apt install certbot python3-certbot-nginx

# SSL 인증서를 받기위한 사전 작업

    mkdir -p /var/www/letsencrypt/.well-known/acme-challenge

    vi /etc/nginx/snippets/letsencrypt.conf

# Webroot 방식으로 SSL 인증서 획득하기

    certbot certonly --webroot --webroot-path=/var/www/letsencrypt  -d 사이트명.com

# /etc/nginx/snippets/letsencrypt.conf

    location ^~ /.well-known/acme-challenge/ {
        default_type "text/plain";
        root /var/www/letsencrypt;
    }

# vi /etc/nginx/sites-available/default

    server {

        listen 80 ;
        listen [::]:80 ;

        server_name test.domain.kr;

        if ($host test.domian.kr) {
            return 301 https://test.domain.kr:443$request_uri;
        }

        return 404;
    }

    server {
        include /etc/letsencrypt/options-ssl-nginx.conf;

        listen [::]:443 ssl ipv6only=on;
        listen 443 ssl;
        ssl_certificate /etc/letsencrypt/live/test.domain.kr/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/test.domain.kr/privkey.pem;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

        location / {
            proxy_pass http://1270.0.0.1:8080$request_uri;
        }
    }
