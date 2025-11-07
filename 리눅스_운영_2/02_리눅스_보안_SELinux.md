# 2.1 SELinux 소개
## 2.1.1 접근 제어 모델
- 리눅스는 접근 제어 모델을 사용해서 파일이나 자원에 대한 접근을 제어함.
- `DAC`, `MAC`, `RBAC`로 세 가지 모델 존재
> 주체 (Subject): 시스템의 자원에 접근하는 프로세스 또는 사용자를 의미
> 객체 (Object): 파일 또는 포트와 같은 시스템의 자원을 의미
### DAC (Discretionary Access Control) 모델
  - 사용자가 임의로 객체에게 권한을 부여하여 객체에 대한 접근을 제어함
  - 주체는 객체에 대한 권한만 설정되어 있다면 접근할 수 있음
### MAC (Mandatory Access Control) 모델
  - 주체와 객체에 부여된 보안 레이블과 정책 허용 스위치에 의해 접근을 제어함
  - 주체가 객체에 접근할 때 먼저 객체에 접근할 수 있는 권한이 있는지 확인한 후 보안 레이블을 확인함. 이 때 주체의 보안 레이블이 객체에 접근할 수 있도록 지정된 보안 레이블일 경우 접근을 허용하지만, 그렇지 않을 경우는 접근 거부
  - 스위치는 On/Off로 설정할 수 있음
  - 구성이 어려워서 일반적인 시스템보다는 군부대나 비밀 유지가 각별한 곳에서 사용되는 경우가 많음
### RBAC (Role Based Access Control) 모델
  - 관리자가 역할(다수의 권한을 묶어놓은 그룹)을 사용자에게 부여함으로써 권한(Right)을 제어

## 2.1.2 SELinux 동작 원리
- 모든 것을 기본적으로 차단
- 컨텍스트 기반 접근 제어 (MAC 모델)
- **최소 권한 원칙**
## 2.1.3 semanage 도구
- SELinux의 보안 정책을 조회하거나 설정할 수 있는 관리 도구

# 2.2 SELinux 모드
## 2.2.1 SELinux 모드 종류
### Disabled 모드
  - SELinux 커널 모듈을 메모리에 로드하지 않기 때문에 완전 `비활성화` 되어있음
  - 시스템은 `DAC` 모델을 사용하며, 프로세스가 파일에 접근할 때 파일에 부여된 권한을 기준으로 접근 제어
### Enforcing 모드
  - SELinux 커널 모듈을 메모리에 로드
  - SELinux가 `활성화`되어 있고, SELinux 정책을 강제함
  - `MAC` 모델이 적용되며, 프로세스가 파일에 접근할 때 권한과 컨텐스트, 부울 등을 확인하고 필요시 차단
### Permissive 모드
  - SELinux 커널 모듈을 메모리에 로드
  - SELinux가 `활성화`되어 있지만 정책을 강제하지는 않음
  - 대신 정책을 위반했을 때 경고 메시지를 남김

> Disabled 모드와 Permissive 모드의 차이
> Disabled 모드는 SELinux 커널 모듈을 메모리에 로드하지 않았기 때문에 Enforcing <-> Disabled 전환할 경우에는 반드시 시스템 재부팅이 필요함. 반면 Enforcing <-> Permissive 전환할 때는 재부팅하지 않고 전환할 수 있음

## 2.2.2 SELinux 모드 설정
### getenforce
```
[vagrant@user01 ~]$ getenforce # SELinux 모드 확인
Enforcing # 디폴트

[vagrant@user01 ~]$ sudo setenforce 0 # SELinux 모드 변경
[vagrant@user01 ~]$ getenforce
Permissive
```

## 2.2.2 SELinux 설정 파일
- SELinux 설정 파일은 `/etc/selinux/config`
- 이 파일은 SELinux 모드를 영구적으로 저장하거나 SELinux의 정책 유형을 지정할 수 있음
- 단, 커널 레벨 동작으로 인해 후속 변경이 불가능하기 때문에 반드시 재부팅 필요
```
sudo vi /etc/selinux/config # 에서
SELINUX=enforcing # 부분을 변경하면 디폴트 상태가 변경됨
```

