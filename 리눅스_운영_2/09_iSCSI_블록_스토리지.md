# 9.1 iSCSI 소개
## 9.1.1 스토리지 (Storage) 구성
- 스토리는 데이터를 저장하는 스토리지 혹은 저장 기술을 의미함
- 처음에는 시스템의 내장 디스크를 사용했으나, 저장 공간과 물리적인 제약에서 벗어나고자 DAS, NAS, SAN 방식으로 발전함
### DAS (Direct Attached Storage)
- 스토리지 장치와 시스템과 케이블을 통해 직접 연결되는 방식
- `ATA`, `SATA`, `SCSI`, `SAS` 등의 케이블 사용
- 스토리지 장치를 독자적으로 사용할 수 없고, 케이블을 통해 직접 연결된 시스템에서만 접근할 수 있기 때문에, 한 번에 하나의 시스템에서만 접근 가능
- 설치 시에 시스템이 위치한 공간에 인접해야 함
- 블록 단위
- 가격이 저렴, 확장 용이, 보안에 유리

### NAS (Network Attached Storage)
- 네트워크를 통해 연결된 스토리지 장치
- 파일 기반 스토리지
- 파일 I/O에 최적화된 서버
- 스토리지 기능을 강조해서 OS나 하드웨어 및 소프트웨어적인 기술을 변경해서 보다 나은 I/O 성능 제공
- 공간적인 제약에서 벗어나 사용이 가능하고 DAS와는 달리 하나의 스토리지에 여러 시스템이 접근 가능

### SAN (Storage Area Network)
- 네트워크를 통해 스토리지 장치를 연결하고 사용하는 방식의 하나로, 스토리지 전용으로 사용하는 네트워크를 별도로 가짐 => 신뢰성이 높고 처리 속도가 빠름
- 블록 기반 스토리지 
- `FC-SAN` 방식, `IP-SAN` 방식이 있음
#### FC-SAN
- 구리 케이블이나 광섬유 케이블을 모두 사용할 수 있지만, 구리 케이블은 거리가 매우 짧기 때문에 실제로는 광섬유를 사용
- 광섬유를 사용하면 손실이 적고 대역폭을 크게 늘릴 수 있지만 케이블 및 주변 장치를 모두 교체해야하기 때문에 초기 비용이 크게 발생하는 단점이 있음
- 사용 거리는 조금 늘어났지만 여전히 거리 제약이 존재하며, 비용적인 문제가 있음
#### IP-SAN
- `AoE(ATA over Ethernet)`, `FCoE(Fiber Channel over Ethernet)`, `iSCSI(Internet Small Computer System Interface)` 등이 있음
- AoE나 FCoE는 저렴하고 간단하게 저장장치로 접근할 수 있도록 돕는 기술이지만 이더넷 기반으로 동작해서 장거리로 사용하기에는 무리가 있음
- iSCSI 방식은 TCP/IP 기반으로 동작하므로 장거리에도 적합하고 FC-SAN에 비해 가격이 저렴하여 많이 사용

## 9.1.2 블록 스토리지 ★
### 파일 기반 스토리지
  - 주로 파일 서버나 NAS에서 사용하는 방식
  - 제공하는 대상이 파일 단위로 검색 및 접근하게 되고, 일반적으로 우리가 사용하는 계층적 파일 시스템
### 블록 기반 스토리지
  - DAS, SAN 등에서 사용하는 방식
  - 제공하는 대상을 블록 디바이스로 제공하고 접근함 => 접근하는 사용자는 해당 스토리지를 일반적인 디스크장치처럼 인식하고 사용할 수 있음

## 9.1.3 iSCSI 용어 설명
### 기본 용어
- `타겟 (Target)`: 스토리지 장치를 제공하는 시스템으로 서버 역할을 함 ★
- `초기자 (Initiator)`: 타겟에서 제공하는 스토리지 장치에 연결해서 블록 스토리지를 제공받는 클라이언트 시스템 ★
- `IQN (iSCSI Qualified Name)`: 타겟과 초기자의 이름

### 타겟 용어
- `TPG (Target Portal Group)`: 대상 포털 그룹으로서 `ACL`, `LUN`, `Portal` 세 항목을 하나의 그룹으로 만든 연결에 대한 설정집
- `ACL (Access Control List)`: 접근 제어 리스트로서 타겟에 의해 제공되는 스토리지에 연결할 수 있는 초기자를 지정하는 항목
- `LUN (Logical Unit Number)`: 초기자에게 제공할 스토리지 장치에게 부여된 논리 장치 번호
- `Portal`: 초기자가 타겟에 연결할 때 사용하는 IP 주소와 포트번호

### 초기자 용어
- `Discovery`: 초기자에서 연결하려는 대상을 검색하기 위한 단계
- `Login`: `Discovery`에서 발견한 대상으로 연결하는 단계

