# 3.1 DNS(Domain Name System) 소개
# 3.2 DNS 동작 방식
## 3.2.1 DNS 구조
### DNS 계층 구조 및 동작 방식
![alt text](image.png)
- DNS 요청을 중계하는 DNS 서버를 제외한 나머지 서버들은 지정된 계층에만 해당하는 DNS 응답만을 처리
#### 재귀 쿼리 (Recursive Query)
  - 자신이 요청한 DNS 서버에게만 요청을 전송하고, 요청에 대한 응답을 수신하는 과정
  - 일반적으로는 클라이언트가 DNS 서버에게 요청할 때 발생하고, 경우에 따라 DNS 서버가 다시 다른 DNS 서버에게 요청하는 경우에도 사용하는데 이는 요청의 전달(Forwarding)이라고 함
#### 순환 쿼리 (Iterative Query)
  - 클라이언트의 요청을 수신한 DNS 서버는 요청을 처리하기 위하여 다른 DNS 서버로부터 단계적으로 질의하는 과정을 수행함
  - 이 과정을 통해 DNS 서버는 요청을 처리하기 위한 정보를 순차적으로 수집하게 되고, 최종적으로 정확한 IP 주소를 획득할 수 있음

### 동작 방식
  - 순환 쿼리를 처리하기 위해서가장 높은 단계의 요청을 처리하는 DNS 서버를 루트(Root) DNS 서버라고 하는데, 이 서버는 다음 단계의 이름에 대한 답을 가지고 있는 서버의 정보를 가지고 있음
  - 이와 같은 서버 구조에서 중간 단계의 각 DNS 서버들은 다음 단계의 도메인 이름에 대한 요청만 처리하므로 특정 도메인 이름 전체에 대한 정보를 가지고 있을 필요가 없음

### 정방향 조회 / 역방향 조회
### DNS 캐시
  

## 3.2.3 DNS 조회 방법
- DNS 서비스 요청을 사용자가 이름 기반의 통신을 시도하면 자동으로 수행됨
- 만약 사용자가 호스트 이름에 대한 실제 IP 주소를 직접 확인하고 싶다면 `host`, `dig`, `nslookup` 등의 도구가 있음

### host
- 옵션을 통해 DNS 조회와 관련된 가장 많은 기능을 지원함
```
[vagrant@user01 ~]$ host google.com
google.com has address 216.58.220.110
google.com has IPv6 address 2404:6800:4004:822::200e
google.com mail is handled by 10 smtp.google.com.

[vagrant@user01 ~]$ host naver.com
naver.com has address 223.130.200.236
naver.com has address 223.130.192.248
naver.com has address 223.130.192.247
naver.com has address 223.130.200.219
naver.com mail is handled by 20 mx6.mail.naver.com.
naver.com mail is handled by 20 mx4.mail.naver.com.
naver.com mail is handled by 10 mx1.naver.com.
naver.com mail is handled by 10 mx3.naver.com.
naver.com mail is handled by 10 mx2.naver.com.
naver.com mail is handled by 20 mx5.mail.naver.com.
[vagrant@user01 ~]$ host -t MX naver.com
naver.com mail is handled by 10 mx1.naver.com.
naver.com mail is handled by 10 mx2.naver.com.
naver.com mail is handled by 10 mx3.naver.com.
naver.com mail is handled by 20 mx4.mail.naver.com.
naver.com mail is handled by 20 mx5.mail.naver.com.
naver.com mail is handled by 20 mx6.mail.naver.com.
[vagrant@user01 ~]$ host 8.8.8.8
8.8.8.8.in-addr.arpa domain name pointer dns.google.
```

### nslookup
- 기본적으로 대화형으로 실행하나, 명령의 인자로 요청할 이름과 서버를 지정하여 비대화형으로 명령을 실행할 수도 있음
```
[vagrant@user01 ~]$ nslookup google.com
Server:         168.126.63.1
Address:        168.126.63.1#53

Non-authoritative answer:
Name:   google.com
Address: 142.251.42.206
Name:   google.com
Address: 2404:6800:4004:822::200e
```

