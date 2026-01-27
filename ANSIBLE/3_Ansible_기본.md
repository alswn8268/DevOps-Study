# 3.1 인벤토리
- Ansible은 인프라에 존재하는 여러 호스트를 관리하는데, 이런 호스트의 목록 또는 그룹을 지정한 인벤토리 파일이 필요함
- 인벤토리가 정의되면 패턴을 사용하여 Ansible을 실행할 노드 또는 그룹을 선택함

## 3.1.1 정적 인벤토리
- 사용자가 직접 INI 또는 YAML 형식으로 파일을 직접 작성함

### 3.1.1.1 기본 그룹
- all: 모든 호스트 포함
- ungrouped: 그룹에 속하지 않은 모든 호스트 포함

### 3.1.1.2 호스트 범위
- 호스트 범위 지정 시 스트라이드(시퀀스 번호 증가)를 지정할 수 있음
```bash 
vim inventory

new-vm

# INI 형식 
[webserver] 
servera
serverb
serverc

[database]
system1
system2
system3
192.168.56.[10:100]   

[loadbalancer]                                                                             
web-lb                                                                                     
db-lb                                                                                      

[servers:children]                                                                         
webserver                
```

# 3.2 구성 파일
- Ansible 구성 파일은 Ansible 작동 방식을 구성하는 파일로, 인벤토리 파일의 위치, 관리 노드에 연결하는 방법, 연결 후 작동 방법 등 무수히 많은 구성을 지정할 수 있음
- 기본 ansible 구성 파일의 위치는 `/etc/ansible/ansible.cfg`이며, 변형된 INI 방식을 사용함

## 3.2.1 Ansible 구성 파일 우선 순위
1. ANSIBLE_CONFIG 환경 변수
2. 현재 디렉토리의 ansible.cfg
3. 홈 디렉토리의 ~/.ansible.cfg
4. /etc/ansible/ansible.cfg
참고) 우선 순위가 높은 파일에 정의된 값이 이전 정의된 값보다 우선함

- 현재 적용되는 구성 파일 확인
``` bash
ansible --version
```
## 3.2.2 Ansible 작동 방식 제어 우선 순위
- Ansible 작동 방식을 제어하기 위해 Ansible 구성 파일 외에도 ansible 명령의 옵션, 플레이북 키워드, 변수를 이용해 동작을 제어할 수 있음. 카테고리의 우선 순위는 다음과 같음
1. -e 옵션에 지정한 변수
2. 변수
3. 플레이북 키워드
4. 명령의 옵션
5. Ansible 구성 파일
참고) 우선 순위가 높은 파일에 정의된 값이 이전 정의된 값보다 우선함

## 3.2.3 일반적인 구성 파일 설정
- Ansible 구성 파일은 [section] 대괄호로 묶어진 여러 섹션이 있음. 각 섹션에는 `키 = 값` 설정이 포함됨

### [defaults] 섹션
- inventory: 인벤토리 파일의 위치 (기본: /etc/ansible/ansible.cfg)
- remote_user: SSH 인증을 하기 위한 사용자 (기본: 현재 사용자)
- ask_pass: SSH 인증을 하기 위한 패스워드 요청 및 입력 여부 (기본: false)
### [privilege_escalation] 섹션
- become: 권한 상승 여부 (기본: false)
- become_method: 권한 상승 방법 (기본: sudo)
- become_user: 권한 상승할 사용자 (기본: root)
- become_ask_pass: 권한 상승 방법의 패스워드 요청 및 입력 여부 (기본: false)


