# 1. 카테고리별 평균 연체 기간 현황 파악

---

### DB : PostgreSQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`rental`** (대여 기록)

* `rental_id`: **대여 고유 ID** (PK)
* `inventory_id`: **재고 ID** (FK)
* `rental_date`: **대여 일시** (TIMESTAMP)
* `return_date`: **반납 일시** (TIMESTAMP) - 반납되지 않은 경우 **NULL**

**`inventory`** (재고 정보)

* `inventory_id`: **재고 고유 ID** (PK)
* `film_id`: **영화 ID** (FK)

**`film`** (영화 정보)

* `film_id`: **영화 고유 ID** (PK)
* `title`: **영화 제목**
* `rental_duration`: **대여 허용 기간** (INTEGER, 단위: 일/Days) - 이 기간을 넘기면 연체로 간주

**`film_category`** (영화-카테고리 연결)

* `film_id`: **영화 ID** (FK)
* `category_id`: **카테고리 ID** (FK)

**`category`** (카테고리 정보)

* `category_id`: **카테고리 ID** (PK)
* `name`: **카테고리 이름** (예: Action, Drama)

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
대여 기간을 초과하여 반납된(연체된) 기록들을 분석하고자 합니다.
영화 카테고리 (**Genre**) 별로 사람들이 약속된 기간보다 얼마나 늦게 반납하는지 성향을 파악해 주세요.

**[연체 여부 판단 (Overdue Logic)]**

* `rental` 테이블의 실제 대여 기간 (`return_date` - `rental_date`) 이 `film` 테이블에 명시된 `rental_duration` (일 단위) 보다 긴 경우를 '연체'로 간주합니다.
* 아직 반납되지 않은 (**return_date** 가 **NULL** 인) 건은 분석에서 제외합니다.

**[계산할 지표]**

1. **Category Name**: 카테고리 이름 (`name`)
2. **Overdue Count**: 해당 카테고리에서 발생한 총 연체 횟수
3. **Average Overdue Duration**: 약속된 기간보다 평균적으로 며칠이나 더 늦게 반납했는지 계산

---

## 3. 원하는 결과 (Expected Output)

* 평균 연체 기간 (**Average Overdue Duration**) 이 가장 긴 카테고리부터 내림차순으로 정렬하세요.

| name | overdue_count | avg_overdue_duration |
| --- | --- | --- |
| New | 509 | 2 day 20:27:42.554027 |
| Sports | 648 | 2 day 15:08:57.685185 |
| Documentary | 571 | 2 day 14:01:21.751313 |
| ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **Type Casting (형변환):** `rental_date` 와 `return_date` 의 차이는 **INTERVAL** 타입이지만, `rental_duration` 은 단순 **숫자** (**INTEGER**) 입니다. 숫자를 기간(예: 5일)으로 바꾸려면 어떻게 해야 할까요?
* PostgreSQL의 `INTERVAL` 키워드를 활용하거나, 곱하기 연산 (`* '1 day'::interval`) 을 활용해 보세요.


* **조건절:** `WHERE` 절에서 "실제 대여 기간" > "허용 대여 기간" 조건을 비교할 때도 위에서 고민한 형변환이 필요합니다.
* **연체 기간 계산:** "얼마나 더 늦었는지"는 `(실제 대여 기간) - (허용 대여 기간)` 으로 구할 수 있습니다.

---

# 2. 장기 미활동(Churn) 예상 고객 리스트 추출

---

### DB : MySQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`customers`** (고객 정보)

* `customerNumber`: **고객 고유 ID** (PK)
* `customerName`: **고객사 이름**

**`orders`** (주문 정보)

* `orderNumber`: **주문 번호** (PK)
* `customerNumber`: **고객 ID** (FK)
* `orderDate`: **주문 일자**

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
작년에는 단골이었지만, 올해 들어 소식이 끊긴 **'이탈 우려 고객'** 을 찾으려 합니다.
**2004년** 에는 주문 이력이 최소 1건 이상 존재하지만, **2005년** 에는 주문 이력이 단 한 건도 없는 고객 목록을 추출해 주세요.

**[대상 고객]**

* **2004년** (`2004-01-01` ~ `2004-12-31`) 주문 유저 (집합 A)
* **2005년** (`2005-01-01` ~ `2005-12-31`) 주문 유저 (집합 B)

**[구하는 것]**

