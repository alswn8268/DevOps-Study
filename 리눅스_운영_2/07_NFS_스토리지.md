# 7.1 NFS (Network File System) 소개
## 7.1.1 NFS 소개
  - NFS는 공유 네트워크를 사용하는 분산 파일 시스템 프로토콜
  - 로컬 환경에서 사용
  - 디렉터리를 마운트해서 사용
  - `커버로스`와 함께 사용하여 인증, 무결성 검증, 암호화 통신 사용 가능 
  - 현재 버전의 리눅스에서는 `NFSv3`과 `NFSv4`을 지원
## 7.1.2 NFSv3
### 특징
- 안전한 비동기식 쓰기 지원
- 에러 처리 능력 향상
- 64 bit 파일 사이즈 및 오프셋을 지원하여 2GB 이상의 파일에 접근 가능
- `RPC(Remote Procedure Protocol)`에 의존 => 따라서 `rpcbind` 서비스 필요
- NFS 서버의 공유 디렉토리에 대한 잠금 및 마운팅을 지원하기 위해 별도의 서비스 함께 사용
- 방화벽 설정이 복잡
> 동기식: 파일이 디스크에 완전히 저장될 때까지 기다림 -> 안전하지만 느림
> 비동기식: 메모리에 일단 저장하고 나중에 디스크에 저장 -> 빠르지만 정전 시 위험

### 사용
- 최신 버전 리눅스에서는 NFSv4를 기본으로 사용하도록 설정되어 있기 때문에, 설정을 변경하기 위해서는 `/etc/sysconfig/nfs` 파일을 수정함

### 관련된 서비스
- `nfs`: 공유된 NFS 서버에 대한 요청을 처리하기 위하여 NFS 서버와 적절한 RPC 프로세스 수행
- `nfslock`: NFS 서버의 파일을 클라이언트가 잠금할 수 있는 서비스
- `rpcbind`: 로컬 RPC 서비스로부터 포트를 예약함. rpcbind는 RPC 서비스에 대한 요청에 응답하고 요청된 RPC 서비스로 연결을 설정함
- `rpc.mountd`: NFSv3으로부터 마운트 요청을 처리하기 위해 NFS 서버에서 사용됨. 또한 요청된 공유 디렉터리가 현재 NFS 서버에서 export 파일에 등록되었는지 확인함.
  만약 마운트 요청이 허용되면 rpc.mountd는 성공(Success) 상태로 응답하고 클라이언트에게 NFS 공유에 대한 파일-핸들(File-Handle)을 제공함
- `lockd`: 서버와 클라이언트 모두에서 실행되는 커널 쓰레드


## 7.1.3 NFSv4
### 특징
- `rpcbind`를 사용하지 않고 서버 자체가 `2049/TCP` 포트에서 대기
- 마운팅과 잠금 프로토콜을 NFSv4 프로토콜로 통합되어 별개의 서비스를 요구하지 않기 때문에 방화벽 구성이 쉬움
- NFS 서버 성능 향상, 보안 기능 강화
- pNFS(Parallel NFS)를 사용할 수 있음
- 이전 버전의 NFS 서버와 하위 호환되어 동작

### 관련된 서비스
- `rpcbind`의 제어를 받지 않기 때문에 서버나 클라이언트에서 동작하는 서비스가 많지 않음

# 7.2 NFS 서버 연결
- 서버 로그인
```
Last login: Tue Nov  4 07:13:35 2025 from 10.0.2.2
[vagrant@server ~]$ sudo su - # root로 변경
[root@server ~]# ip a | grep 192.168 # ip 확인
    inet 192.168.56.44/24 brd 192.168.56.255 scope global noprefixroute enp0s8
```
- 공유 폴더 생성 후 파일 생성
```
[root@server ~]# yum install -y nfs-utils
[root@server ~]# mkdir /srv/share
[root@server ~]# echo "NFS share Test file" > /srv/share/test.txt
[root@server ~]# cat /srv/share/test.txt 
NFS share Test file
```
- 
```
[root@server ~]# vi /etc/exports # 현재 상태 확인
/srv/share      *(rw,sync,no_root_squash,insecure) # 모든 클라이언트가 /srv/share 폴더에 읽기, 쓰기, 접근 가능
[root@server ~]# exportfs -r # 설정 재적용
[root@server ~]# exportfs # 공유 디렉터리 및 옵션 확인
/srv/share      <world>

# start -> 설정 파일 수정 -> reload
# 패키지 설정 -> 설정 파일 수정 -> start

[root@server ~]# systemctl start nfs-server # nfs 서비스 실행
[root@server ~]# systemctl enable nfs-server.service # 부팅시 자동 시작
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.

[vagrant@server ~]$ ss -tuln # 포트 2049 열람 확인
...
tcp     LISTEN   0        64               0.0.0.0:2049           0.0.0.0:*
...

[vagrant@server ~]$ rpcinfo -p localhost # rpc 프로그램 포트 확인
[root@server ~]# firewall-cmd --add-service=nfs --permanent
success
[root@server ~]# firewall-cmd --reload
success
[root@server ~]# firewall-cmd --list-services
cockpit dhcpv6-client nfs ssh

[root@server ~]# firewall-cmd --add-service=rpc-bind
success
[root@server ~]# firewall-cmd --add-service=mountd
success
[root@server ~]# firewall-cmd --reload

[root@server ~]# systemctl is-active nfs-server
active
[root@server ~]# systemctl is-enabled nfs-server.service
enabled
[root@server ~]# exportfs -v
/srv/share      <world>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,insecure,no_root_squash,no_all_squash)
[root@server ~]# ls -l /srv/share/
total 4
-rw-r--r--. 1 root root 20 Nov  5 01:36 test.txt
```