```bash
[vagrant@ansible-controller ~]$ mkdir first_day
[vagrant@ansible-controller ~]$ cd first_day/
[vagrant@ansible-controller first_day]$ vim ansible.cfg
# ansible 기본 설정 파일 생성 (우선순위 1: 현재 작업 디렉토리의 ansible.cfg)

[defaults]
inventory=inventory # 기본 인벤토리 파일로 현재 디렉토리의 inventory 사용
remote_user=vagrant # ssh 기본 접속 사용자
ask_pass = false # ssh 접속할 때 이 서버를 신뢰할 지 묻는 과정 생략

[privilege_escalation]
become = true # sudo 권한 활성화
become_user = root # root 권한으로 명령 실행
become_method = sudo # sudo 사용 방식
become_ask_pass = false # sudo 비밀번호 묻지 않음

[vagrant@ansible-controller first_day]$ ansible all --list-hosts
[WARNING]: Unable to parse /home/vagrant/first_day/inventory as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
  hosts (0):

[vagrant@ansible-controller first_day]$ cp ~/inventory .

[vagrant@ansible-controller first_day]$ ansible all --list-hosts | head -n3
  hosts (100):
    new-vm
    web-lb
```

누구세여나야 보람ㅋㅋㅋ
나 집에서 원격 접속으로 하는중 열심히 하네 보기좋다야 너가 캐리해주는거지??
아니지 은선님이 다 해주실거야
엥
누구지
미주언니? 은미언니?  나랑ㄲㅔ 보람
보람언니가 아니라는데
당신누구야!!!
  ㅎㅎ 보람
아니야 보람언니는 본인이라고 안 밝힐 것 같아
미주언니 아니야?


# 3.3 관리 노드 연결
- 기본적으로 OpenSSH를 통해 관리 노드에 접근함. OpenSSH는 ControlPersist(성능 향상 기능), Kerberos 및 점프 호스트(~/.ssh/config) 옵션을 지원함


```bash
[vagrant@ansible-controller first_day]$ ansible -m ping webserver
servera | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname servera: Name or service not known",
    "unreachable": true
}

[vagrant@ansible-controller first_day]$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/.ssh/id_rsa
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:gWD5Y8l0KcVUEJ/6NlFvY20SdG6VsBLM6Mhx2RiisVQ vagrant@ansible-controller
The key's randomart image is:
+---[RSA 3072]----+
|    o+.E=*X. .o +|
|   .o.*o=* =...+.|
|     *o+* o.... o|
|      *o + ... + |
|     . .S .   * o|
|         . . o + |
|          +      |
|         . .     |
|                 |
+----[SHA256]-----+

[vagrant@ansible-controller first_day]$ ls -l ~/.ssh/id_rsa*
-rw-------. 1 vagrant vagrant 2610 Nov 20 06:09 /home/vagrant/.ssh/id_rsa # 개인키는 소유자만 권한이 있음
-rw-r--r--. 1 vagrant vagrant  580 Nov 20 06:09 /home/vagrant/.ssh/id_rsa.pub # 공개키는 모두에게 권한이 있음
```
```bash
vagrant@ansible-controller first_day]$ ssh-copy-id vagrant@192.168.56.201
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host '192.168.56.201 (192.168.56.201)' can't be established.
ED25519 key fingerprint is SHA256:OJUv7B5w3TNIwf8BHM/nolb69JzCktD4b7KgrzoedW8.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? grep PasswordAuth* /etc/ssh/sshd_config
Please type 'yes', 'no' or the fingerprint: yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@192.168.56.201's password: 
Permission denied, please try again.
vagrant@192.168.56.201's password:

[vagrant@ansible-controller first_day]$ ls ~/.ssh/known_hosts
/home/vagrant/.ssh/known_hosts
[vagrant@ansible-controller first_day]$ ssh vagrant@192.168.56.202
The authenticity of host '192.168.56.202 (192.168.56.202)' can't be established.
ED25519 key fingerprint is SHA256:ZMZFhQ3VJhat8hi3s8R3xG2UP7HSkDiEodzI0XGAVjA.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? no
Host key verification failed.
```
# Ansible 초기 구성 순서

- ansible 설치 ( 패키지 설치 ) **→ control-node**

```bash
dnf install -y ansible
dnf install -y ansible-core

# dnf는 repo에 저장소가 등록되어있어야 사용 가능
# pip는 python과 pip가 설치되어있어야 쓸 수 있다.
```

- 모든 시스템에 작업할 사용자인 ansible이라는 사용자를 **모든 서버**에 만들기