* A에는 속하지만 B에는 속하지 않는 고객 (A - B)

**[권장 사항 (Step 2 목표)]**
이 문제는 `NOT IN`, `LEFT JOIN / IS NULL`, `NOT EXISTS`, `EXCEPT` 등 여러 방법으로 풀 수 있습니다.
그중 **가독성이 좋고, 직관적인 방법** 을 선택해서 풀어보세요. (작성 후 왜 그 방식을 택했는지 설명해 주시면 더 좋습니다!)

---

## 3. 원하는 결과 (Expected Output)

* `customerNumber` 를 기준으로 오름차순 정렬하세요.

| customerNumber | customerName |
| --- | --- |
| 103 | Atelier graphique |
| 112 | Signal Gift Stores |
| 114 | Australian Collectors, Co. |
| ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **가독성:** "2005년에 주문이 존재하는가?"를 묻는 `WHERE NOT EXISTS (...)` 패턴은 문장처럼 읽혀서 이해하기 쉽습니다.
* **집합 연산:** PostgreSQL에서는 `EXCEPT` (차집합) 연산자를 지원합니다. 이를 사용하면 코드가 얼마나 짧아질까요?
* **NOT IN의 함정:** `NOT IN` 은 서브쿼리 결과에 **NULL** 이 하나라도 포함되어 있으면, 전체 결과가 아무것도 나오지 않는 치명적인 특징이 있습니다. (이 데이터셋에는 NULL이 없지만, 습관이 중요합니다.)

---

# 3. 회원 가입 이메일 도메인 점유율 통계

---

### DB : PostgreSQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`customer`** (고객 정보)

* `customer_id`: **고객 고유 ID** (PK)
* `email`: **고객 이메일** (예: `mary.smith@sakilacustomer.org`)

---

## 2. 문제 상황 (Scenario)

**[마케팅 팀의 요청]**
"고객들이 주로 사용하는 이메일 서비스 제공업체 (**Domain**) 가 어디인지 파악해 달라고 합니다.
`customer` 테이블의 `email` 컬럼을 분석하여, 도메인 부분만 추출하고 각 도메인별 사용자 수를 집계해 주세요."

**[문제 사양]**

* **도메인 추출:** 이메일 주소의 **'@'** 기호 뒷부분을 도메인으로 정의합니다.
* 예: `mary.smith@sakilacustomer.org` → `sakilacustomer.org`


* **집계 및 정렬:** 도메인별로 몇 명의 고객이 있는지 세어주세요 (`user_count`). 사용자 수가 많은 순서대로 내림차순 정렬하세요.

---

## 3. 원하는 결과 (Expected Output)

| domain | user_count |
| --- | --- |
| sakilacustomer.org | 599 |

---

#### 힌트 (고민을 돕는 질문)

* **전통적인 방법 (Hard way):** "전체 문자열에서 **'@'** 의 위치를 찾고, 그 위치 +1부터 끝까지 잘라내라." 머리 아프죠? 🤕 (`SUBSTRING`, `POSITION` 조합)
* **PostgreSQL 전용 함수 (Smart way):** "문자열을 **'@'** 로 쪼갠 다음, 그중 **2번째 조각** 을 줘." 아주 쉽습니다!
* PostgreSQL에는 특정 구분자 (**Delimiter**) 를 기준으로 문자를 쪼개서 가져오는 강력하고 직관적인 전용 함수가 있습니다. (힌트: `SPLIT_PART`)

# 4. 영화별 전체 출연진 목록(Cast List) 정리

---

### DB : PostgreSQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`film`** (영화 정보)

* `film_id`: **영화 고유 ID** (PK)
* `title`: **영화 제목**

**`film_actor`** (영화-배우 연결)

* `film_id`: **영화 ID** (FK)
* `actor_id`: **배우 ID** (FK)

**`actor`** (배우 정보)

* `actor_id`: **배우 고유 ID** (PK)
* `first_name`: **이름** (First Name)
* `last_name`: **성** (Last Name)

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
영화 목록을 출력하는데, 각 영화에 출연한 배우들의 이름을 **'한 줄 (One Row)'** 로 보고 싶습니다.
영화 제목과 함께, 그 영화에 출연한 배우들의 이름을 **콤마** (`,`) 로 구분된 하나의 문자열로 합쳐서 출력해 주세요.

**[출력 단위]**

