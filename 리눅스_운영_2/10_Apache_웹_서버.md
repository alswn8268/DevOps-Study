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