# 7.3 NFS 클라이언트 연결
## 7.3.1 수동 마운트
```
[root@client ~]# yum install -y nfs-utils # NFS 클라이언트/서버 관련 명령어를 포함한 필수 패키지 설치
[root@client ~]# systemctl start nfs-utils.service  # 활성화

[root@client ~]# showmount -e 192.168.56.44 # 서버의 공유 목록 확인
Export list for 192.168.56.44:
/srv/share *
[root@client ~]# mkdir /mnt/nfs
[root@client ~]# mount -t nfs 192.168.56.44:/srv/share /mnt/nfs # nfs 마운트

[root@client ~]# mount | grep mnt # 마운트 확인
192.168.56.44:/srv/share on /mnt/nfs type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.56.33,local_lock=none,addr=192.168.56.44)

[root@client ~]# ls -l /mnt/nfs # 파일 접근 테스트
total 4
-rw-r--r--. 1 root root 20 Nov  5 01:36 test.txt
[root@client ~]# cat /mnt/nfs/test.txt # 파일을 읽을 수 있음
NFS share Test file
```

### 공유 폴더 정상 작동 확인
```
[root@client ~]# echo "${HOSTNAME} Test file" > /mnt/nfs/client.txt # 클라이언트에서 파일 생성
[root@client ~]# cat /mnt/nfs/client.txt
client Test file

[root@server ~]# ls -l /srv/share # 서버에 동일 파일 있는지 확인
total 8
-rw-r--r--. 1 root root 17 Nov  5 02:42 client.txt
-rw-r--r--. 1 root root 20 Nov  5 01:36 test.txt
[root@server ~]# cat /srv/share/client.txt
client Test file
```

### /etc/fstab 파일에 등록
```
[root@client ~]# umount /mnt/nfs
[root@client ~]# vi /etc/fstab 
# 클라이언트가 부팅할 때 자동으로 NFS 서버(192.168.56.44)의 /srv/share를 /mnt/nfs로 마운트하도록 설정
192.168.56.44:/srv/share        /mnt/nfs        nfs     defaults        0 0   

[root@client ~]# mount -a # /etc/fstab 에 있는 모든 항목을 다시 마운트
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
[root@client ~]# mount | grep /mnt
192.168.56.44:/srv/share on /mnt/nfs type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.56.33,local_lock=none,addr=192.168.56.44)
```
> 부팅 시 등록한 정보로 마운트가 되는 방법이지, 자동 마운트가 아님

## 7.3.2 자동 마운트
- NFS 공유 디렉토리를 자동으로 마운트하거나 마운트 해제하는 서비스
- 수동 마운트는 지속적으로 마운트 되어 있는 상태이기 때문에 시스템 자원을 낭비하게 됨. 자동 마운트를 사용하면 클라이언트에서 설정한 정보대로 시스템 자원을 효율적으로 사용할 수 있음
> 마스터 맵, 직접 맵, 간접 맵 파일을 생성해야 함
```
[root@client ~]# yum install -y autofs
[root@client ~]# umount /mnt/nfs
[root@client ~]# vi /etc/fstab # 위에 작성한 라인 주석처리

```

