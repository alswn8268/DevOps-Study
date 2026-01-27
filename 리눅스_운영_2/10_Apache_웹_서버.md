# 10.1 웹의 이해

> Client <--Request, Response--> Web Server <-> Web Application Module <-> DataBase
```
sudo yum -y install -y httpd 
sudo systemctl start httpd
sudo systemctl enable httpd

sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload

curl localhost

[vagrant@client ~]$ sudo vi /etc/httpd/conf/httpd.conf
```
# 10.2 웹 서버 구성
## 10.2.1 httpd
### Apache 서버 기능과 관련된 주요 디렉토리 목록
|위치|설명|
|---|---|
|/var/www|웹 페이지 콘텐츠 기본 디렉토리 위치|
|/etc/httpd/conf|웹 서버 주 설정 파일인 httpd.conf 파일 위치|
|/etc/httpd/conf.d|웹 서버의 추가 설정 파일 위치|
|/etc/httpd/conf.modules.d|웹 서버와 함께 설치된 모듈 설정 관련 파일 위치|
|/usr/share/httpd|테스트 페이지, 에러 페이지 등 기본 콘텐츠 위치|
|/usr/share/doc/httpd|웹 서버 관련 문서 파일 위치|



# 10.3 가상 호스트 구성
- 한 웹 서버가 여러 웹 사이트를 서비스할 수 있도록 구성하는 것
- `이름 기반 가상 호스트 (Name-Based Virtual Host)`, `IP 기반 가상 호스트 (IP-Based Virtual Host)`, `포트 기반 가상 호스트 (Port-Based Virtual Host)`가 있음


```
sudo vi /etc/hosts # 서버에서 first.example.com과 second.example.com을 192.168.56.33 서버 IP로 해석하도록 설정
127.0.1.1 user02 user02
192.168.56.33 first.example.com
192.168.56.33 second.example.com

# 가상 호스트 설정
sudo vi /etc/httpd/conf.d/vhosts.conf

<VirtualHost *:80> # 모든 IP에서 80포트 요청을 처리
        ServerName      first.example.com # 요청 도메인 별로 어떤 DocumentRoot를 사용할지 지정
        DocumentRoot    "/var/www/first"

        <Directory "/var/www/first"> # 접근 권한 설정
                AllowOverride   None
                Require all granted
        </Directory>
</VirtualHost>

<VirtualHost *:80>
        ServerName      second.example.com
        DocumentRoot    "/var/www/second"

        <Directory "/var/www/second">
                AllowOverride   None
                Require all granted
        </Directory>
</VirtualHost>
```
```
[vagrant@client ~]$ sudo mkdir /var/www/first
[vagrant@client ~]$ echo "First V Host" | sudo tee /var/www/first/index.html
First V Host

[vagrant@client ~]$ sudo mkdir /var/www/second
[vagrant@client ~]$ echo "Second V Host" | sudo tee /var/www/second/index.html
Second V Host

[vagrant@client ~]$ sudo systemctl reload httpd
```
## 10.3.1 이름 기반 가상 호스트 (Name-Based Virtual Host)
## 10.3.2 IP 기반 가상 호스트 (IP-Based Virtual Host)

```
# enp0s8 인터페이스에 추가 ip 192.168.56.13/24를 설정
[vagrant@user02 ~]$ sudo nmcli connection modify "System enp0s8" +ipv4.addresses 192.168.56.13/24
# 변경사항 적용, 인터페이스 활성화
[vagrant@user02 ~]$ sudo nmcli connection up System\ enp0s8 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)

[vagrant@user02 ~]$ nmcli con show
NAME           UUID                                  TYPE      DEVICE 
enp0s3         53663641-fc96-4c9b-a390-fdc1aa6967cd  ethernet  enp0s3
System enp0s8  00cb8299-feb9-55b6-a378-3fdc720e0bc6  ethernet  enp0s8
lo             2252e989-eeed-4ef0-8c27-87908a91523b  loopback  lo

[vagrant@user02 ~]$ ip a
...
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:94:33:39 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.12/24 brd 192.168.56.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet 192.168.56.13/24 brd 192.168.56.255 scope global secondary noprefixroute enp0s8
       valid_lft forever preferred_lft forever
...

# Apache 가상호스트 설정
[vagrant@user02 ~]$ sudo vi /etc/httpd/conf.d/vhosts.ip.conf
<VirtualHost 192.168.56.12:80>
        ServerName      first.example.com
        DocumentRoot    "/var/www/first"

        <Directory "/var/www/first">
                AllowOverride   None
                Require all granted
        </Directory>
</VirtualHost>

<VirtualHost 192.168.56.13:80>
        ServerName      second.example.com
        DocumentRoot    "/var/www/second"

        <Directory "/var/www/second">
                AllowOverride   None
                Require all granted
        </Directory>
</VirtualHost>

[vagrant@user02 ~]$ sudo systemctl reload httpd
[vagrant@user02 ~]$ curl 192.168.56.12
[vagrant@user02 ~]$ curl 192.168.56.13
```

