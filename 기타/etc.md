** 컨트롤 노드에는 리눅스 명령어로 직접 설정, 나머지 관리 대상 노드에는 앤서블로 설정해보기 **

1. 각 시스템에 다음 조건에 맞는 사용자를 설정하세요.
    1) ansible-user 이라는 이름의 사용자 생성
    2) 사용자의 UID는 2000
    3) 사용자의 패스워드는 123
    4) 패스워드 없이 sudo 명령어를 사용할 수 있게 설정
    5) 컨트롤 노드의 vagrant 사용자로 키 기반 인증 방식의 접속을 할 수 있도록 설정
```
[vagrant@al-controller ~]$ sudo useradd ansible-user -u 2000
[vagrant@al-controller ~]$ sudo passwd ansible-user
123
123
[vagrant@al-controller ~]$ sudo vi /etc/sudoers.d/ansible-user
%ansible ALL=(ALL) NOPASSWD: ALL
```

2. 각 시스템에 다음 조건에 맞게 파일을 생성 및 수정해보세요.
    1) /tmp/empty-file 이라는 빈 파일을 생성하세요.
-> [vagrant@al-controller ~]$ touch /tmp/empty-file
    2) /tmp/newdir  이라는 디렉토리를 생성하세요.
-> [vagrant@al-controller ~]$ mkdir /tmp/newdir
    3) /tmp/myfile 이라는 파일을 생성하세요. 이 파일은 내용물로 hello-world 라는 한 줄이 있습니다.
-> [vagrant@al-controller ~]$ echo "hello-world" | sudo tee /tmp/myfile
    4) 컨트롤 노드의 /etc/hosts 파일을 각 시스템의 root 사용자의 홈 디렉토리 아래에 hosts 라는 이름의 파일로 복사하세요.
-> [vagrant@al-controller ~]$ sudo cp /etc/hosts /root/hosts
    5) /tmp/myfile 이라는 파일의 내용에 add text 라는 글자 한 줄을 추가해보세요.
-> [vagrant@al-controller ~]$ echo "add text" | sudo tee -a /tmp/myfile
    6) /tmp/myfile 파일의 내용에서 hello-world 라는 글자를 good-bye 라고 수정해보세요.
-> 
    7) /root/hosts 파일의 끝에 각 시스템들의 주소와 이름을 한 줄씩 추가 입력해주세요.
->
    8) /root/hosts 파일의 링크 파일을 /tmp 디렉토리에 생성하세요.
->
    9) /tmp/myfile 파일의 소유자를 root, 소유 그룹을 wheel, 권한은 600으로 변경하세요.
->
    10) /tmp/empty-file 파일을 삭제하세요.
-> [vagrant@al-controller ~]$ rm -f /tmp/empty-file

3. 각 시스템에 패키지를 설치하세요.
    1) 컨트롤 노드에는 httpd에 mariadb-server 패키지를 둘 다 설치합니다.
    2) 노드1과 노드2는 하나의 플레이에서 조건문을 활용해서 노드1에는 httpd를, 노드2에는 mariadb-server를 설치하세요.

4. 각 시스템에 서비스를 활성화하세요.
    1) 컨트롤 노드에는 httpd와 mariadb 서비스를 활성화하세요.
    2) 노드1과 노드2에는 하나의 작업으로 각각의 서비스를 활성화할 수 있도록 변수를 활용합니다. 변수는 host_vars 디렉토리에 각각 노드1은 httpd, 노드2는 mariadb로 설정해서 만드세요.

5. 방화벽을 설정합니다.
    1) 컨트롤 노드에는 http와 mariadb가 사용하는 포트번호를 확인해서 둘 다 설정하세요.
    2) 하나의 작업을 통해서 방화벽 구성을 서로 다르게 설정합니다.
    4-2와 같은 방식으로 변수를 설정해서, 노드1에는 http 포트를 현재만 활성화
    노드2에는 mariadb 포트를 현재와 영구 모두 활성화해보세요.

6. 안전한 ssh 서버 구성을 위해 설정 파일을 수정해서 모든 시스템이 ssh 접속 시 root 사용자는 접속이 안 되고 일반 사용자는 키 기반 인증만 가능하도록 설정하고 적용시키세요.