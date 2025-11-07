### **실습 1**

**"company" 그룹을 생성하고, "manager1", "manager2" 사용자를 생성하여 이 그룹에 포함시키세요. 두 사용자의 홈 디렉토리는 "/home/managers/" 아래에 생성하세요.**
-> sudo groupadd company
    **sudo mkdir -p /home/managers **
    sudo useradd -d /home/managers/manager1 -g company -m manager1
    sudo useradd -d /home/managers/manager2 -g company -m manager2

### **실습 2**

**"developer" 사용자를 생성하고, 기본 그룹은 "company", 추가 그룹은 "wheel"로 설정한 후, 계정 만료일을 2026년 3월 15일로 설정하세요.**
    -> sudo useradd -g company -G wheel -e 20260315 developer

### **실습 3**

**"guest123" 사용자를 생성하되, 로그인 쉘을 "/sbin/nologin"으로 설정하고, 비밀번호를 "temp2025"로 설정하세요.**
    -> ** sudo useradd -s /sbin/nologin guest123
    echo "temp2025" | sudo passwd --stdin guest123 **

### **실습 4**

**"trainee" 그룹을 생성하고, "student_a", "student_b", "student_c" 사용자를 생성하여 모두 이 그룹에 속하게 하세요. 홈 디렉토리는 자동 생성되지 않도록 설정하세요.**
  -> sudo groupadd trainee
  sudo useradd -M -g trainee student_a
  sudo useradd -M -g trainee student_b
  sudo useradd -M -g trainee student_c

### **실습 5**

**"intern" 사용자의 기본 그룹을 "trainee"로 변경하고, "company" 그룹을 추가 그룹으로 설정하세요. (intern 사용자가 없으면 먼저 생성)**
  -> sudo useradd -g trainee -G company intern

### **실습 6**

**"testuser" 계정을 생성하고, 이 사용자의 계정이 30일 후에 자동으로 잠기도록 설정하세요.**
  -> sudo useradd testuser
  ** sudo usermod -e $(date -d "+30 days" +%Y-%m-%d) testuser **

### **실습 7**

**"manager1" 사용자가 패스워드 없이 모든 sudo 명령을 실행할 수 있도록 "/etc/sudoers.d/managers" 파일을 생성하여 설정하세요.**
  -> ** sudo visudo -f /etc/sudoers.d/managers 
  manager1 ALL=(ALL) NOPASSWD:ALL
  **

### **실습 8**

**현재 시스템에 생성된 모든 사용자 중 UID가 1000 이상인 사용자들의 정보를 확인하는 명령어를 실행하세요.**
  -> ** awk -F: '$3 >= 1000 {print $0}' /etc/passwd **

### **실습 9**

**매일 오전 6시 30분에 "/var/log/daily_backup.log" 파일에 현재 날짜와 시간을 추가하는 cron job을 설정하세요.**
  -> sudo touch /var/log/daily_backup.log
  crontab -e
  30 6 * * * date >> /var/log/daily_backup.log

### **실습 10**

**매주 월요일 오전 1시에 홈 디렉토리의 ".cache" 폴더를 삭제하는 cron job을 등록하세요.**
  -> ** sudo crontab -e
  ** 0 1 * * 1 rm -rf /home/.cache

### **실습 11**

**2분마다 시스템의 메모리 사용량을 확인하여 "/tmp/memory_check.log"에 기록하는 작업을 설정하세요.**

### **실습 12**

**매월 1일 오후 11시 45분에 "Monthly Report Due" 메시지를 "/var/log/reminders.log"에 추가하는 cron을 설정하세요.**

### **실습 13**

**"/dev/sdb"에 2GB 크기의 파티션을 생성하고, EXT4 파일시스템을 만든 후 "/mnt/data1"에 마운트하세요.**

### **실습 14**

**"/dev/sdb"에 두 번째 파티션을 10GB 크기로 생성하고, XFS 파일시스템을 만든 후 "/mnt/data2"에 마운트하세요.**

### **실습 15**

**위에서 생성한 두 파일시스템이 부팅 시 자동으로 마운트되도록 "/etc/fstab"에 등록하세요.**

### **실습 16**

**"/dev/sdb"의 세 번째 파티션(1GB)을 생성하고 스왑으로 설정한 후 활성화하세요.**

### **실습 17**

**"/mnt/data1"에 "test_file.txt" 파일을 생성하고, "EXT4"라는 내용을 추가하세요.**

### **실습 18**

**"/mnt/data2"의 소유 그룹을 "company"로 변경하고, 그룹 멤버들이 읽기/쓰기 할 수 있도록 권한을 설정하세요.**

### **실습 19**

**"/dev/sdc" 전체를 LVM 파티션으로 설정하고, 물리 볼륨을 생성하세요.**

### **실습 20**

**"vg_production"이라는 볼륨 그룹을 생성하고 위에서 만든 물리 볼륨을 포함시키세요.**

### **실습 21**

**"vg_production"에서 다음 논리 볼륨들을 생성하세요:**

- **"lv_app" (2GB)**
- **"lv_log" (1GB)**
- **"lv_backup" (5GB)**

### **실습 22**

**각 논리 볼륨에 XFS 파일시스템을 생성하고, "/app", "/logs", "/backups" 디렉토리에 각각 마운트하세요.**

### **실습 23**

**"lv_app" 볼륨의 크기를 5GB로 확장하고, 파일시스템도 함께 확장하세요.**

### **실습 24**

**LVM 구성이 부팅 시에도 자동으로 마운트되도록 "/etc/fstab"에 등록하세요.**