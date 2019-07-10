---
title: "NGINX : 꼭 알아야 할 configuration 기초 개념!"
categories: tech
tags:
- NGINX
author_profile: true
---

NGINX를 사용하는데 있어 꼭 필요한 기초 개념들을 정리한 글이다. 전반적인 기초를 다룬 뒤 NIGNX를 web server로 이용하는 방법에 초점을 두어 작성하였다. 또한 너무 자세한 내용은 다루지 않았다.

## NGINX 시작, 종료, 재시작

NGINX의 파일을 invoke 하기 위해 -s 옵션을 사용한다. 다음과 같이 사용한다.

    $ nginx -s [signal]

[signal]에는 다음 4가지 설정이 가능하다.

- **stop** : fast shutdown
- **quit** : graceful shutdown - 이 명령어는 NGINX를 시작한 사람만 사용할 수 있다.
- **reload**  : reloading config file
- **reopen** : reopening the log files

즉, 새로 바꾼 configuration 파일을 적용하고 싶다면 다음과 같이 이용해야 한다.

    $ nginx -s reload

**Note**<br><br>물론 **kill** 과 같은 명령어를 이용하여 프로세스를 직접 종료할 수도 있다. 이때 NGINX의 PID는 **/usr/local/nginx/logs** 또는 **/var/run** 에서 확인할 수 있다.
{: .notice--info}

## Configuration File 구조 분석하기

