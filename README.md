## SQL 문제를 찾아다니다가 AI로 만들어 공부하기 시작했습니다.

### 사용 DBMS , DB
- DBMS : [PostgreSQL](https://www.postgresql.org/)
- DB : ClassicModelsShop [MySQL Sample Database](https://www.mysqltutorial.org/getting-started-with-mysql/mysql-sample-database/)
- DB : DVDRental [PostgreSQL Sample Database](https://www.postgresqltutorial.com/postgresql-sample-database/)
- DB : Yongmazon [Oracle Sample Database](https://www.oracletutorial.com/getting-started/oracle-sample-database/)

### 참고 사항
- PostgreSQL에서 MySQL, Oracle Sample Database 를 사용하려면 변환이 필요합니다.
- DB `.sql` 을 AI 에 입력하여 PostgreSQL 용으로 편집을 요청하시거나 해당하는 DBMS를 사용하시면 됩니다.
- 답안 Issue는 PostgreSQL 문법으로 작성되어있습니다.
- [목표](https://github.com/Lustiora/ANSI_SQL_TEST/issues/58)

### 참고 링크
- [DBeaver](https://dbeaver.io/)<br>
- [w3schools PostgreSQL Tutorial](https://www.w3schools.com/postgresql/index.php)<br>
- [DBMS 별 사용 가능 구문 정리](https://www.sql-workbench.eu/dbms_comparison.html)
- [PostgreSQL 성능 최적화](https://edbkorea.com/blog/postgresql-oltp-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94/)

### 해당 Project의 문제를 푸시기 전에 하단 링크의 문제를 풀어보시는걸 추천합니다.
- [programmers SQL 고득점 Kit](https://school.programmers.co.kr/learn/challenges?tab=sql_practice_kit)
- 영문 [leetcode](https://leetcode.com/studyplan/top-sql-50/)

## 난이도 (Difficulty)

### **하 (Low)** :
* 단일 테이블 혹은 단순 2개 테이블 조인.
* 기본적인 `WHERE`, `GROUP BY`, `ORDER BY` 사용.
* 문제의 요구사항이 SQL 문법과 1:1로 직관적으로 매칭되는 경우.

### **중 (Medium)** :
* 3개 이상의 테이블 조인, 혹은 서브쿼리(Subquery) 활용.
* `CASE WHEN`, `COALESCE`, 날짜/문자열 함수 등의 활용.
* 논리적인 단계가 2단계 이상 필요함 (예: 먼저 집계하고, 그 결과를 다시 필터링).

### **상 (High)** :
* **윈도우 함수(Window Function)** 의 심화 활용 (`LAG`, `LEAD`, `Frame` 설정 등).
* **재귀 쿼리(Recursive CTE)**, Self-Join, 복잡한 집합 연산.
* 성능 최적화(Index) 고려 필요, 혹은 데이터 구조를 완전히 바꾸는(Pivot) 작업.

## 문제 목록

### **Step 1 (논리적 구조화)** : 정확한 데이터 추출, 다중 조인(Join), 기본 집계.
- **Low** ["천의 얼굴"을 가진 배우 찾기 (GROUP BY + HAVING)](https://github.com/Lustiora/SQL-TEST/issues/7)
- **Middle** [전 세계 지사(Office)별 매출 효율성 분석 (Fan-out)](https://github.com/Lustiora/SQL-TEST/issues/5)
- **Middle** [대륙별 창고 재고 자산 가치 평가 (Multi-join)](https://github.com/Lustiora/SQL-TEST/issues/9)

### **Step 2 (가독성과 효율성)** : `CTE`, `CASE WHEN`, `FILTER`, 문자열 처리 등 "깔끔한 쿼리" 작성.
- **Middle** [카테고리별 평균 연체 기간 분석 (날짜/시간 Function)](https://github.com/Lustiora/SQL-TEST/issues/3)
- **Middle** [VIP 고객 분류 및 소비 패턴 분석 (RFM 변형) (Join + 집계 + CTE)](https://github.com/Lustiora/SQL-TEST/issues/26)
- **Middle** [이탈 고객(Churn) 발굴](https://github.com/Lustiora/SQL-TEST/issues/28)
- **Middle** [고객 이메일 도메인 분석](https://github.com/Lustiora/SQL-TEST/issues/32)
- **Middle** [영화별 출연 배우 리스트](https://github.com/Lustiora/SQL-TEST/issues/34)
- **High** [제품 라인별 "평균 이상"의 프리미엄 제품 찾기 (중첩 CTE)](https://github.com/Lustiora/SQL-TEST/issues/13)
- **High** [VIP 고객과 구매 성향 분석 (중첩 CTE)](https://github.com/Lustiora/SQL-TEST/issues/17)

### **Step 3 (물리적 최적화)** : 인덱스 원리, SARGable(가공 금지), 실행 속도 고려.
- **Low** [인덱스가 있음에도 불구하고 1,000만 건 전체를 뒤지는(Full Table Scan) 현상이 발생하여 속도가 매우 느림](https://github.com/Lustiora/ANSI_SQL_TEST/issues/40)
- **Low** [성능 저하의 주범찾기](https://github.com/Lustiora/ANSI_SQL_TEST/issues/42)
- **Low** [인덱스를 타지 않고 Full Table Scan이 발생](https://github.com/Lustiora/ANSI_SQL_TEST/issues/44)
- **Middle** [어떤 인덱스가 더 효율적인가?](https://github.com/Lustiora/ANSI_SQL_TEST/issues/46)
- **Middle** [자유게시판 게시글 목록 로딩이 너무 느려요!](https://github.com/Lustiora/ANSI_SQL_TEST/issues/48)
- **Middle** [인덱스를 걸었는데 왜 이렇게 조회가 느려](https://github.com/Lustiora/ANSI_SQL_TEST/issues/52)
- **Middle** [게시글 가장 최근 댓글 10개를 출력하는데 CPU 사용량 100%](https://github.com/Lustiora/ANSI_SQL_TEST/issues/54)
- **High** [인덱스를 걸었는데 왜 안 타지?](https://github.com/Lustiora/ANSI_SQL_TEST/issues/50)
- **High** [옛날 로그를 보는데 왜 이렇게 느릴까](https://github.com/Lustiora/ANSI_SQL_TEST/issues/56)

### **Step 4 (고급 분석)** : 순서가 있는 데이터(`LAG`), 계층 구조(재귀), 복잡한 순위(`RANK`).
- **Low** [3명의 심판이 서로 다른 결과를 원한다](https://github.com/Lustiora/ANSI_SQL_TEST/issues/61)
- **Middle** [고객별 '마지막' 대여 영화 찾기](https://github.com/Lustiora/SQL-TEST/issues/36)
- **Middle** [고객별 평균 재구매 주기](https://github.com/Lustiora/SQL-TEST/issues/38)
- **Middle** [매일 아침 리포트로 매출액의 전체적인 추세를 알고 싶다](https://github.com/Lustiora/ANSI_SQL_TEST/issues/59)
- **Middle** [각 부서 별 고연봉자 정리](https://github.com/Lustiora/ANSI_SQL_TEST/issues/63)
- **Middle** [전날 대비 방문자 변동 사항](https://github.com/Lustiora/ANSI_SQL_TEST/issues/65)
- **High** [영업 사원별 "최대 매출" 기여 카테고리 추출 (Join , Window Function)](https://github.com/Lustiora/SQL-TEST/issues/1)
- **High** [월별 매출 성장률(MoM) 분석 (Window Function)](https://github.com/Lustiora/SQL-TEST/issues/11)
- **High** [각 라인별 "재고 부족" 비상 제품 파악 (중첩 CTE)](https://github.com/Lustiora/SQL-TEST/issues/15)
- **High** [2004년 제품 라인별 "베스트셀러"와 매출 기여도 분석 (중첩 CTE)](https://github.com/Lustiora/SQL-TEST/issues/19)
- **High** [조직 계층별 팀 누적 매출 분석 (재귀 CTE)](https://github.com/Lustiora/SQL-TEST/issues/21)
- **High** [매니저별 글로벌 시장 지배력 분석 (재귀 CTE + 집합 집계)](https://github.com/Lustiora/SQL-TEST/issues/24)
- **High** [조직의 계층(Level) 파악](https://github.com/Lustiora/ANSI_SQL_TEST/issues/67)

### **Step 5 (데이터 재구조화)** : 피벗(Pivot), 언피벗(Unpivot), JSON 처리 등 리포팅을 위한 변형.
- **Middle** [매장별 가격대(Price Tier) 선호도 분석](https://github.com/Lustiora/SQL-TEST/issues/30)
- **Middle** [경영진은 한눈에 보이는 데이터가 필요하다](https://github.com/Lustiora/ANSI_SQL_TEST/issues/69)