# 10.4 HTTPS 구성
- SSL (Secure Socket Layer), TLS (Transport Layer Security)
- 암호화, 무결성, 인증, 신뢰도, SEO 향상

```
[vagrant@user02 ~]$ sudo yum install -y mod_ssl
```

## 10.4.1 암호화와 인증서
```
# /etc/pki/tls/private/example.key 경로에 2048비트 RSA 개인 키 생성
[vagrant@user02 ~]$ sudo openssl genrsa -out /etc/pki/tls/private/example.key 2048

[vagrant@user02 ~]$ sudo ls -l /etc/pki/tls/private/
total 4
-rw-------. 1 root root 1704 Nov  7 02:43 example.key

# 개인 키(example.key)를 사용해 CSR 생성
# C=KR : 국가, ST=Seoul : 주/도, L=Seoul : 도시, O=Sesac : 조직, OU=Infra : 조직 단위
# CN=example.com : Common Name (도메인 이름)

[vagrant@user02 ~]$ sudo openssl req -new -key /etc/pki/tls/private/example.key -out /tmp/example.csr -subj "/C=KR/ST=Seoul/L=Seoul/O=Sesac/OU=Infra/CN=example.com"
[vagrant@user02 ~]$ sudo openssl x509 -req -days 365 -in /tmp/example.csr -signkey /etc/pki/tls/private/example.key -out /etc/pki/tls/certs/example.crt
Certificate request self-signature ok
subject=C=KR, ST=Seoul, L=Seoul, O=Sesac, OU=Infra, CN=example.com

[vagrant@user02 ~]$ ls -l /etc/pki/tls/certs/example.crt
-rw-r--r--. 1 root root 1261 Nov  7 03:06 /etc/pki/tls/certs/example.crt

<VirtualHost *:443> # 443 포트에서 HTTPS 요청 수신
        ServerName https.example.com # 접속할 도메인
        DocumentRoot "/var/www/html" # 웹 루트 경로

        SSLEngine on # SSL 활성화
        SSLCertificateFile /etc/pki/tls/certs/example.crt # 서버 인증서 경로
        SSLCertificateKeyFile /etc/pki/tls/private/example.key # 개인 키 경로

        <Directory "/var/www/html"> # 디렉터리 접근 권한 설정
                AllowOverride None 
                Require all granted # 모든 클라이언트 접근 허용
        </Directory>
</VirtualHost>

[vagrant@user02 ~]$ sudo mv /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/ssl.conf.bak
[vagrant@user02 ~]$ sudo systemctl reload httpd
[vagrant@user02 ~]$ sudo systemctl start httpd
[vagrant@user02 ~]$ curl https://localhost
curl: (7) Failed to connect to localhost port 443: Connection refused

[vagrant@user02 ~]$ sudo ss -tlnp | grep httpd
LISTEN 0      511                *:80              *:*    users:(("httpd",pid=33817,fd=4),("httpd",pid=33816,fd=4),("httpd",pid=33815,fd=4),("httpd",pid=33813,fd=4))

[vagrant@user02 ~]$ sudo vi /etc/httpd/conf/httpd.conf
Listen 443 # 리슨할 포트 명시

[vagrant@user02 ~]$ sudo vi /etc/httpd/conf.d/ssl.conf.bak
[vagrant@user02 ~]$ sudo vi /etc/httpd/conf.d/ssl.vhost.conf
```