NGINX의 기본 설정 파일은 **nginx.conf** 이며 다음 **/usr/local/nginx/conf, /etc/nginx, or /usr/local/etc/nginx** 경로 중 하나에 있다. (혹시 기본적인 linux file system을 모른다면 [다음 글](https://seonkyukim.github.io/tech/Linux 완전 기초! 파일 시스템, 프로세스 및 서비스, 로그파일 관리까지!/)의 '리눅스 파일 시스템' 부분을 보고오자).

NGINX의 모듈들은 configuration 파일에 있는 **directives**에 의해 제어된다. 먼저 기본적으로 주어지는 nginx.conf 파일은 다음과 같이 생겼다. 

    user www-data;
    worker_processes auto;
    pid /run/nginx.pid;
    include /etc/nginx/modules-enabled/*.conf;
    
    events {
            worker_connections 768;
            # multi_accept on;
    }
    
    http {
    
            ...
    
            include /etc/nginx/conf.d/*.conf;
            include /etc/nginx/sites-enabled/*;
    }

directives에는 두 가지 종류가 있다. 위의 네 줄은 **simple directive**, 아래 { } 로 둘러쌓인 것은 **block directive**이다.

**simple directive**

이름, 인자값이 있고 세미콜론(;)으로 끝난다.

- **user** : linux 시스템의 어떤 사용자가 nginx 서버를 동작시킬지 기술한다.
- **worker_processes** : 몇 개의 thread 가 사용될지 정의한다. CPU 코어 수에 맞추는 것이 권장된다.
- **pid** : nginx pid가 적혀있는 파일이다.
- **include** : 외부 configuration 내용을 가져온다. 모듈에 따라 다른 파일에 작성하고 Include 하는 것이 권장된다.

**block directive**

simple directive와 구성이 같지만 세미콜론 대신 추가적인 내용들이 { } 안에 들어간다. 위 예시에서 event 블럭과 http 블럭이 있다.

### Include Configuration files

configuration 파일들을 쉽게 관리하기 위해 기능들 단위로 파일을 나눠 저장하는 것이 좋다. 파일을 **/etc/nginx/conf.d** 디렉토리에 저장한 후 include를 이용하여 configuration 파일을 불러오자. 이때, block directive 안에서 include를 쓸 경우, 해당 블럭 안에 파일 내용이 포함된다.

### Context

block directive 안에 다른 directive가 들어가 있는 경우 **context**라고 하며, 대표적으로 events, http, server, location directive들이 있다. 바깥의 directive들은 **main** **context** 안에 있는 것으로 간주한다.

- **events** : 일반적인 connection process를 담당한다.
- **http** : HTTP traffic
- **mail** : Mail traffic
- **stream** : TCP and UDP traffic

### Virtual Servers

트래픽을 다루는 context에서 request를 처리하기 위해 한 개 이상의 server block을 추가해야 한다. 트래픽 타입에 따라 server context 안에 추가할 수 있는 directives들이 다르다.

가상 호스트?<br><br>혹시라도 가상 호스트의 개념을 잘 모른다면 보고 오자: [가상 호스트](https://opentutorials.org/module/384/4529)
{: .notice--info}

**http context** 안에 있는 server directive는 특정 도메인이나 IP 주소로의 요청을 처리한다. server context 안에 있는 location context가 특정 URI set을 어떻게 처리할지 정한다.

다음은 전체 configuration file의 예시이다.

    user nobody; # a directive in the 'main' context
    
    events {
        # configuration of connection processing
        worker_connections 768;
    }
    
    http {
        # Configuration specific to HTTP and affecting all virtual servers  
    
        server {
            # configuration of HTTP virtual server 1       
            location /one {
                # configuration for processing URIs starting with '/one'
            }
            location /two {
                # configuration for processing URIs starting with '/two'
            }
        } 
        
        server {
            # configuration of HTTP virtual server 2
        }
    }
    
    stream {
        # Configuration specific to TCP/UDP and affecting all virtual servers
        server {
            # configuration of TCP virtual server 1 
        }
    }

- **worker_connections** : worker process 하나 당 몇 개의 connection을 처리할 지 정한다.
- **location** : 처리할 URI 형식을 표시한다.

### directive 상속

하나의 context(부모) 안에 있는 또 다른 context(자식)는 상위 레벨의 directive들을 상속받는다. 몇 가지 directives들은 여러 개의 context에서 동시에 나타날 수 있는데, 이럴 경우 자식 directive는 부모 설정을 override하게 된다.

## Web Server로 NGINX 사용하기

### 가상 서버 세팅하기

NGINX configuration 파일에는 적어도 하나의 server directive가 있어야 한다. NGINX가 요청을 처리할 때, 가장 먼저 요청을 처리할 virtual server를 선택한다. 위 예시처럼 하나의 http context안에 여러 개의 server directive가 있을 수 있다.

보통 server configuration block에는 요청을 listen할 특정 **IP address**와 **포트**, 혹은 **Unix domain socket**과 **path**를 적어둔다. 

아래 예시는 IP 주소 127.0.0.1 과 포트 8080으로 listen하는 configuration이다.

    server {
        listen 127.0.0.1:8080;
        # The rest of server configuration
    }

또는 **server_name** directive를 사용하여 도메인 명을 사용할 수도 있다. 이때 NGINX는 요청의 Host header의 필드값과 server_name을 비교하여 맞는 server를 찾는다. server name으로는 wild card와 정규식 등을 사용할 수 있다.

    server {
        listen      80;
        server_name example.org www.example.org;
        ...
    }

### Location 세팅하기

NGINX는 요청 URI에 따라 다른 서버로 트래픽을 전송한다. 이는 server block 안의 **location directive**를 이용해 정할 수 있다. 여러 location 블럭을 사용할 수도 있고, location 안에 다른 location directive를 설정할 수도 있다.

location은 URI 경로의 일부인 **prefix string**이거나 **정규식** 표현이 될 수 있다. 다음 예시는 /some/path/document.html 과 같은 경로의 요청을 처리한다.

    location /some/path/ {
        ...
    }

location context 안에 있는 directive는 요청을 어떻게 처리할지 정할 수 있다. **static file**을 보여주거나 **proxy server**로 요청을 전송한다. 다음 예시의 첫 번째 location context와 일치하는 패턴은 **/data** 디렉토리에서 파일들을 보여준다. 두 번째 location context와 일치하는 패턴은 **www.example.com** 도메인으로 요청을 전송한다. 

    server {
        location /images/ {
            root /data;
        }
    
        location / {
            proxy_pass http://www.example.com;
        }
    }

- **root** : static file 이 있는 **파일 시스템의 경로**이다. 이때 root 뒤에 location의 경로가 추가된 상태로 파일의 경로를 찾는다. 예를 들어 /images/example.png의 요청이 들어오면 NGINX는 /data/images/example.png 에서 파일을 찾는다.
- **proxy_pass** : 위의 예시에서 /images/ 와 맞지 않는 모든 패턴의 요청은 프록시 서버로 전송된다. 이후 프록시 서버에서의 응답이 클라이언트에게 전송된다.

## 참고 문서

[https://technerd.tistory.com/19](https://technerd.tistory.com/19)

- 매우 정리가 잘 된 한글 블로그

[https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/#directives](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/#directives)

- 영어 공식 문서

[https://nginx.org/en/docs/beginners_guide.html](https://nginx.org/en/docs/beginners_guide.html)

- 영어 공식 문서