### dig
- 기본 출력 정보가 상세함
```
[vagrant@user01 ~]$ dig google.com

; <<>> DiG 9.16.23-RH <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31981
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 32f3f09d6c8bdd80010000006908538136d3674d0328bc50 (good)
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             141     IN      A       172.217.175.46

;; Query time: 14 msec
;; SERVER: 168.126.63.1#53(168.126.63.1)
;; WHEN: Mon Nov 03 16:02:25 KST 2025
;; MSG SIZE  rcvd: 83
```

```
sudo vi /etc/resolv.conf
nameserver 10.0.2.3
```

```
[vagrant@user01 ~]$ dig @8.8.8.8 google.com

; <<>> DiG 9.16.23-RH <<>> @8.8.8.8 google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15800
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             20      IN      A       142.250.198.110

;; Query time: 45 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Nov 03 16:25:04 KST 2025
;; MSG SIZE  rcvd: 55
```

# 3.3 DNS 서버 구성
- 개인용 시스템은 잘 알려진 DNS 서버를 주로 사용하지만, 직접 도메인을 호스팅하거나 ,디렉터리 서비스를 사용할 경우 DNS 서버를 자체적으로 구성해야 함. 또는 단위 네트워크에서 DNS 요청 중복을 줄이기 위하여 내부에 캐싱(Caching) DNS 서버를 구성하기도 함
## 3.3.1 BIND 설치 및 구성
- BIND는 리눅스 시스템을 DNS 서버로 사용하기 위해서 가장 많이 사용하는 도구
- 패키지 설치 후에는 `/etc/named.conf` 파일을 수정해야 함
```
sudo yum install bind bind-utils -y # 설치

sudo vi /etc/named.conf
options { # 서버 전체에 적용되는 전역 설정 
        # DNS 서버가 IPv4를 기준으로 53번 포트(기본 DNS 포트)를 모든 네트워크 인터페이스(any)에서 요청을 받겠다는 의미 -> 외부 클라이언트도 이 서버에 DNS 질의를 보낼 수 있음
        listen-on port 53 { any; }; // 127.0.0.1 -> any 
        # IPv6에서의 DNS 요청을 받지 않겠다는 의미
        listen-on-v6 port 53 { none; }; // ::1 -> none
        ...
        # 어떤 클라이언트든지 이 DNS 서버에 질의를 보낼 수 있음 -> 외부에서 접근 가능한 공용 DNS 서버가 됨
        allow-query     { any; }; // localhost -> any
        ...
}

[vagrant@user01 ~]$ sudo systemctl start named

# firewalld에 등록된 dns 서비스를 허용
[vagrant@user01 ~]$ sudo firewall-cmd --add-service=dns --permanent
success
# 방금 변경된 방화벽 설정을 즉시 적용하는 명령
[vagrant@user01 ~]$ sudo firewall-cmd --reload
success


[vagrant@user01 ~]$ dig google.com @192.168.56.11
; <<>> DiG 9.16.23-RH <<>> google.com @192.168.56.11
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 52880
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
...

[vagrant@user01 ~]$ sudo systemctl stop named.service
[vagrant@user01 ~]$ dig google.com @192.168.56.11

; <<>> DiG 9.16.23-RH <<>> google.com @192.168.56.11
;; global options: +cmd
;; connection timed out; no servers could be reached
```
### 3.3.2 영역(Zone) 구성
- 도메인(zone) 파일 작성 및 접근 권한과 구문 점검
- DNS 서버(named)가 사용할 데이터(도메인 이름 <-> IP 주소 매핑)를 `/var/named/data/` 안에 넣고, BIND가 그걸 읽을 수 있도록 권한과 문법을 점검