```bash
useradd ansible # 사용자 생성
passwd ansible # 비밀번호 설정
```

- 권한 설정 → /etc/sudoers.d에 만든 사용자 모든 권한 쓸 수 있게 (모든 서버)

```bash
vi /etc/sudoers.d/ansible

%ansible ALL=(ALL) NOPASSWD: ALL
```
→ ansible 사용자로 키 기반 인증으로 접속해서 sudo 권한으로 패스워드 없이 작업 가능하도록 설정 준비하기

- 구성 파일 생성  **→ control-node**
  ansible.cfg = Ansible “사용법 설정”

```bash
vi /etc/ansible/ansible.cfg  # 이건 전체 설정
vi ansible.cfg            # 홈디렉토리 밑 사용자 설정 (해서 지금은 이걸 사용)
[defaults]
inventory = inventory
remote_user = ansible
ask_pass = false

[privilege_escalation]
become = true
become_user = root
become_method = sudo
become_ask_pass = false   
```
- 인벤토리 생성 **→ control-node**
  inventory = ansible가 접속할 “서버 목록 + 그룹 + 변수”

```bash
vi inventory
[control]
192.168.56.200
[node1]
192.168.56.201
[node2]
192.168.56.202  

# 적용
ansible all --list-hosts -i inventory
```

- 키 생성 **→ control-node**

```bash
ssh-keygen
```

- 키 복사 **→ control-node 에서 각 노드 대상**으로 실행
  - 만약 키 복사가 안되는 경우
    ssh 패스워드 인증 허용 설정을 각 노드에서 해줘야 복사할 수 있다.
    → 보통 기본적으로 되어있지만 확인해보기

```bash
ssh-copy-id ansible@192.168.56.200
ssh-copy-id ansible@192.168.56.201
ssh-copy-id ansible@192.168.56.202
```
- 실행

```bash
ansible -m command -a id all

# -m : 모듈 설정 → command : 리눅스 명령어 사용할 수 있게
# id all : 사용자 모두 보이게
```

## 3.3.3 SSH 이외 다른 연결 방법
- Ansible은 `smart`라는 연결 방식을 사용하는데, 이는 `OpenSSH` 연결을 먼저 시도하고, OpenSSH에 `ControlPersist` 기능이 없는 버전인 경우 `Paramiko SSH`로 접근을 시도함

### 3.3.3.1 주요 연결 플러그인
- docker, kubectl, local, network_cli, paramiko_ssh, ssh(기본값), winrm

#### 기타 연결 방법 확인
```bash
ansible-doc -t connection -l
```

### 3.3.3.2 연결 방법 변경
- 인벤토리 파일에 환경 변수로 지정하는 방법
```bash
foo.example.com ansible_connection=local
```

- ansible/ansible-playbook 명령을 사용하는 방법
`-c CONNECTION` 또는 `--connection CONNECTION` 옵션 사용 

# 3.4 Ad-Hoc 명령
- 하나 이상의 관리 노드에 단일 작업을 실행하는 임시 명령. 거의 반복되지 않은 간단한 작업에 주로 사용함
- 서버 재부팅, 파일 관리, 패키지 관리, 사용자 및 그룹 관리, 서비스 관리, 팩스 변수 수집...
## 3.4.1. Ad-hoc 명령 사용

> ansible [pattern] -m [module] -a "[module options]"

- -m: 모듈 이름 지정 (기본값: command)
- -a: 모듈의 옵션 / 아규먼트



```bash
[vagrant@ansible-controller second]$ ansible -m ping all
192.168.56.200 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.56.202 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.56.201 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

```bash
[vagrant@ansible-controller second]$ ansible -m command -a id all
192.168.56.200 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
192.168.56.201 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
192.168.56.202 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[vagrant@ansible-controller second]$ ansible -m command -a id all -b

192.168.56.200 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
192.168.56.202 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
192.168.56.201 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

### ansible 동작
1. 기본 동작은 ssh로 접속해서 작업
2. ssh로 접속할 때 사용자는? remote_user를 ansible로 설정
  -> ssh ansible@localhost id -> ansible