* 영화 (`title`) 하나당 무조건 **한 줄** 만 나와야 합니다. (배우가 여러 명이어도 행이 늘어나면 안 됩니다.)

**[배우 이름 포맷]**

* `first_name` 과 `last_name` 을 합쳐서 **풀 네임** 으로 만드세요. (예: `Tom Hanks`)
* 여러 명일 경우 **이름 순** (알파벳 순) 으로 정렬해서 합쳐주세요.

**[필터링]**

* 출연 배우가 **10명 이상** 인 '초호화 캐스팅' 영화만 출력하세요.

---

## 3. 원하는 결과 (Expected Output)

* `title` 을 기준으로 오름차순 정렬하세요.

| title | actor_list |
| --- | --- |
| Academy Dinosaur | Christian Gable, Johnny Cage, Lucille Tracy, Mary Keitel, Mena Temple, Oprah Kilmer, Penelope Guiness, Rock Dukakis, Sandra Peck, Warren Nolte |
| Arabia Dogma | Burt Posey, Frances Tomei, Greta Malden, Johnny Cage, Jude Cruise, Julia Mcqueen, Karl Berry, Lisa Monroe, Rip Crawford, Russell Bacall, Sean Williams, Walter Torn |
| Berets Agent | Angela Witherspoon, Cate Harris, Grace Mostel, Ian Tandy, Jessica Bailey, Julia Barrymore, Julia Fawcett, Julianne Dench, Meryl Allen, William Hackman |
| ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **행을 열로 합치기:** 보통 `GROUP BY` 는 숫자를 더하거나 세는 데 쓰지만, **문자열** (Text) 을 합치는 집계 함수도 있습니다.
* **함수 사용법:** `STRING_AGG` 함수를 찾아보세요. (`STRING_AGG(컬럼명, 구분자 ORDER BY 정렬기준)`)
* **이름 합치기:** `first_name || ' ' || last_name`
* **필터링:** 집계 결과(배우 수)에 대한 조건은 `WHERE` 절이 아니라 `HAVING` 절을 사용해야 합니다.

# 5. 보고서 작성을 위한 서식 최적화

---

### DB : 

---

## 1. 테이블 정보 (Schema)

**`team_projects` (팀 프로젝트 명단)**

* `team_name`: 팀 이름
* `member_name`: 팀원 이름

| team_name | member_name |
| --- | --- |
| **A팀** | **철수** |
| **A팀** | **영희** |
| **B팀** | **민수** |

---

## 2. 문제 설명 (Problem)

**[상황]**
현재 데이터는 팀원 한 명당 한 줄씩 출력됩니다.
보고서 공간이 부족해서, **팀별로 한 줄만** 출력하되 팀원 이름은 콤마(`,`)로 이어서 보여달라고 합니다.

**[요구 사항]**
`team_projects` 테이블을 변환하여, **팀 이름(`team_name`)** 과 **팀원 리스트(`members`)** 를 출력하세요.
(팀원 이름 사이에는 `, ` 콤마와 공백을 넣어주세요.)

**[원하는 결과]**

| team_name | members |
| --- | --- |
| A팀 | 철수, 영희 |
| B팀 | 민수 |

---

# 6. 보고서 작성을 위한 날짜 표기 형식 표준화

---

### DB : 

---

## 1. 테이블 정보 (Schema)

**`posts` (게시글 정보)**

* `post_id`: 게시글 번호
* `created_at`: 작성일 (DATE 타입)

| post_id | created_at |
| --- | --- |
| 1 | 2024-05-05 |
| 2 | 2024-12-25 |

---

## 2. 문제 설명 (Problem)

**[상황]**
현재 날짜가 `YYYY-MM-DD` (예: 2024-05-05) 형태로 저장되어 있습니다.
보고서 작성을 위해 이를 **"2024년 05월"** 이라는 **한글 포맷**으로 변환하여 출력해야 합니다.

**[요구 사항]**
`posts` 테이블의 `created_at` 컬럼을 조회하되, **`TO_CHAR`** 함수를 사용하여 아래와 같은 문자열 형태로 출력하세요.

* **힌트:** 연도는 `YYYY`, 월은 `MM`입니다. 한글(`년`, `월`)을 포맷 안에 넣을 때는 큰따옴표(`"`)로 감싸주세요.

**[원하는 결과]**

| upload_date |
| --- |
| 2024년 05월 |
| 2024년 12월 |