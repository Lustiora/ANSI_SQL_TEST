# 🛠️ SQL 학습 및 실습 환경 가이드

이 프로젝트는 **PostgreSQL**을 기반으로 다양한 스키마의 데이터베이스를 활용하여 SQL 역량을 향상하는 것을 목표로 합니다.

## 1. 사용 DBMS 및 데이터베이스 (Database)

실습은 **PostgreSQL** 환경에서 진행되며, 아래 3가지 샘플 데이터베이스를 사용합니다.

* **DBMS:** [PostgreSQL](https://www.postgresql.org/)
* **실습용 DB 목록:**
1. **ClassicModelsShop** ([MySQL Sample Database](https://www.mysqltutorial.org/getting-started-with-mysql/mysql-sample-database/))
2. **DVDRental** ([PostgreSQL Sample Database](https://www.postgresqltutorial.com/postgresql-sample-database/))
3. **Yongmazon** ([Oracle Sample Database](https://www.oracletutorial.com/getting-started/oracle-sample-database/))

## 2. 📌 참고 및 주의사항

타 DBMS(MySQL, Oracle) 기반의 샘플 데이터를 PostgreSQL에서 사용하기 위해 다음 사항을 유의해주세요.

* **데이터 변환 필요:** MySQL, Oracle용 `.sql` 파일은 PostgreSQL 문법에 맞게 변환해야 합니다.
> *Tip: AI에게 `.sql` 파일을 입력하여 PostgreSQL용으로 변환을 요청하거나, DBeaver 등의 도구를 활용하세요.*

* **답안 기준:** 모든 문제의 정답 쿼리(Issue)는 **PostgreSQL 문법**을 기준으로 작성되었습니다.
* **힌트 정책:** 단계(Step)와 난이도가 상승할수록, 문제 해결 능력을 기르기 위해 힌트는 제공되지 않습니다.
* **프로젝트 목표:** [튜닝 : DB 엔진의 물리적 동작 방식을 이용하여 극한의 성능 추구](https://github.com/Lustiora/ANSI_SQL_TEST/issues/58)

## 3. 📚 유용한 도구 및 참고 자료

실습을 진행하면서 참고하면 좋은 도구와 문서입니다.

* **SQL 클라이언트 도구:** [DBeaver](https://dbeaver.io/) (추천)
* **PostgreSQL 튜토리얼:** [w3schools PostgreSQL Tutorial](https://www.w3schools.com/postgresql/index.php)
* **문법 비교:** [DBMS별 사용 가능 구문 정리](https://www.sql-workbench.eu/dbms_comparison.html)
* **심화 학습:** [PostgreSQL 성능 최적화 가이드](https://edbkorea.com/blog/postgresql-oltp-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94/)

## 4. ✅ 사전 권장 문제 (Warm-up)

본 프로젝트의 문제를 풀기 전, 기초 실력을 점검하기 위해 아래 사이트의 문제들을 먼저 풀어보시는 것을 추천합니다.

* **국문:** [프로그래머스 SQL 고득점 Kit](https://school.programmers.co.kr/learn/challenges?tab=sql_practice_kit)
* **영문:** [LeetCode Top SQL 50](https://leetcode.com/studyplan/top-sql-50/)

---

### 📚 SQL 학습 커리큘럼

| 스텝 | 주제 (Title) |  | 상세 기술 정의 | 요구 역량 |
| --- | --- | --- | --- | --- |
| **Step 1** | **논리적 구조화** | 🏗️ | 정규화된 테이블에서의 정확한 데이터 추출 | ERD(개체 관계) 이해, `JOIN`(Inner/Outer) 및 기초 집계(`GROUP BY`) 능력 |
| **Step 2** | **가독성과 효율성** | 🧹 | 유지보수가 쉽고 논리적인 쿼리 작성 | `CTE`(With절) 활용, `CASE WHEN`을 통한 조건 분기, 가독성 높은 코드 습관 |
| **Step 3** | **물리적 최적화** | ⚡ | DB 내부 동작 원리를 고려한 성능 튜닝 | 인덱스(Index) 구조 이해, **SARGable**(인덱스 태우기) 쿼리 작성, 실행 계획 분석 |
| **Step 4** | **고급 분석** | 📈 | 순서와 흐름, 순위를 다루는 심화 통계 | **Window Function**(`RANK`, `LAG`, `LEAD`) 및 `PARTITION BY` 활용 능력 |
| **Step 5** | **데이터 재구조화** | 🔄 | 리포팅을 위한 데이터 형태 변환 (행  열) | `PIVOT`/`UNPIVOT` 개념 이해, `GROUP BY`를 응용한 데이터 차원 변경 능력 |
| **Step 6** | **서브쿼리와 필터링** | 🔍 | 복합 조건 및 데이터 간의 존재 여부 판단 | **상관 서브쿼리**, `EXISTS`, `IN` 등을 활용한 메인 쿼리와의 논리적 연결 제어 |
| **Step 7** | **데이터 가공 함수** | 🎨 | 최종 출력을 위한 포맷팅 (Presentation) | 문자열 자르기(`SPLIT`), 날짜 변환(`TO_CHAR`) 등 데이터 시각화를 위한 전처리 |
| **Step 8** | **복합 데이터 타입** | 📦 | 관계형 DB 내 비정형 데이터(NoSQL) 처리 | **JSON 파싱**(`->>`) 및 배열 처리 등 최신 DB 트렌드에 맞춘 하이브리드 쿼리 작성 |

---

### 📝 SQL 문제

#### 🥉 Lv.1 Bronze (기초 체력) 
* (단일 테이블 조회 및 단순 필터링) (기본적인 `SELECT`, `WHERE` 문법 이해 및 단순 조건절 작성 능력)

*기본적인 데이터 추출 및 단순 현상 파악 업무입니다.*

* **[Step 1 🏗️]** [배우별 출연작 장르 다양성 분석](https://github.com/Lustiora/ANSI_SQL_TEST/issues/7)
* **[Step 3 ⚡]** [대용량 테이블 전체 조회(Full Scan) 발생 원인 분석](https://github.com/Lustiora/ANSI_SQL_TEST/issues/40)
* **[Step 3 ⚡]** [특정 쿼리의 성능 저하 병목 구간 진단](https://github.com/Lustiora/ANSI_SQL_TEST/issues/42)
* **[Step 3 ⚡]** [인덱스 적용 실패 및 비효율적 스캔 원인 규명](https://github.com/Lustiora/ANSI_SQL_TEST/issues/44)
* **[Step 4 📈]** [심판별 채점 결과 비교 데이터 추출](https://github.com/Lustiora/ANSI_SQL_TEST/issues/61)

#### 🥈 Lv.2 Silver (데이터 가공) 
* (기초적인 데이터 집계 및 가공) (`GROUP BY`, 내장 함수(문자열/날짜)를 활용한 1차원적인 데이터 요약 능력)

*데이터를 정리하여 통계 지표를 만드는 업무입니다.*

* **[Step 2 🧹]** [카테고리별 평균 연체 기간 현황 파악](https://github.com/Lustiora/ANSI_SQL_TEST/issues/3)
* **[Step 2 🧹]** [장기 미활동(Churn) 예상 고객 리스트 추출](https://github.com/Lustiora/ANSI_SQL_TEST/issues/28)
* **[Step 2 🧹]** [회원 가입 이메일 도메인 점유율 통계](https://github.com/Lustiora/ANSI_SQL_TEST/issues/32)
* **[Step 2 🧹]** [영화별 전체 출연진 목록(Cast List) 정리](https://github.com/Lustiora/ANSI_SQL_TEST/issues/34)
* **[Step 5 🔄]** [보고서 작성을 위한 서식 최적화](https://github.com/Lustiora/ANSI_SQL_TEST/issues/73)
* **[Step 7 🎨]** [보고서 작성을 위한 날짜 표기 형식 표준화](https://github.com/Lustiora/ANSI_SQL_TEST/issues/81)

#### 🥇 Lv.3 Gold (논리적 연결) 
* (복합 논리 설계 및 다중 테이블 결합) (`JOIN`, `HAVING`, 서브쿼리 등 2개 이상의 개념을 조합하여 데이터 관계를 파악하는 능력)

*여러 데이터를 조합하여 비즈니스 인사이트를 도출하거나, 성능 이슈를 해결하는 단계입니다.*

* **[Step 1 🏗️]** [해외 지사별 매출 효율성 및 기여도 분석](https://github.com/Lustiora/ANSI_SQL_TEST/issues/5)
* **[Step 1 🏗️]** [대륙별 물류 창고 재고 자산 가치 평가](https://github.com/Lustiora/ANSI_SQL_TEST/issues/9)
* **[Step 2 🧹]** [우수 고객(VIP) 구매 패턴 기반 그룹핑](https://github.com/Lustiora/ANSI_SQL_TEST/issues/26)
* **[Step 3 ⚡]** [조회 시나리오별 최적 인덱스 선정 전략](https://github.com/Lustiora/ANSI_SQL_TEST/issues/46)
* **[Step 3 ⚡]** [게시판 목록 로딩 지연 현상 해결](https://github.com/Lustiora/ANSI_SQL_TEST/issues/48)
* **[Step 3 ⚡]** [복합 조건 조회 시 발생하는 속도 저하 개선](https://github.com/Lustiora/ANSI_SQL_TEST/issues/52)
* **[Step 3 ⚡]** [댓글 조회 서비스의 시스템 CPU 부하 최적화](https://github.com/Lustiora/ANSI_SQL_TEST/issues/54)
* **[Step 4 📈]** [회원별 최근 대여 기록 조회](https://github.com/Lustiora/ANSI_SQL_TEST/issues/36)
* **[Step 4 📈]** [고객별 평균 재구매 주기 산출](https://github.com/Lustiora/ANSI_SQL_TEST/issues/38)
* **[Step 4 📈]** [일별 전체 매출 추세 모니터링 리포트](https://github.com/Lustiora/ANSI_SQL_TEST/issues/59)
* **[Step 4 📈]** [부서별 급여 상위 구간 직원 현황](https://github.com/Lustiora/ANSI_SQL_TEST/issues/63)
* **[Step 4 📈]** [일일 방문자 증감(Diff) 추이 분석](https://github.com/Lustiora/ANSI_SQL_TEST/issues/65)
* **[Step 5 🔄]** [데이터 구조 변환](https://github.com/Lustiora/ANSI_SQL_TEST/issues/71)
* **[Step 6 🔍]** [일별 매출 합산 후 평균 지표 산출](https://github.com/Lustiora/ANSI_SQL_TEST/issues/79)

#### 🔶 Lv.4 Platinum (고급 테크닉 & 최적화) 
* (실행 계획 최적화 및 데이터 구조 변형) (인덱스 원리 이해(최적화), `PIVOT`, 윈도우 함수 심화 등 데이터의 물리적/논리적 구조를 제어하는 능력)

*복잡한 분석 요구사항을 처리하거나, 까다로운 성능 문제를 튜닝하는 단계입니다.*

* **[Step 2 🧹]** [제품 라인별 고수익 프리미엄 상품군 식별](https://github.com/Lustiora/ANSI_SQL_TEST/issues/13)
* **[Step 2 🧹]** [VIP 고객 세그먼트별 구매 성향 심층 분석](https://github.com/Lustiora/ANSI_SQL_TEST/issues/17)
* **[Step 3 ⚡]** [옵티마이저 실행 계획 이상(Anomaly) 점검](https://github.com/Lustiora/ANSI_SQL_TEST/issues/50)
* **[Step 3 ⚡]** [대용량 과거 로그 데이터 조회 속도 개선](https://github.com/Lustiora/ANSI_SQL_TEST/issues/56)
* **[Step 4 📈]** [영업 사원별 주력 판매 카테고리 분석](https://github.com/Lustiora/ANSI_SQL_TEST/issues/1)
* **[Step 4 📈]** [월간 매출 성장률(MoM) 지표 산출](https://github.com/Lustiora/ANSI_SQL_TEST/issues/11)
* **[Step 4 📈]** [라인별 재고 부족 위험군 조기 탐지](https://github.com/Lustiora/ANSI_SQL_TEST/issues/15)
* **[Step 4 📈]** [연도별 베스트셀러 및 매출 비중 분석](https://github.com/Lustiora/ANSI_SQL_TEST/issues/19)
* **[Step 4 📈]** [카테고리별 연간 매출 성장률 (YoY) 분석 보고서](https://github.com/Lustiora/ANSI_SQL_TEST/issues/87)
* **[Step 5 🔄]** [매장별 가격대 선호도 매트릭스 구성](https://github.com/Lustiora/ANSI_SQL_TEST/issues/30)
* **[Step 5 🔄]** [임원 보고용 대시보드 데이터 포맷팅](https://github.com/Lustiora/ANSI_SQL_TEST/issues/69)
* **[Step 5 🔄]** [접속 로그(User Agent) 파싱 및 환경 데이터 구조화](https://github.com/Lustiora/ANSI_SQL_TEST/issues/77)
* **[Step 5 🔄]** [부서별 직원 근태 유형 요약 보고서](https://github.com/Lustiora/ANSI_SQL_TEST/issues/85)
* **[Step 6 🔍]** [구매 이력이 존재하는 실사용 고객 명단 추출](https://github.com/Lustiora/ANSI_SQL_TEST/issues/75)

#### 💎 Lv.5 Diamond (알고리즘 & 마스터) 
* (알고리즘 구현 및 절차적 사고) (`RECURSIVE CTE`(재귀), 복잡한 비즈니스 로직(계층, 경로 분석)을 SQL로 구현하는 프로그래밍적 사고력)

*계층 구조나 경로 분석 등 복잡한 로직을 구현해야 하는 최상위 업무입니다.*

* **[Step 4 📈]** [조직 계층을 반영한 팀별 누적 실적 집계](https://github.com/Lustiora/ANSI_SQL_TEST/issues/21)
* **[Step 4 📈]** [관리자별 관할 지역 시장 점유율 산출](https://github.com/Lustiora/ANSI_SQL_TEST/issues/24)
* **[Step 4 📈]** [전체 조직 구조의 계층(Depth) 측정](https://github.com/Lustiora/ANSI_SQL_TEST/issues/67)
* **[Step 8 📦]** [Chrome 사용자들의 월별 구매력 분석 보고서](https://github.com/Lustiora/ANSI_SQL_TEST/issues/83)