### SELINUX 속성
- SELinux 모드를 영구적으로 설정하는 속성
### SELINUXTYPE 속성
  - 대상 정책 (Targeted Policy)
  : 공격자가 공격할 가능성이 높은 프로세스의 접근을 제어하는 정책. 대부분의 프로세스는 DAC 모델과 같이 동작하고, 공격 대상이 될 가능성이 높은 프로세스는 SELinux의 자체 도메인 내에서 실행되어 파일에 대한 접근이 제어됨
  - 최소 정책 (Minimum Policy)
  : 최소한의 프로세스만 접근을 제어하며, 가상머신 또는 휴대용 기기 등에서 SELinux를 적용할 때 사용하면 시스템 자원을 효율적으로 사용할 수 있음
  - MLS (Multi Level Security Policy) 정책
  : 사용자 접근에 대해서 서로 다른 규칙(Rule)을 가지고 있는 다양한 레벨을 사용하여 프로세스의 접근을 제어하는 정책. 최상위 비밀(Top Secret), 비밀(Secret), 기밀(Confident), 미분류(Unclassified)로 분류됨

# 2.3 SELinux 컨텍스트 (SELinux Context) ★
## 2.3.1 컨텍스트 소개
  - 시스템에 SELinux 기능이 활성화되면 MAC 모델이 적용되기 때문에 시스템에 존재하는 모든 프로세스와 파일에 컨텍스트가 부여됨
  - 그리고 이 컨텍스트는 프로세스가 파일에 접근할 때 비교하는 요소로 사용됨
  - `ps` 또는 `ls` 명령에 `-Z` 옵션을 사용해서 확인
```
[vagrant@user01 ~]$ ls -lZ /home
total 0
drwx------. 2 guest123  guest123 unconfined_u:object_r:user_home_dir_t:s0  62 Oct 31 20:36 guest123
...
```
```
[vagrant@user01 ~]$ ps -eZ
LABEL                               PID TTY          TIME CMD
system_u:system_r:init_t:s0        3448 ?        00:00:00 (sd-pam)
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 3461 ? 00:00:00 sshd
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 3462 pts/0 00:00:00 bash
```
> 사용자 : 역할 : 유형 : 레벨

  - `사용자(SELinux User)`
  : 시스템에 존재하는 리눅스 사용자가 아니라 SELinux 사용자를 의미함
  SELinux 사용자는 SELinux 정책에 의해 각 리눅스 사용자에게 연결되어 사용됨
  연결된 SELinux 사용자는 역할, 레벨 등과 함께 자원이나 파일에 대한 접근을 제어함
  - `역할(Role)`
  : RBAC 모델의 특징 중 하나로, 역할에는 도메인에 대한 권한이 부여되고, SELinux 사용자는 이 역할을 부여받음
  => 역할은 SELinux 사용자와 도메인을  연결하는 기능이며, 궁극적으로 접근할 수 있는 오브젝트 유형을 결정함
  - `유형(Type)` ★
  : SELinux에서 접근을 제어하는 메커니즘인 TE(Type Enforcement)의 속성
  주체가 객체에 접근하려고 할 때, 컨텍스트를 비교하기 위해 사용 됨
  - `레벨(Level)`
  : MLS(Multi Level Security) 와 MCS(Multi Category Security)를 나타냄
  MLS 레벨은 `저레벨-고레벨`로 표현되며, 동일한 레벨이면 하나로 표현하는데 이를 `민감도`라고 함
  레벨은 `민감도`와 `카테고리`로 표현될 수 있는데, 카테고리가 없을 경우 기존처럼 `저레벨-고레벨`로 표현되며, 카테고리가 존재할 경우 `민감도:카테고리`로 표현됨
  컨텍스트의 레벨 부분은 MLS 정책을 사용하여 보안성을 더욱 강화할 때 사용됨