```
[vagrant@user01 ~]$ sudo mkdir /var/named/data  # var/named/는 BIND가 존 파일을 찾는 기본 디렉터리
[vagrant@user01 ~]$ sudo vi /var/named/data/dns.example.zone # 정방향, 11-04 파일 내용으로 수정
[vagrant@user01 ~]$ sudo vi /var/named/data/db.192.168.56 # 역방향, 11-04 파일 내용으로 수정
[vagrant@user01 ~]$ sudo vi /etc/named.conf # 메인 설정 파일, 11-04 파일 내용으로 수정

[vagrant@user01 ~]$ sudo ls -l /var/named/data/
total 12
-rw-r--r--. 1 root  root   408 Nov  4 11:58 db.192.168.56
-rw-r--r--. 1 root  root   759 Nov  4 11:53 dns.example.zone

[vagrant@user01 ~]$ sudo chown root:named /var/named/data/dns.example.zone # 파일 소유권 named로 변경
[vagrant@user01 ~]$ sudo chown root:named /var/named/data/db.192.168.56 # 파일 소유권 named로 변경

[vagrant@user01 ~]$ sudo ls -l /var/named/data/ 
total 12
-rw-r--r--. 1 root  named  408 Nov  4 11:58 db.192.168.56
-rw-r--r--. 1 root  named  759 Nov  4 11:53 dns.example.zone

[vagrant@user01 ~]$ sudo chmod 640 /var/named/data/dns.example.zone # 권한 변경
[vagrant@user01 ~]$ sudo chmod 640 /var/named/data/db.192.168.56 # 권한 변경
[vagrant@user01 ~]$ sudo ls -l /var/named/data/ 
total 12
-rw-r-----. 1 root  named  408 Nov  4 11:58 db.192.168.56
-rw-r-----. 1 root  named  759 Nov  4 11:53 dns.example.zone

[vagrant@user01 ~]$ sudo named-checkconf /etc/named.conf # 문법 오류 확인
[vagrant@user01 ~]$ sudo named-checkzone dns.example /var/named/data/dns.example.zone
zone dns.example/IN: loaded serial 2024040801
OK
[vagrant@user01 ~]$ sudo named-checkzone 56.168.192.in-addr.arpa /var/named/data/db.192.168.56
zone 56.168.192.in-addr.arpa/IN: loaded serial 2024040801
OK

```
### 도메인 정보를 질의
```
[vagrant@user01 ~]$ host -t MX dns.example localhost
Using domain server:
Name: localhost
Address: 127.0.0.1#53
Aliases:

dns.example mail is handled by 10 mail.dns.example.

[vagrant@user01 ~]$ host -t NS dns.example localhost
Using domain server:
Name: localhost
Address: 127.0.0.1#53
Aliases: 

dns.example name server ns.dns.example.
```
# 도메인 확인
```
[vagrant@user01 ~]$ host 192.168.56.11
Host 11.56.168.192.in-addr.arpa. not found: 3(NXDOMAIN)
[vagrant@user01 ~]$ host 192.168.56.11 localhost
Using domain server:
Name: localhost
Address: 127.0.0.1#53
Aliases:

Host 11.56.168.192.in-addr.arpa. not found: 3(NXDOMAIN)
[vagrant@user01 ~]$ host 192.168.56.11
Host 11.56.168.192.in-addr.arpa. not found: 3(NXDOMAIN)
[vagrant@user01 ~]$ host MX.dns.example localhost
Using domain server:
Name: localhost
Address: 127.0.0.1#53
Aliases:

Host MX.dns.example not found: 3(NXDOMAIN)
[vagrant@user01 ~]$ host -t MX dns.example localhost
Using domain server:
Name: localhost
Address: 127.0.0.1#53
Aliases:

dns.example mail is handled by 10 mail.dns.example.
[vagrant@user01 ~]$ host -t NS dns.example localhost
dns.example name server ns.dns.example.`^C
[vagrant@user01 ~]$ host dns.example localhost
Using domain server:
Name: localhost
Address: 127.0.0.1#53
Aliases:

dns.example has address 192.168.56.11
dns.example mail is handled by 10 mail.dns.example.
```

