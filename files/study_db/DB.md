## 1. DDL (데이터 정의 언어) 실습

### **문제 1: 테이블 생성하기 (CREATE TABLE)**
* **질문**: `attendance` 테이블은 중복된 데이터가 쌓이는 구조이다. 중복된 데이터는 어떤 컬럼인가?
    * **정답**: `nickname` (출석 기록이 생성될 때마다 동일한 크루의 이름이 계속해서 반복 저장됨)
* **질문**: `attendance` 테이블에서 중복을 제거하기 위해 `crew` 테이블을 만들려고 한다. 어떻게 구성해 볼 수 있을까?
    * **정답**: 고유 식별자인 `id`와 중복을 허용하지 않는 `nickname` 컬럼으로 구성합니다.
* **질문**: `crew` 테이블에 들어가야 할 크루들의 정보는 어떻게 추출할까? (hint: DISTINCT)
    ```sql
    SELECT DISTINCT nickname FROM attendance;
    ```
* **질문**: 최종적으로 `crew` 테이블 생성
    ```sql
    CREATE TABLE crew (
        id INT AUTO_INCREMENT PRIMARY KEY,
        nickname VARCHAR(255) NOT NULL UNIQUE
    );
    ```
* **질문**: `attendance` 테이블에서 크루 정보를 추출해서 `crew` 테이블에 삽입하기
    ```sql
    INSERT INTO crew (nickname) 
    SELECT DISTINCT nickname FROM attendance;
    ```

---

### **문제 2: 테이블 컬럼 삭제하기 (ALTER TABLE)**
* **질문**: `crew` 테이블을 만들고 중복을 제거했다. `attendance`에서 불필요해지는 컬럼은?
    * **정답**: `nickname` (이제 `crew_id`라는 외래키를 통해 `crew` 테이블의 정보를 참조할 수 있기 때문)
* **질문**: 컬럼을 삭제하려면 어떻게 해야 하는가?
    ```sql
    ALTER TABLE attendance DROP COLUMN nickname;
    ```

### **문제 3: 외래키 설정하기**
* **질문**: `crew` 테이블에는 없는 `crew_id`가 `attendance` 테이블에 존재하지 않도록 데이터 무결성을 설정하려면? (ALTER)
    ```sql
    ALTER TABLE attendance 
    ADD CONSTRAINT fk_crew_id 
    FOREIGN KEY (crew_id) REFERENCES crew(id);
    ```

### **문제 4: 유니크 키 설정**
* **질문**: 닉네임의 중복이 금지되는 규칙을 지키기 위해 `crew` 테이블의 결함을 어떻게 해결할 수 있을까? (ALTER)
    ```sql
    ALTER TABLE crew ADD CONSTRAINT UNIQUE (nickname);
    ```

---

## 2. DML (CRUD) 실습

### **문제 5: 크루 닉네임 검색하기 (LIKE)**
* **질문**: 닉네임 첫 글자가 '디'로 시작하는 크루 찾기
    ```sql
    SELECT * FROM crew WHERE nickname LIKE '디%';
    ```

---

### **문제 6: 출석 기록 확인하기 (SELECT + WHERE)**
* **질문**: 어셔의 3월 6일 기록이 실제로 누락되었는지 확인하기
    ```sql
    SELECT * FROM attendance 
    WHERE crew_id = (SELECT id FROM crew WHERE nickname = '어셔') 
    AND date = '2026-03-06';
    ```

---

### **문제 7: 누락된 출석 기록 추가 (INSERT)**
* **질문**: 어셔의 3월 6일 출석 데이터(09:31 등교, 18:01 하교) 추가하기
    ```sql
    INSERT INTO attendance (crew_id, date, start_time, end_time) 
    VALUES ((SELECT id FROM crew WHERE nickname = '어셔'), '2026-03-06', '09:31:00', '18:01:00');
    ```

---

### **문제 8: 잘못된 출석 기록 수정 (UPDATE)**
* **질문**: 주니의 3월 12일 등교 시각을 10:05에서 10:00로 수정하기
    ```sql
    UPDATE attendance 
    SET start_time = '10:00:00' 
    WHERE crew_id = (SELECT id FROM crew WHERE nickname = '주니') 
    AND date = '2026-03-12';
    ```

---

### **문제 9: 허위 출석 기록 삭제 (DELETE)**
* **질문**: 아론의 3월 12일 허위 출석 기록 삭제하기
    ```sql
    DELETE FROM attendance 
    WHERE crew_id = (SELECT id FROM crew WHERE nickname = '아론') 
    AND date = '2026-03-12';
    ```

---

### **문제 10: 출석 정보 조회하기 (JOIN)**
* **질문**: `crew_id` 대신 `nickname` 필드 값을 함께 포함하여 전체 출석 기록 조회하기
    ```sql
    SELECT c.nickname, a.date, a.start_time, a.end_time 
    FROM attendance a 
    JOIN crew c ON a.crew_id = c.id;
    ```

---

### **문제 11: nickname으로 쿼리 처리하기 (서브 쿼리)**
* **질문**: nickname을 입력받아 해당 크루의 출석 기록을 조회하기
    ```sql
    SELECT * FROM attendance 
    WHERE crew_id = (SELECT id FROM crew WHERE nickname = '원하는닉네임');
    ```

---

### **문제 12: 가장 늦게 하교한 크루 찾기 (ORDER BY + LIMIT)**
* **질문**: 3월 5일 가장 늦게 하교한 크루의 닉네임과 하교 시각 조회하기
    ```sql
    SELECT c.nickname, a.end_time 
    FROM attendance a 
    JOIN crew c ON a.crew_id = c.id 
    WHERE a.date = '2026-03-05' 
    ORDER BY a.end_time DESC 
    LIMIT 1;
    ```

---

## 3. 집계 함수 실습

### **문제 13: 크루별로 '기록된' 날짜 수 조회**
```sql
SELECT crew_id, COUNT(*) AS record_count 
FROM attendance 
GROUP BY crew_id;
```

---

### **문제 14: 크루별로 등교 기록이 있는 날짜 수 조회**
```sql
SELECT crew_id, COUNT(start_time) AS attended_days 
FROM attendance 
WHERE start_time IS NOT NULL 
GROUP BY crew_id;
```

---

### **문제 15: 날짜별로 등교한 크루 수 조회**
```sql
SELECT date, COUNT(crew_id) AS crew_count 
FROM attendance 
GROUP BY date;
```

---

### **문제 16: 크루별 가장 빠른 등교 시각(MIN)과 가장 늦은 등교 시각(MAX)**
```sql
SELECT 
    crew_id, 
    MIN(start_time) AS earliest_start, 
    MAX(start_time) AS latest_start 
FROM attendance 
GROUP BY crew_id;
```