### 맵파일
#### 마스터 맵 파일
  - 마운트 포인트와 맵 파일 경로 지정
  ```
  - `/etc/auto.master`파일에 기본적으로 존재
  [root@client ~]# grep -v '^#' /etc/auto.master
  /misc   /etc/auto.misc
  /net    -hosts
  +dir:/etc/auto.master.d
  +auto.master

  [root@client ~]# vi /etc/auto.master.d/nfs.autofs # 자동으로 파일 시스템을 마운트/언마운트하는 서비스
  /- /etc/auto.direct # 절대 경로를 직접 매핑하겠다는 의미 / 실제 마운트 규칙이 적혀 있는 파일의 경로

  /mnt/nfs -rw,sync 192.168.56.44:/srv/share # 클라이언트에서 접근할 마운트 지점 / 읽기 및 쓰기 가능, 동기화 / NFS 서버 IP와 실제 공유 경로
  # 192.168.56.44 서버의 /srv/share 디렉토리를 /mnt/nfs에 자동 마운트하라는 의미

  [root@client ~]# systemctl start autofs
  [root@client ~]# mount | grep /mnt 
  /etc/auto.direct on /mnt/nfs type autofs (rw,relatime,fd=13,pgrp=32764,timeout=300,minproto=5,maxproto=5,direct,pipe_ino=63349)

  [root@client ~]# ls -l /mnt/nfs/
  total 8
  -rw-r--r--. 1 root root 17 Nov  5 02:42 client.txt
  -rw-r--r--. 1 root root 20 Nov  5 01:36 test.txt
  ```

  ```
  # /mnt/nfs는 실제 nfs가 직접 마운트된 게 아니라 autofs가 감시하고 있는 자동 마운트 포인트이기 때문에 umount를 해도 autofs 자체는 언마운트되지 않음
  [root@client ~]# umount /mnt/nfs  
  [root@client ~]# df -h | grep nfs
  [root@client ~]# mount | grep /mnt
  /etc/auto.direct on /mnt/nfs type autofs (rw,relatime,fd=13,pgrp=32764,timeout=300,minproto=5,maxproto=5,direct,pipe_ino=63349)
  [root@client ~]# df -h | grep nfs
  ```

#### 직접 맵 파일
  - 절대 경로로 마운트 포인트 지정

#### 간접 맵 파일
  - 상대 경로로 마운트 포인트 지정

```
[root@server ~]# mkdir -p /srv/share2
[root@server ~]# echo "NFS SERVER share2" > /srv/share2/test2.txt

[root@server ~]# cat /etc/exports 
/srv/share      *(rw,sync,no_root_squash,insecure)
[root@server ~]# vi /etc/exports

/srv/share      *(rw,async,no_root_squash,insecure)
/srv/share2     *(rw,async,no_root_squash,insecure)

[root@server ~]# exportfs -r
[root@server ~]# exportfs -v
/srv/share      <world>(async,wdelay,hide,no_subtree_check,sec=sys,rw,insecure,no_root_squash,no_all_squash)
/srv/share2     <world>(async,wdelay,hide,no_subtree_check,sec=sys,rw,insecure,no_root_squash,no_all_squash)

[root@client ~]# vi /etc/auto.master.d/indirect.autofs
# /mnt/auto 경로 아래에서 auto.indirect 파일의 규칙을 따라 자동 마운트 하라는 의미
/mnt/auto       /etc/auto.indirect

[root@client ~]# vi /etc/auto.indirect # /mnt/auto/share에 접근하면 /src/share가 자동 마운트 됨
share1  -rw,async       192.168.56.44:/srv/share
share2  -rw,async       192.168.56.44:/srv/share2

[root@client ~]# systemctl restart autofs
[root@client ~]# ls -l /mnt/auto/
total 0
[root@client ~]# cd /mnt/auto/share1
[root@client share1]# ls -l
total 8
-rw-r--r--. 1 root root 17 Nov  5 02:42 client.txt
-rw-r--r--. 1 root root 20 Nov  5 01:36 test.txt

[root@client share1]# cd /mnt/auto/share2
[root@client share2]# ls -l
total 4
-rw-r--r--. 1 root root 18 Nov  5 04:48 test2.txt
[root@client share2]# cat test2.txt
NFS SERVER share2

```
# 타임아웃
```
[root@client ~]# grep ^timeout /etc/autofs.conf
timeout = 300

[root@client ~]# vi /etc/autofs.conf
timeout = 10 #으로 변경
root@client ~]# grep ^timeout /etc/autofs.conf
timeout = 10

[root@client ~]# systemctl restart autofs.service

[root@client ~]# df -h | grep mnt
[root@client ~]# cd /mnt/auto/share2
[root@client share2]# df -h | grep mnt
192.168.56.44:/srv/share2   49G  1.6G   47G   4% /mnt/auto/share2
[root@client share2]# sleep 11
[root@client share2]# df -h | grep mnt
192.168.56.44:/srv/share2   49G  1.6G   47G   4% /mnt/auto/share2

[root@client share2]# cd ~
[root@client ~]# sleep 11
[root@client ~]# df -h | grep mnt

```
