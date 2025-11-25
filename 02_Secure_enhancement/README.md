## 서버 보안강화
(아래 내용의 코드블럭 안은 Linux 명령어 or PowerShell 명령어 입니다.)<br>
### 학습 목표: 인수인계 받은 서버의 보안을 강화합니다.
### 1. SSH보안 강화

1) SSH포트 변경, root 로그인 금지 
- Brute-force 형태의 공격면적을 줄이기 위해 SSH포트변경, root로그인 금지를 설정합니다.
    ```bash
    # vi 편집기로 sshd_config 파일을 열어 Port 변경 22 > 2222
    # root 로그인 금지 PermitRootLogin yes > no
    sudo vi /etc/ssh/sshd_config
    
    # 데몬 리로드, SSH restart 후 상태 확인
    sudo systemctl daemon-reload
    sudo systemctl restart ssh
    sudo systemctl status ssh
    ```
    <img src="../screenshots/02_Secure_enhancement/01_Portchange.png" width="800">
    <img src="../screenshots/02_Secure_enhancement/01-1_ Portcheck.png" width="800">
<br>

2) 비밀번호 로그인 금지
- 비밀번호 로그인 금지를 설정하기 전에 SSH 키 생성과 공개키 서버 업로드를 합니다.
    ```bash
    # MS PowerShell 에서 아래의 명령어로 공개키와 개인키를 생성합니다.
    ssh-kegen -t ed25519
    ```
    <img src="../screenshots/02_Secure_enhancement/01-2_ keygen.png" width="800"><br>
    ```bash
    # MS PowerShell 에서 아래 명령어로 공개키를 열고, 키 안의 문자열 전체를 복사합니다.
    cat $env:USERPROFILE\.ssh\id_ed25519.pub

    # 복사한 문자열을 Linux 서버에서 아래 명령어를 사용해 authorized_keys 생성 시 붙여 넣습니다.
    mkdir -p ~/.ssh
    nano ~/.ssh/authorized_keys

    # 공개키의 권한을 설정합니다.
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys

    # 키가 준비 됐으니 비밀번호 로그인을 꺼줍니다. PasswordAuthentication yes > no
    sudo vi /etc/ssh/sshd_config
    ```
    : sshd_config 파일은 이번 보안 조치로 최종적으로 아래와 같이 설정 되어야 합니다.
    - Port 2222
    - PermitRootLogin <span style="color:red">no</span>
    - PasswordAuthentication <span style="color:red">no</span>
    - PubkeyAuthentication <span style="color:green">yes</span>
<br><br>

- 최종 결과를 확인 합니다.<br>
    : 기본포트(22)로 접속 실패, root 접속 실패, 포트 2222로 로그인 시 패스워드를 입력하지 않음(개인키로 로그인 했기 때문)
    <img src="../screenshots/02_Secure_enhancement/01-3_ keylogin.png" width="800"><br>


### 2. Fail2ban 확장 & 커스텀 jail 구성



    ```bash
	ip a
    ip route | grep default
    ```
    <img src="../screenshots/01_Server_handover/02_ipinfo.png" width="800">
<br>