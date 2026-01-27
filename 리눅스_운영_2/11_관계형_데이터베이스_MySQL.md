# 11.1 관계형 데이터베이스

```
[vagrant@user02 ~]$ sudo yum install -y mysql-server
[vagrant@user02 ~]$ sudo systemctl start mysqld.service


sudo mysql -p
qwer

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)

mysql> CREATE DATABASE company;
Query OK, 1 row affected (0.01 sec)

# 서버에 있는 데이터베이스(스키마) 목록을 보여줌. 기본적으로 MySQL 자체가 관리하는 DB들
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| company            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> USE company # company라는 새 데이터베이스를 생성
Database changed
mysql> SHOW TABLES; # company DB에 있는 테이블을 보여줌
Empty set (0.01 sec)

# employee라는 테이블 생성
mysql> CREATE TABLE employee (id INT, name VARCHAR(50), email VARCHAR(50));
Query OK, 0 rows affected (0.02 sec)

# 행 삽입
mysql> INSERT INTO employee (id, name, email) VALUES  (1, '두부', 'asd@asd.com');
mysql> INSERT INTO employee (id, name, email) VALUES  (2, '세부', 'a3d@asd.com'), (3, '두콩이', 'aSd@nasd.com');

SELECT * FROM employee;

mysql> DELETE FROM employee WHERE ID = 3;


mysql> create table mysql ( 
    -> id INT AUTO_INCREMENT PRIMARY KEY,
    -> name VARCHAR(100),
    -> age INT
    -> );

mysql> insert into mysql(name,age) values ('피카츄', 20), ('파이리', 22), ('망나뇽',100);
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> select * from mysql

mysql> delete from mysql where id = 1;
Query OK, 1 row affected (0.00 sec)
``` 


```
[vagrant@user02 ~]$ sudo yum install -y mysql-server
Last metadata expiration check: 1:38:01 ago on Fri 07 Nov 2025 05:35:20 AM UTC.
Package mysql-server-8.0.43-1.el9_6.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
[vagrant@user02 ~]$ sudo yum install -y httpd php php-mysqlnd
Last metadata expiration check: 1:38:17 ago on Fri 07 Nov 2025 05:35:20 AM UTC.
Package httpd-2.4.62-4.el9_6.4.x86_64 is already installed.
Dependencies resolved.
====================================

# 워드프레스 6.8.2 공식 압축을 다운로드
[vagrant@user02 ~]$ sudo curl -o wordpress.tar.gz https://wordpress.org/wordpress-6.8.2.tar.gz
# /var/www/html/wordpress 경로에 압축을 풂
[vagrant@user02 ~]$ sudo tar xzf wordpress.tar.gz -C /var/www/html/

[vagrant@user02 ~]$ sudo systemctl start mysqld
[vagrant@user02 ~]$ sudo mysql_secure_installation y y n y

# root로 로그인 후
sudo mysql -u root -h localhost -p

CREATE DATABASE wp;
CREATE USER 'wpus'@'localhost' IDENTIFIED BY 'Qwer1234!';
GRANT ALL PRIVILEGES ON wp.* TO 'wpus'@'localhost';

```

```
# 워드프레스에서 제공하는 기본 설정 템플릿 파일을 실제 설정 파일로 복사한 것
[vagrant@user02 ~]$ sudo cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php

# wp-config.php 안에서 DB_NAME, DB_USER, DB_PASSWORD, DB_HOST를 자신의 DB 정보로 수정해야 함
[vagrant@user02 ~]$ sudo vi /var/www/html/wordpress/wp-config.php

# Apache 웹서버 설정 블록. 어떤 주소로 어떤 폴더를 웹사이트로 보여줄지를 지정하는 가상 호스트(Virtual Host) 설정
sudo tee /etc/httpd/conf.d/wordpress.conf
<VirtualHost *:80> # 포트 80으로 들어오는 모든 요청에 대해 아래 설정 적용
        # 이 사이트의 도메인 또는 IP 주소 지정. (즉, 192.168.56.11로 접속하면 /var/www/html/wordpress 폴더 내용을 보여줌)
        ServerName  192.168.56.11 
        DocumentRoot  "/var/www/html/wordpress" 
        # 위에서 지정한 폴더에 대해 추가 접근 규칙을 지정함. Apache는 보안상 기본적으로 디렉토리 접근을 제한하기 때문에, "이 폴더는 웹에서 접근해도 괜찮다"고 명시해야 함
        <Directory "/var/www/html/wordpress"> 
                AllowOverride None # .htaccess 파일을 지정할 수 있는지 지정
                Require all granted # 이 디렉터리의 파일을 모든 사용자에게 공개적으로 접근 허용
        </Directory>
</VirtualHost>   

# 80번 포트 일시 허용
[vagrant@user02 ~]$ sudo firewall-cmd --add-service=http
success
```