3. 구성 파일에서 remote_user 값을 vagrant 라고 설정
  -> ssh vagrant@localhost id -> vagrant
4. 구성 파일에서 become을 true로 했을 때, 
  -> ssh ansible@localhost sudo id -> root
5. 추가설정으로 ansible_connection 값을 local 이라고 하는 경우
  -> id -> 현재 ansible 명령어를 실행한 사용자 (vagrant)
6. 5번에 + become 설정까지 true로 하면
  -> sudo id -> root

```
[vagrant@ansible-controller second]$ ansible -m ping all -b
192.168.56.200 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.56.202 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.56.201 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
[vagrant@ansible-controller second]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):  
/home/vagrant/.ssh/id_rsa already exists.
Overwrite (y/n)? n

[vagrant@ansible-controller second]$ ansible node1 -m command -a "/sbin/reboot" -u ansible -b
192.168.56.201 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: Shared connection to 192.168.56.201 closed.",
    "unreachable": true
}

```

### ssh로 원격 접속 시 동작
1. control node에서 node1으로 ssh 접속을 처음 시도
  - node1 시스템에 있는 /etc/ssh/ssh_host_xxx.pub의 내용을 복사
  - control node에 있는 ~/.ssh/knownhosts 파일에 추가
  - 접속
2. 이후 다시 control node에서 node1로 ssh 접속을 시도
  - control node에 있는 ~/.ssh/knownhosts 파일에 내용을 확인
  - node1 시스템에 있는 /etc/ssh/ssh_host_xxx.pub의 내용과 비교
  - 일치하면 접속
  - 일치하지 않을 경우에는 host key xxxxxx 에러가 발생


```
[vagrant@ansible-controller second]$ ansible node1 -m file -a "dest=/tmp/a.txt state=touch mode=600"
192.168.56.201 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "dest": "/tmp/a.txt",
    "gid": 0,
    "group": "root",
    "mode": "0600",
    "owner": "root",
    "secontext": "unconfined_u:object_r:user_tmp_t:s0",
    "size": 0,
    "state": "file",
    "uid": 0
}
[vagrant@ansible-controller second]$ ssh ansible@192.168.56.201 ls -l /tmp/a.txt
-rw-------. 1 root root 0 Nov 21 03:04 /tmp/a.txt
[vagrant@ansible-controller second]$ ansible node1 -m file -a "dest=/tmp/a.txt mode=644"
192.168.56.201 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/tmp/a.txt",
    "secontext": "unconfined_u:object_r:user_tmp_t:s0",
    "size": 0,
    "state": "file",
    "uid": 0
}
[vagrant@ansible-controller second]$ ssh ansible@192.168.56.201 ls -l /tmp/a.txt
-rw-r--r--. 1 root root 0 Nov 21 03:04 /tmp/a.txt
```

```
[vagrant@ansible-controller second]$ ansible node1 -m yum -a "name=httpd state=present"
192.168.56.201 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Installed: httpd-filesystem-2.4.62-4.el9_6.4.noarch",
        "Installed: mod_http2-2.0.26-4.el9_6.1.x86_64",
        "Installed: httpd-tools-2.4.62-4.el9_6.4.x86_64",
        "Installed: httpd-core-2.4.62-4.el9_6.4.x86_64",
        "Installed: rocky-logos-httpd-90.16-1.el9.noarch",
        "Installed: httpd-2.4.62-4.el9_6.4.x86_64",
        "Installed: apr-util-openssl-1.6.1-23.el9.x86_64",
        "Installed: apr-util-1.6.1-23.el9.x86_64",
        "Installed: apr-1.7.0-12.el9_3.x86_64",
        "Installed: mod_lua-2.4.62-4.el9_6.4.x86_64",
        "Installed: apr-util-bdb-1.6.1-23.el9.x86_64"
    ]
}

[vagrant@ansible-controller second]$  ansible node1 -m user -a "name=user01 state=present"
192.168.56.201 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1002,
    "home": "/home/user01",
    "name": "user01",
    "shell": "/bin/bash",
    "state": "present",
    "stderr": "useradd: warning: the home directory /home/user01 already exists.\nuseradd: Not copying any file from skel directory into it.\nCreating mailbox file: File exists\n",       
    "stderr_lines": [
        "useradd: warning: the home directory /home/user01 already exists.",
        "useradd: Not copying any file from skel directory into it.",
        "Creating mailbox file: File exists"
    ],
    "system": false,
    "uid": 1002
}

[vagrant@ansible-controller second]$ ansible node1 -m service -a "name=httpd state=started enabled=true"
...
```