```
[vagrant@iscsi-server ~]$ sudo su - 
[root@iscsi-server ~]# yum install -y targetcli

# /backstore/block은 실제 저장 공간, 물리적 블록 장치를 의미

# /dev/sdb를 iSCSI에서 사용할 실제 스토리지 공간으로 등록, 가상의 디스크(disk1)
/> /backstores/block create name=disk1 dev=/dev/sdb 
# IQN은 iSCSI Qualified Name으로, iSCSI 네트워크에서 디바이스를 식별하는 고유 이름
/> /iscsi create wwn=iqn.2025-11.com.example:storage1
# ACL(Access Control List)를 설정하여 어떤 클라이언트가 접근할 수 있는지 허용을 지정
/> /iscsi/iqn.2025-11.com.example:storage1/tpg1/acls create iqn.2025-11.com.example:client1
# 위에서 생성한 실제 블록 장치를 iSCSI 타깃의 LUN으로 연결
/> /iscsi/iqn.2025-11.com.example:storage1/tpg1/luns create /backstores/block/disk1 
# => "iqn.2025-11.com.example:storage1" 라는 타깃에 "LUN 0"으로 연결된 디스크 하나(disk1)가 있음

/> saveconfig
/> exit

# iSCSI는 기본적으로 TCP 3260 포트 사용
[root@iscsi-server ~]# firewall-cmd --add-port=3260/tcp --permanent
[root@iscsi-server ~]# firewall-cmd --reload
[root@iscsi-server ~]# firewall-cmd --list-ports
3260/tcp
```

```
[root@iscsi-client ~]# yum install -y iscsi-initiator-utils
InitiatorName=iqn.2025-11.com.example:client1

```

# 9.3 iSCSI 초기자
## 9.3.1 도구 설치
```
[root@iscsi-client ~]# systemctl start iscsi
# 서버 검색
[root@iscsi-client ~]# iscsiadm -m discovery -t st -p 192.168.56.144
# -m: iSCSI 타깃 검색 모드 /-t: sendtargets 방식 (서버가 제공하는 모든 타깃 정보 조회) /-p: 타깃 서버의 IP와 포트 지정 (기본 3260)
# 서버의 iSCSI 타깃 정상적으로 발견
192.168.56.144:3260,1 iqn.2025-11.com.example:storage1

# -m node: 특정 타깃과의 연결을 설정 / -T: Target IQN 지정 / -p: 서버 주소 / -l: login (연결)
[root@iscsi-client ~]# iscsiadm -m node -T iqn.2025-11.com.example:storage1 -p 192.168.56.
144 -l
Logging in to [iface: default, target: iqn.2025-11.com.example:storage1, portal: 192.168.56.144,3260]
Login to [iface: default, target: iqn.2025-11.com.example:storage1, portal: 192.168.56.144,3260] successful.

# 현재 연결된 세션과 LUN 정보를 상세히 보여줌
[root@iscsi-client ~]# iscsiadm -m session -P 3 
iSCSI Transport Class version 2.0-870
version 6.2.1.9
Target: iqn.2025-11.com.example:storage1 (non-flash)
        Current Portal: 192.168.56.144:3260,1
        Persistent Portal: 192.168.56.144:3260,1
                **********
                Interface:
                **********
                ...
                Attached SCSI devices:
                ************************
                Host Number: 1  State: running
                scsi1 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdb          State: running 
                        # iSCSI 서버의 디스크(/dev/sdb)가 클라이언트에서 새로운 로컬 디스크처럼 인식


[root@iscsi-client ~]# fdisk /dev/sdb # w 눌러서 저장 잊지 말기!
[root@iscsi-client ~]# mkfs.xfs /dev/sdb1
[root@iscsi-client ~]# mkdir -p /mnt/iscsi
[root@iscsi-client ~]# mount /dev/sdb1 /mnt/iscsi/
[root@iscsi-client ~]# echo "iscsi" > /mnt/iscsi/test.file
[root@iscsi-client ~]# cat /mnt/iscsi/test.file
iscsi

[root@iscsi-client ~]# echo "/dev/sdb1 /mnt/iscsi xfs defaults 0 0" >> /etc/fstab
[root@iscsi-client ~]# tail -n1 /etc/fstab

[root@iscsi-client ~]# umount /mnt/iscsi
[root@iscsi-client ~]# df -h | grep iscsi
[root@iscsi-client ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   50G  0 disk
├─sda1   8:1    0  600M  0 part /boot/efi
├─sda2   8:2    0    1G  0 part /boot
└─sda3   8:3    0 48.4G  0 part /
sdb      8:16   0   20G  0 disk

[root@iscsi-client ~]# iscsiadm -m node -T iqn.2025-11.com.example:storage1 -p 192.168.56.
144 -u

[root@iscsi-client ~]# iscsiadm -m node -T iqn.2025-11.com.example:storage1 -p 192.168.56.144 -o delete
```