## 2.3.2 컨텍스트 변경
- SELinux가 활성화되어 있는 시스템에서 파일을 생성하면 시스템에 등록된 보안 레이블 정책에 의해 컨텍스트가 부여되는데, 이때 대부분 상위 디렉토리의 컨텍스트가 부여됨.
- `mv` 또는 `cp -a` 명령을 사용하여 파일을 이동하거나 복사하면 컨텍스트는 변경되지 않고 기존의 것을 유지함
```
[vagrant@user01 ~]$ ls -dZ /var/www/html/
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/

# undefined_u: 사용자가 만듦
# system_u: SELinux가 정한 보안 사용자 카테고리
# httpd_sys_content_t: Apache 웹서버_시스템 레벨에서 관리_웹 콘텐츠_타입
```
### 컨텍스트 상속
```
[vagrant@user01 ~]a$ ls -Z /bin/passwd
system_u:object_r:passwd_exec_t:s0 /bin/passwd
[vagrant@user01 ~]$ sudo cp /bin/passwd /var/www/html/fileB
[vagrant@user01 ~]$ ls -Z /bin/passwd
system_u:object_r:passwd_exec_t:s0 /bin/passwd
[vagrant@user01 ~]$ ls -Z /var/www/html/fileB
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/fileB
[vagrant@user01 ~]$ sudo cp -a /bin/passwd /var/www/html/fileC
[vagrant@user01 ~]$ ls -Z /var/www/html/fileC
system_u:object_r:passwd_exec_t:s0 /var/www/html/fileC
```
### chcon 명령
- 파일의 컨텍스트를 일시적으로 변경하는 명령
```
[vagrant@user01 ~]$ echo "test file" > ~/test_selinux.txt
[vagrant@user01 ~]$ ls -Z test_selinux.txt
unconfined_u:object_r:user_home_t:s0 test_selinux.txt
[vagrant@user01 ~]$ chcon -t httpd_sys_content_t test_selinux.txt
[vagrant@user01 ~]$ ls -Z test_selinux.txt
unconfined_u:object_r:httpd_sys_content_t:s0 test_selinux.txt
```

```
[vagrant@user01 ~]$ sudo firewall-cmd --add-service=http --permanent
Warning: ALREADY_ENABLED: http
success
[vagrant@user01 ~]$ sudo firewall-cmd --add-service=httpd --permanent
Error: INVALID_SERVICE: Zone 'public': 'httpd' not among existing services
[vagrant@user01 ~]$ sudo firewall-cmd --reload
success
[vagrant@user01 ~]$ sudo firewall-cmd --list-services
cockpit dhcpv6-client http ssh
[vagrant@user01 ~]$ echo "SELinux Test" | sudo tee /var/www/html/index.html
SELinux Test

[vagrant@user01 ~]$ ls -Z /var/www/html/
unconfined_u:object_r:httpd_sys_content_t:s0 fileA            system_u:object_r:passwd_exec_t:s0 fileC
unconfined_u:object_r:httpd_sys_content_t:s0 fileB  unconfined_u:object_r:httpd_sys_content_t:s0 index.html
[vagrant@user01 ~]$ mkdir /tmp/webtest
[vagrant@user01 ~]$ echo "custom" > /tmp/webtest/custom.html
[vagrant@user01 ~]$ sudo cp /tmp/webtest/custom.html /var/www/html/
[vagrant@user01 ~]$ ls -Z /tmp/webtest/custom.html 
unconfined_u:object_r:user_tmp_t:s0 /tmp/webtest/custom.html
[vagrant@user01 ~]$ sudo chcon -t user_tmp_t /var/www/html/custom.html
[vagrant@user01 ~]$ ls -Z /var/www/html/custom.html
unconfined_u:object_r:user_tmp_t:s0 /var/www/html/custom.html
[vagrant@user01 ~]$ curl localhost/custom.html
custom

```

# 2.4 SELinux 부울 (SELinux Boolean)
## 2.4.1 부울의 개념
- SELinux에서 시스템이 운영중일 때 정책의 동작을 변경할 수 있는 스위치와 같은 기능
- `getsebool`, `setsebool`, `semanage boolean`이 있음

## 2.4.2 부울의 확인
### getsebool
```
[vagrant@user01 ~]$ getsebool -a 
abrt_anon_write --> off
abrt_handle_event --> off
abrt_upload_watch_anon_write --> on

[vagrant@user01 ~]$ getsebool httpd_can_network_connect
httpd_can_network_connect --> off
...
```

