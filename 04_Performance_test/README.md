## 성능 테스트, 병목분석
(아래 내용의 코드블럭 안은 Linux 명령어 or PowerShell 명령어 입니다.)<br>
### 학습 목표: 서버에 부하가 걸리고 느려지는 경우 병목 포인트를 찾아내는 상황을 시뮬레이션 합니다. 

<br>

### 1. 테스트용 페이지 준비

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

    $mysqli = new mysqli("127.0.0.1", "testuser", "testpass", "testdb");
    if ($mysqli->connect_errno) {
        echo "DB connection fail: " . $mysqli->connect_error;
        exit;
    }

    // 일부러 DB에 지연을 거는 쿼리 (SELECT SLEEP)
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

### 2. 병목 테스트
- 테스트 용 페이지들이 준비 완료됐습니다.




