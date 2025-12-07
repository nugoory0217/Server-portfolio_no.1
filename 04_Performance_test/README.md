## 성능 테스트, 병목분석
(아래 내용의 코드블럭 안은 Linux 명령어 or PowerShell 명령어 입니다.)<br>
### 학습 목표: 서버에 부하가 걸리고 느려지는 경우 병목 포인트를 찾아내는 상황을 시뮬레이션 합니다. 

<br>

### 1. 테스트 준비 - 테스트 용 페이지 제작, 테스트 툴 설치

1) 정적 페이지(static page) 준비
- Nginx만 사용하는 간단한 정적 페이지를 준비합니다.
    ```bash
    sudo vi /var/www/html/static.html
    ```
    ```html
    # html 안은 아래와 같이 작성 해줍니다.
    <!DOCTYPE html>
    <html>
    <head>
    <title>Static Test</title>
    </head>
    <body>
    <h1>Static Page</h1>
    <p>This page is Nginx static file performance test page.</p>
    </body>
    </html>
    ```
<br>

2) PHP + CPU 부하 페이지 준비
- PHP-FPM 병목 현상을 재현하기 위한 페이지를 준비합니다.
    ```bash
    sudo vi /var/www/html/cpu_test.php
    ```
    ```php
    <?php
    // CPU를 일부러 많이 쓰는 코드 (비효율적인 루프)
    $start = microtime(true);

    $sum = 0;
    for ($i = 0; $i < 5000000; $i++) {
        $sum += sqrt($i);
    }

    $end = microtime(true);
    $elapsed = $end - $start;

    echo "CPU Test Done<br>";
    echo "Elapsed: " . $elapsed . " seconds<br>";
    echo "Sum: " . $sum . "<br>";
    ?>
    ```

3) PHP + DB 부하 페이지 준비
    ```bash
    sudo vi /var/www/html/db_test.php
    ```
    ```php
    <?php
    $start = microtime(true);

    $mysqli = new mysqli("127.0.0.1", "testuser", "testpass", "testdb"); // 테스트를 하려는 DB의 user, 비번, DB 입니다.
    if ($mysqli->connect_errno) {
        echo "DB connection fail: " . $mysqli->connect_error;
        exit;
    }

    // 일부러 DB에 지연을 거는 쿼리 (SELECT SLEEP): 쿼리 하나당 0.2초씩 지연 됩니다.
    for ($i = 0; $i < 20; $i++) {
        $mysqli->query("SELECT SLEEP(0.2)");
    }

    $mysqli->close();

    $end = microtime(true);
    $elapsed = $end - $start;

    echo "DB Test Done<br>";
    echo "Elapsed: " . $elapsed . " seconds<br>";
    ?>
    ```

4) 과부하 테스트 툴 설치
- 웹 서버 부하 테스트 툴인 Apache Bench를 설치 합니다.<br>
  Apache Bench는 웹 서버에 몇 명의 사용자가 동시에 접속해서 요청을 보내도 서버가 버틸 수 있는지 테스트 하는 도구입니다.
  ```bash
  sudo apt update
  sudo apt install apache2-utils -y
  ```
<br><br>

### 2. 병목 테스트

1) 정상 상태 기준 설정 하기
- 정상과 비정상 여부를 확인 하기 위해서는 먼저 정상일 때의 시간을 측정해서 "정상 상태"의 기준을 파악해야 합니다.<br>
  서버 자체의 병목만을 테스트 해야, 병목 발생 시 서버 문제인지 네트워크 문제인지 알수 있기 때문에<br>
  time curl과 ab는 MS PowerShell 같은 클라이언트 측에서 실행하지 않고 서버에서 실행 해야 됩니다.

  1. time 명령어를 활용한 기본 시간 측정
   - time 명령어는 프로그램이나 스크립트의 실행시간을 확인 하는 명령어 입니다.<br>
    우리는 위에서 단순한 정적 페이지, PHP에서 SQRT 500만 번을 돌리는 CPU Heavy Load 페이지,
    DB에 SLEEP을 거는, 지연이 큰 DB 요청 하나를 인위적으로 만들어내는 테스트용 페이지를 작성했습니다.<br>
    테스트 페이지는 대략 평소에 이러한 요청은 "얼마나 걸린다" 하는 가정하에 작성 한것이기에
    실제 서버에서는 그 서버의 용도에 따라 평소에 들어오는 실제 트래픽의 대표 URL을 골라서 정상 기준을 잡아야 합니다.
        ```bash
        # /dev/null : HTML 코드는 보여주지 않고 시간 결과만 보여줍니다.
        time curl -s http://192.168.159.128/static.html > /dev/null
        time curl -s http://192.168.159.128/cpu_test.php > /dev/null
        time curl -s http://192.168.159.128/db_test.php > /dev/null
        ```
        <img src="../screenshots/04_Performance_test/01_Loadtest_standard.png" width="800"><br>
        : time 명령어의 항목 의미는 다음과 같습니다.
        - real: 사람 기준으로 실제 흐른 시간, 시작버튼 누르고 끝날 때까지 걸린 전체 시간
        - user: CPU가 user space에서 쓴 시간, 커널 밖에서 실행되는 시간
        - sys: CPU가 커널 내부에서 사용한 시간, syscall, 메모리 할당 등