### semanage
```
[vagrant@user01 ~]$ sudo semanage boolean -l
SELinux boolean                State  Default Description

abrt_anon_write                (off  ,  off)  Allow ABRT to modify public files used for public file transfer services.
abrt_handle_event              (off  ,  off)  Determine whether ABRT can run in the abrt_handle_event_t domain to handle ABRT event scripts.
abrt_upload_watch_anon_write   (on   ,   on)  Determine whether abrt-handle-upload can modify public files used for public file transfer services in /var/spool/abrt-upload/.
antivirus_can_scan_system      (off  ,  off)  Allow antivirus programs to read non security files on a system
...
```

## 2.4.3 부울의 설정
```
[vagrant@user01 ~]$ sudo setsebool httpd_can_network_connect on 
httpd_can_network_connect      (on   ,   off)  Allow HTTPD scripts and modules to connect to the network using TCP.

[vagrant@user01 ~]$ sudo setsebool httpd_can_network_connect on # -P 옵션을 추가하면 영구 설정
httpd_can_network_connect      (on   ,   on)  Allow HTTPD scripts and modules to connect to the network using TCP.
```
```
[vagrant@user01 ~]$ sudo semanage boolean -m --on use_nfs_home_dirs
[vagrant@user01 ~]$ sudo semanage boolean -l | grep use_nfs_home_dirs
use_nfs_home_dirs              (on   ,   on)  Support NFS home directories

```

# 2.5 SELinux 포트 레이블 (SELinux Port Label)
## 2.5.1 포트 레이블 개념
- SELinux가 활성화되면 서비스들이 사용하는 포트에도 레이블이 부여됨
  => 따라서 서비스가 기본적으로 사용하는 포트 외의 것을 사용하려면 해당 포트에 포트 레이블을 부여해야 함

## 2.5.2 포트 레이블 확인 및 설정
```
[vagrant@user01 ~]$ sudo semanage port -l | grep http # http 포트 넘버 확인
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000

[vagrant@user01 ~]$ sudo semanage port -a -t http_port_t -p tcp 3333 # 포트 레이블 설정
[vagrant@user01 ~]$ sudo semanage port -l | grep http
http_port_t                    tcp      3333, 80, 81, 443, 488, 8008, 8009, 8443, 9000
```

# 2.6 SELinux 문제 해결
## 2.6.1 문제 해결 순서
1) Permissive 모드 전환
2) 파일의 보안 레이블 확인
3) 포트 레이블 확인
4) 부울 확인

## 2.6.2 문제 해결 방법
### 로그 확인
- `/var/log/audit/audit.log`에 기록되어 있는 거부 메시지 확인

```
[vagrant@user01 ~]$ grep 'denied' /var/log/audit/audit.log
grep: /var/log/audit/audit.log: Permission denied
[vagrant@user01 ~]$ sudo grep 'denied' /var/log/audit/audit.log
type=AVC msg=audit(1761531989.987:759): avc:  denied  { open } for  pid=3963 comm="20-chrony-dhcp" path="/etc/sysconfig/network-scripts/ifcfg-enp0s8" dev="sda3" ino=34560071 scontext=system_u:system_r:NetworkManager_dispatcher_chronyc_t:s0 tcontext=unconfined_u:object_r:user_tmp_t:s0 tclass=file permissive=0
type=AVC msg=audit(1761532307.431:54): avc:  denied  { open } for  pid=942 comm="20-chrony-dhcp" path="/etc/sysconfig/network-scripts/ifcfg-enp0s8" dev="sda3" ino=34560071 scontext=system_u:system_r:NetworkManager_dispatcher_chronyc_t:s0 tcontext=unconfined_u:object_r:user_tmp_t:s0 tclass=file permissive=0
...

[vagrant@user01 ~]$ sudo ausearch -m avc --start recent
<no matches>

[vagrant@user01 ~]$ sudo sealert -a /var/log/audit/audit.log
100% done
found 2 alerts in /var/log/audit/audit.log
--------------------------------------------------------------------------------

SELinux is preventing /usr/bin/bash from open access on the file /etc/sysconfig/network-scripts/ifcfg-enp0s8.
...
```