### Ad-hoc 명령 실습
* 실습 시 모듈 사용법은 ansible-doc <모듈명>으로 확인
아래의 모든 작업은 node2 시스템에만 설정

1. copy 모듈로 컨트롤 노드의 /etc/hosts 파일을 /tmp 디렉토리에 복사하기
```bash
[vagrant@ansible-controller second]$ ansible node2 -m copy -a "src=/etc/hosts dest=/tmp"
[vagrant@ansible-node2 ~]$ cat /tmp/hosts
```
2. file 모듈로 복사한 hosts 파일의 권한을 666으로 설정
```bash
[vagrant@ansible-node2 ~]$ ls -l /tmp/hosts
-rw-r--r--. 1 root root 206 Nov 21 04:50 /tmp/hosts
[vagrant@ansible-controller second]$ ansible node2 -m file -a "dest=/tmp/hosts mode=666"
[vagrant@ansible-node2 ~]$ ls -l /tmp/hosts
-rw-rw-rw-. 1 root root 206 Nov 21 04:50 /tmp/hosts
```
3. file 모듈로 /tmp/new-file 이라는 빈 파일을 생성
```bash
[vagrant@ansible-controller second]$ ansible node2 -m file -a "dest=/tmp/new-file state=touch"
[vagrant@ansible-node2 ~]$ ls /tmp/new-file
```
4. file 모듈로 /tmp/new-dir 이라는 디렉토리를 생성
```bash
[vagrant@ansible-controller second]$ ansible node2 -m file -a "dest=/tmp/new-dir/ state=directory"
[vagrant@ansible-node2 ~]$ ls /tmp
```

5. file 모듈로 /tmp/new-file 이라는 파일 삭제
```bash
[vagrant@ansible-controller second]$ ansible node2 -m file -a "dest=/tmp/new-file state=absent"
[vagrant@ansible-node2 ~]$ ls /tmp/new-file
```

6. file 모듈로 /tmp/new-dir 디렉토리의 소유자를 vagrant, 소유 그룹을 wheel로 변경
```bash
[vagrant@ansible-controller second]$ ansible node2 -m file -a "dest=/tmp/new-dir owner=vagrant group=wheel state=directory"
```

7. user 모듈로 newuser라는 사용자 생성
```bash
[vagrant@ansible-controller second]$ ansible node2 -m user -a "name=newuser"
```

8. user 모듈로 uid가 2025인 user-year 사용자 생성
```bash
[vagrant@ansible-controller second]$ ansible node2 -m user -a "name=user-year uid=2025" 
[vagrant@ansible-controller second]$ ansible node2 -m command -a "id user-year"
```

9. user 모듈로 newuser 사용자의 쉘을 /bin/sh로 변경
```bash
[vagrant@ansible-controller second]$ ansible node2 -m user -a "name=newuser shell=/bin/sh"
```

10. yum 모듈로 httpd 패키지 설치
```bash
[vagrant@ansible-controller second]$ ansible node2 -m yum -a "name=httpd state=present"
```

11. yum 모듈로 git 패키지를 최신 버전으로 설치
```bash
[vagrant@ansible-controller second]$ ansible node2 -m yum -a "name=git state=latest"
```

12. service 모듈로 httpd 서비스를 실행하고 활성화
```bash
[vagrant@ansible-controller second]$ ansible node2 -m service -a "name=httpd state=started enabled=true"
```

