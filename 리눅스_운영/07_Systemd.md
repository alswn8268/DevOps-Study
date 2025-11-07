# 7.1 systemd 소개
## 7.1.1 systemd
- 가장 먼저 실행되는 데몬(백그라운드에서 실행되는 프로세스), PID 1번을 가짐
- 컴퓨터를 켰을 때 필요한 모든 프로그램과 서비스를 지원

```
[vagrant@user01 ~]$ ps -ef | grep systemd
root           1       0  0 02:31 ?        00:00:01 /usr/lib/systemd/systemd rhgb --switched-root --system --deserialize 31
root         484       1  0 02:31 ?        00:00:00 /usr/lib/systemd/systemd-journald
root         498       1  0 02:31 ?        00:00:00 /usr/lib/systemd/systemd-udevd
root         610       1  0 02:31 ?        00:00:00 /usr/lib/systemd/systemd-logind
vagrant     3285       1  0 02:33 ?        00:00:00 /usr/lib/systemd/systemd --user
vagrant     3800    3301  0 06:43 pts/0    00:00:00 grep --color=auto systemd
```
## 7.1.2 systemd 기능 및 특징
- init 프로세스에 대한 호환성 제공
- systemd 유닛 사용
- 시스템 부팅 시 서비스 병렬 시작
- 사용자 요구에 맞게 (On-demand) 서비스 실행
- 의존성 기반의 서비스 제어 로직 제공
- CGroup (Control Group) 관리
- 소켓 기반 활성화 (Socket-based activation)
  -> 지연 시작 (lazy loading)
- 통합 로그 관리
  -> 중앙에서 모두 관리

# 7.2 systemd 유닛
- systemd가 관리하는 모든 것을 나타내는 기본 단위
## 7.2.1 systemd 유닛 파일 위치
- systemd 유닛의 파일은 `/etc/systemd/system`, `run/systemd/system`, `/usr/lib/systemd/system`에 저장됨
### /etc/systemd/sytem
  - 시스템 관리자가 수동으로 생성 및 관리하는 유닛들이 저장되는 디렉터리
### /run/systemd/system
  - 시스템이 runtime 상태일 때 임시로 유닛 파일을 저장하는 디렉터리
  - 시스템이 재부팅되면 모두 삭제됨
### /usr/lib/systemd/system
  - 사용자가 특정 유닛이 포함된 패키지를 설치하면 저장되는 디렉터리
  - 유닛 파일의 원본
  - 사용자가 서비스를 활성화(enable)시키면 이 디렉터리에 존재하는 파일이 /etc/systemd/system 디렉터리에 링크됨

## 7.2.2 systemd 유닛 파일의 구성
- systemd 유닛 파일은 `Unit`, `유닛의 유형(Path)`, `Install`
```
[vagrant@user01 ~]$ cat /usr/lib/systemd/system/sshd.service
[Unit] # 서비스의 설명과 실행 순서
Description=OpenSSH server daemon # 설명
Documentation=man:sshd(8) man:sshd_config(5) # 매뉴얼 위치
After=network.target sshd-keygen.target # 지정된 유닛들이 실행된 뒤에 시작되어야 함
Wants=sshd-keygen.target # 같이 실행되길 원함

[Service] # 실제 프로세스 실행 방법
Type=notify # 서비스 시작 방식
EnvironmentFile=-/etc/sysconfig/sshd # 해당 경로의 환경 변수를 불러옴
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process # 서비스 중지 시 메인 프로세스만 죽임
Restart=on-failure 서비스가 실패했을 때만 자동 재시작
RestartSec=42s # 실패시 42초 대기

[Install] # 부팅 시 자동 실행 설정 정보
WantedBy=multi-user.target # 어느 시점에 자동 실행될지 지정
```
### Unit
- 유닛 자체의 일반적인 정보를 담고 있음
- 유닛에 대한 설명, 유닛의 동작, 다른 유닛과의 의존성 등
### 유닛의 유형
- 유닛의 유형에 대한 정보를 저장
- Service, Socket 등과 같이 유닛의 이름이 지정됨
### Install
- systemctl 서브커맨드인 enable과 disable 명령과 연관됨

# 7.3 systemctl
- systemd 유닛을 관리하는 명령
- 시스템 시작, 중지, 상태 확인, 재시작 등
> start, stop, enable
```
[vagrant@user01 ~]$ sudo yum install -y httpd
[vagrant@user01 ~]$ sudo systemctl status httpd
○ httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:httpd.service(8)
[vagrant@user01 ~]$ sudo systemctl start httpd
[vagrant@user01 ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: active (running) since Tue 2025-10-28 01:45:41 UTC; 7s ago
       Docs: man:httpd.service(8)

[vagrant@user01 ~]$ sudo systemctl enable --now httpd # 부팅시 자동 시작 & 서비스 바로 시작       
# enable만 하면 부팅 시에만 적용됨
```

> restart, reload
```
# restart는 서비스를 완전히 중지했다가 다시 시작
# reload는 서비스를 종료하지 않고 설정 파일만 다시 불러옴
```

| 명령어           | 의미                                   | 재부팅 후 적용 |
|-----------------|--------------------------------------|----------------|
| start           | 지금 바로 서비스 실행                 | ❌             |
| stop            | 지금 바로 서비스 종료                 | ❌             |
| restart         | 서비스 재시작 (stop + start)         | ❌             |
| reload          | 서비스 중지 없이 설정만 다시 적용    | ❌             |
| enable          | 부팅 시 자동 시작 설정               | ✅             |
| disable         | 부팅 시 자동 시작 해제               | ✅             |
| status          | 서비스 상태 확인                      | -              |


