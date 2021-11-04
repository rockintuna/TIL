## Nginx

엔진엑스(Nginx)는 Igor Sysoev라는 러시아 개발자가 `동시접속 처리에 특화된` 웹 서버 프로그램이다. `Apache`보다 동작이 단순하고, 전달자 역할만 하기 때문에 동시접속 처리에 특화되어 있다.

#### 1. 정적 파일을 처리하는 HTTP 서버로서의 역할

웹서버의 역할은 HTML, CSS, Javascript, 이미지와 같은 정보를 웹 브라우저(Chrome, Iexplore, Opera, Firefox 등)에 전송하는 역할을 한다. (HTTP 프로토콜을 준수)

#### 2. 응용프로그램 서버에 요청을 보내는 리버스 프록시 역할

![스크린샷, 2017-07-03 20-49-56](http://i.imgur.com/yReDKjj.png)
클라이언트가 프록시 서버에 요청하고 프록시 서버가 배후 서버(reverse server)로부터 데이터를 가져오는 역할을 한다. 
여기서 프록시 서버가 `Nginx`, 리버스 서버가 `응용프로그램 서버`를 의미한다.

웹 응용프로그램 서버에 리버스 프록시(Nginx)를 두는 이유는 요청(request)에 대한 버퍼링이 있기 때문이다. 클라이언트가 직접 App 서버에 요청하는 경우, 프로세스 1개가 응답 대기 상태가 되어야만 한다. 따라서 프록시 서버를 둠으로써 요청을 `배분`하는 역할을 한다.

### Nginx를 사용하여 무중단 배포하기

#### 시나리오

참고 : https://jojoldu.tistory.com/267

EC2에 Nginx 1대와 스프링부트 jar 2대를 사용할 것이기 때문에 배포를 위해 AWS EC2 인스턴스가 더 필요하지 않는다.

![구조1](https://t1.daumcdn.net/cfile/tistory/997A14375A73F91D04)

스프링부트1만 Nginx에 연결했다고 가정하고

**배포 전 시나리오**

- 사용자가 주소로 접속 (80, 443 포트)
- Nginx는 사용자의 요청을 받아 현재 연결된 스프링부트(1)로 요청을 전달
- 스프링부트2는 Nginx와 연결된 상태가 아니므로 요청을 받지 않는다.

**배포 중 시나리오**

- 이후에 신규 릴리즈 배포가 필요하다면 스프링부트2로 배포한다.

- Nginx가 여전히 스프링부트1과 연결중이므로 다운타임은 없다.
- 배포가 끝나고 스프링부트2가 정상적으로 구동중인지 확인한 뒤 연결을 변경한다. `nginx reload`
- 이후 들어오는 요청은 스프링부트2로 전달한다. (문제 발생시 스프링부트1로 롤백 가능)



#### 환경 구축 (ubuntu)

Nginx 설치

```bash
$ sudo apt-get install nginx
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  linux-aws-5.4-headers-5.4.0-1054 linux-aws-5.4-headers-5.4.0-1055 linux-aws-5.4-headers-5.4.0-1056
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  libgd3 libnginx-mod-http-geoip libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libwebp6
  nginx-common nginx-core
Suggested packages:
  libgd-tools fcgiwrap nginx-doc ssl-cert
The following NEW packages will be installed:
  libgd3 libnginx-mod-http-geoip libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libwebp6 nginx
  nginx-common nginx-core
0 upgraded, 10 newly installed, 0 to remove and 43 not upgraded.
Need to get 902 kB of archives.
After this operation, 3019 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

설치가 끝나면 Nginx가 실행되어 있다.

```bash
$ ps -ef | grep nginx
root      2490     1  0 18:00 ?        00:00:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data  2493  2490  0 18:00 ?        00:00:00 nginx: worker process
```

Nginx 버전 확인

```bash
$ nginx -V
nginx version: nginx/1.14.0 (Ubuntu)
built with OpenSSL 1.1.1  11 Sep 2018
TLS SNI support enabled
```

전에 스프링부트로 포워딩 할 때 사용하던 포트 포워딩 해제

```bash
$ sudo iptables -t nat -D PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
$ sudo iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination  
```



브라우저에서 EC2의 80 포트로 접속하면 Nginx 페이지를 확인할 수 있다.

![image-20211102182130631](image-20211102182130631.png)

Nginx 설정 파일

```bash
$ cd /etc/nginx/
$ ls
conf.d        fastcgi_params  koi-win     modules-available  nginx.conf    scgi_params      sites-enabled  uwsgi_params
fastcgi.conf  koi-utf         mime.types  modules-enabled    proxy_params  sites-available  snippets       win-utf
$ vi nginx.conf 
```

