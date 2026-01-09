# 1. 연도별 베스트셀러 및 매출 비중 분석

---

### DB : MySQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`orders`** (주문 정보)

* `orderNumber`: **주문 번호** (PK)
* `orderDate`: **주문 일자**
* `status`: **주문 상태** (예: 'Shipped', 'Cancelled')

**`orderdetails`** (주문 상세)

* `orderNumber`: **주문 번호** (PK, FK)
* `productCode`: **제품 코드** (PK, FK)
* `quantityOrdered`: **주문 수량**
* `priceEach`: **판매 단가**

**`products`** (제품 정보)

* `productCode`: **제품 코드** (PK)
* `productName`: **제품 이름**
* `productLine`: **제품 라인** (FK) - 카테고리

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
각 제품 라인별로 **2004년** 에 가장 많은 매출을 기록한 **'최고의 제품'** 을 찾고, 그 제품이 해당 라인 전체 매출에서 **차지하는 비중(%)** 을 계산하세요.

* 단순히 1등을 찾는 것을 넘어, 그 1등 제품이 해당 카테고리 전체 매출의 몇 퍼센트를 견인했는지 (**Contribution**) 파악해야 합니다.
* **중첩 CTE 구조** 를 사용하여 문제를 논리적으로 분해해 보세요.

**[비중 계산 공식]**

* `(해당 제품 매출 / 해당 라인 전체 매출) * 100`

---

## 3. 원하는 결과 (Expected Output)

* 결과는 3행까지만 표시합니다.

| productline | productname | price | all_price | parcent |
| --- | --- | --- | --- | --- |
| Classic Cars | 1992 Ferrari 360 Spider red | 114,556.2 | 1,689,307.07 | 6.7812538072 |
| Motorcycles | 2003 Harley-Davidson Eagle Drag Bike | 81,636.19 | 527,243.84 | 15.4835739759 |
| Planes | 1980s Black Hawk Helicopter | 72,349.6 | 445,464.3 | 16.2413912855 |
| ... | ... | ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **부분(Part)과 전체(Whole)의 연결:** "개별 제품의 매출"과 "라인 전체의 매출"을 어떻게 하나의 행에 나란히 둘 수 있을까요? 일반적인 `GROUP BY` 로는 어렵기 때문에 조인(Join)이나 윈도우 함수가 필요합니다.
* **윈도우 함수의 활용:** `LineTotal` CTE를 따로 만들지 않고도, 윈도우 함수 `SUM(product_revenue) OVER(PARTITION BY productLine)` 을 사용하면 쿼리를 더 단순화할 수 있습니다.
* **순위 필터링:** 최종 쿼리에서 `WHERE rank_num = 1` 조건을 걸 때, 1위가 여러 개일 경우 (동점자) 어떻게 처리할지 (`RANK` vs `ROW_NUMBER`) 고민해 보세요.

---

# 2. 카테고리별 연간 매출 성장률 (YoY) 분석 보고서

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`annual_sales`** (연간 매출 요약)

* `year`: **연도** (YYYY)
* `category`: **상품 카테고리**
* `total_sales`: **총매출액**

| year | category | total_sales |
| --- | --- | --- |
| 2023 | Electronics | 10000 |
| 2024 | Electronics | 15000 |
| 2023 | Fashion | 20000 |
| 2024 | Fashion | 18000 |
| 2025 | Electronics | 30000 |

---

## 2. 문제 상황 (Scenario)

**[경영진의 요청]**
"각 카테고리별로 **작년 대비 올해 매출이 몇 퍼센트 성장했는지 (YoY Growth Rate)** 를 보고 싶습니다.
매출액과 성장률을 함께 보여주는 리포트를 만들어 주세요."

**[요구 사항]**

* **성장률** (`growth_rate`) 은 소수점 첫째 자리까지 반올림하여 **%** 기호와 함께 문자열로 출력하세요. (예: `50.0%`)
* 전년도 데이터가 없는 경우 (첫 해) 는 `NULL` 혹은 비어있어도 됩니다.
* 결과는 `category`, `year` 순으로 정렬하세요.

---

## 3. 원하는 결과 (Expected Output)

| category | year | current_sales | prev_year_sales | growth_rate |
| --- | --- | --- | --- | --- |
| Electronics | 2023 | 10000 | NULL | NULL |
| Electronics | 2024 | 15000 | 10000 | 50.0% |
| Electronics | 2025 | 30000 | 15000 | 100.0% |
| Fashion | 2023 | 20000 | NULL | NULL |
| Fashion | 2024 | 18000 | 20000 | -10.0% |

*(해설: Electronics의 2024년 성장률 = (15000 - 10000) / 10000 * 100 = 50%)*

---

#### 힌트 (고민을 돕는 질문)

* **LAG 함수의 범위 지정:** 이전 매출을 가져올 때, 모든 연도를 다 섞어서 가져오면 안 됩니다. **"같은 카테고리 내에서"** 만 이전 연도를 찾아야 하므로 `PARTITION BY` 옵션이 필요합니다.
* **성장률 공식:** `((올해 매출 - 작년 매출) / 작년 매출) * 100`
* **문자열 포맷팅:** DB에 따라 다르지만, 계산된 숫자를 반올림(`ROUND`)한 뒤 문자열로 변환하고 `%` 기호를 붙이는 연산(`CONCAT` 또는 `||`)이 필요합니다.

---

# 3. 매장별 가격대 선호도 매트릭스 구성

---

### DB : PostgreSQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`rental`** (대여 기록)

* `rental_id`: **대여 ID** (PK)
* `inventory_id`: **재고 ID** (FK)

**`inventory`** (재고 정보)

* `inventory_id`: **재고 ID** (PK)
* `film_id`: **영화 ID** (FK)
* `store_id`: **매장 ID** (FK) - 1호점, 2호점...

**`film`** (영화 정보)

* `film_id`: **영화 ID** (PK)
* `rental_rate`: **대여 가격** (예: 0.99, 2.99, 4.99)

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
각 매장 (**Store**) 별로 손님들이 선호하는 **'가격대'** 가 다른지 분석해 보고자 합니다.
매장 ID (`store_id`) 별로, 대여된 영화들의 가격 (`rental_rate`) 을 기준으로 3가지 등급으로 나누어 각 등급별 대여 횟수를 한 줄에 보여주세요.

**[가격 등급 기준]**

* **Cheap**: 대여료가 **$1.00 이하** (예: 0.99)
* **Regular**: 대여료가 **$1.00 초과 ~ $3.00 이하** (예: 2.99)
* **Premium**: 대여료가 **$3.00 초과** (예: 4.99)

**[집계 방식 (Pivoting)]**

* 결과는 매장별로 단 **한 줄 (Row)** 씩만 나와야 합니다.
* 등급을 컬럼으로 펼쳐서 보여주세요.

---

## 3. 원하는 결과 (Expected Output)

| store_id | cheap_count | regular_count | premium_count |
| --- | --- | --- | --- |
| 1 | 2,754 | 2,505 | 2,664 |
| 2 | 2,898 | 2,615 | 2,608 |

---

#### 힌트 (고민을 돕는 질문)

* **가독성 (Readability):** 보통 이런 문제는 `SUM(CASE WHEN 조건 THEN 1 ELSE 0 END)` 패턴을 많이 사용합니다. 하지만 PostgreSQL에는 이보다 훨씬 직관적이고 읽기 쉬운 문법이 존재합니다.
* **FILTER 구문:** `COUNT(*) FILTER (WHERE ...)` 문법을 사용하면 "조건에 맞는 행만 필터링해서 세어라"라는 로직을 아주 깔끔하게 구현할 수 있습니다.
* **CASE WHEN vs FILTER:** 두 방식 중 편한 방법을 사용해도 좋지만, PostgreSQL 특화 문법인 `FILTER` 를 연습해 보시는 것을 추천합니다.

---

# 4. 임원 보고용 대시보드 데이터 포맷팅

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`store_sales`** (매장별 카테고리 매출)

* `store_name`: **매장명** (강남점, 홍대점)
* `category`: **상품 카테고리** ('Coffee', 'Dessert')
* `sales`: **매출액**

| store_name | category | sales |
| --- | --- | --- |
| **강남점** | **Coffee** | **100** |
| **강남점** | **Dessert** | **50** |
| **홍대점** | **Coffee** | **80** |
| **홍대점** | **Dessert** | **120** |

---

## 2. 문제 상황 (Scenario)

**[상황]**
현재 데이터는 카테고리가 행 (Row) 으로 쌓이는 **세로형 (Long format)** 구조입니다.
경영진은 매장별로 'Coffee'와 'Dessert' 매출을 **한 줄에서 동시에 (Side-by-side)** 비교하고 싶어 합니다.

**[요구 사항]**
`store_sales` 테이블을 변환하여, **매장명** (`store_name`) 하나당 **1개의 행 (Row)** 만 출력되도록 하고, 각 카테고리의 매출액을 별도의 **컬럼 (Column)** 으로 분리하세요.

---

## 3. 원하는 결과 (Expected Output)

| store_name | coffee_sales | dessert_sales |
| --- | --- | --- |
| 강남점 | 100 | 50 |
| 홍대점 | 80 | 120 |

---

#### 힌트 (고민을 돕는 질문)

* **기준 잡기:** 결과표를 보면 `store_name` 이 유니크하게 나옵니다. 즉, **매장명으로 그룹핑 (`GROUP BY`)** 을 해야 합니다.
* **골라 담기:**
* `coffee_sales` 컬럼에는 `category` 가 'Coffee'인 행의 `sales` 만 합쳐야 합니다. ('Dessert'인 행은 무시하거나 0으로 취급)
* `dessert_sales` 컬럼에는 `category` 가 'Dessert'인 행의 `sales` 만 합쳐야 합니다.


* **조건부 집계 (Conditional Aggregation):** `SUM()` 함수 안에서 **조건문 (`CASE WHEN`)** 을 사용하여 원하는 데이터만 쏙쏙 골라서 더해보세요.

---

# 5. 접속 로그(User Agent) 파싱 및 환경 데이터 구조화

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`access_logs`** (접속 로그)

* `log_id`: **로그 번호**
* `info`: **접속 정보** (JSON 타입)

| log_id | info |
| --- | --- |
| 1 | `{"browser": "Chrome", "os": "Windows"}` |
| 2 | `{"browser": "Safari", "os": "Mac"}` |

---

## 2. 문제 상황 (Scenario)

**[상황]**
`info` 컬럼에 브라우저와 OS 정보가 뭉쳐서 (JSON) 들어있습니다.
분석을 위해 여기서 **브라우저 (browser)** 정보만 쏙 뽑아서 일반 텍스트 컬럼처럼 보고 싶습니다.

**[요구 사항]**
`access_logs` 테이블을 조회하여, 아래와 같이 **`browser`** 컬럼을 별도로 출력하세요.

* **PostgreSQL** 문법을 기준으로 작성해 주세요. (화살표 연산자 `->` 또는 `->>` 를 사용합니다.)
* 결과는 **텍스트 (String)** 형태로 나와야 합니다.

---

## 3. 원하는 결과 (Expected Output)

| browser |
| --- |
| Chrome |
| Safari |

---

#### 힌트 (고민을 돕는 질문)

* **화살표 연산자의 차이:** PostgreSQL의 JSON 연산자에는 두 가지 종류가 있습니다.
* `->`: 결과를 **JSON 객체** 그대로 반환합니다. (예: `"Chrome"` - 따옴표 포함)
* `->>`: 결과를 **텍스트 (Text)** 로 변환하여 반환합니다. (예: `Chrome` - 따옴표 제거)


* **우리가 원하는 것:** 결과값이 따옴표가 없는 **깔끔한 문자열** 이어야 합니다. 어떤 연산자를 써야 할까요?

---

# 6. 부서별 직원 근태 유형 요약 보고서

---

### DB :

---

## 1. 테이블 정보 (Schema)

**1. `employees` (사원 정보)**

* `emp_id`: **사원 ID**
* `emp_name`: **사원명**
* `dept_name`: **부서명**

| emp_id | emp_name | dept_name |
| --- | --- | --- |
| 1 | 김철수 | 영업팀 |
| 2 | 이영희 | 인사팀 |
| 3 | 박민수 | 영업팀 |

**2. `attendance_logs` (일별 근태 기록)**

* `log_id`: **로그 ID**
* `emp_id`: **사원 ID**
* `work_date`: **날짜**
* `status`: **근태 상태** ('출근', '지각', '결근')

| log_id | emp_id | work_date | status |
| --- | --- | --- | --- |
| 101 | 1 | 2024-04-01 | 출근 |
| 102 | 1 | 2024-04-02 | 지각 |
| 103 | 2 | 2024-04-01 | 결근 |
| 104 | 2 | 2024-04-02 | 출근 |
| 105 | 3 | 2024-04-01 | 출근 |
| 106 | 3 | 2024-04-02 | 출근 |

---

## 2. 문제 상황 (Scenario)

**[인사팀장의 요청]**
"현재 근태 기록이 날짜별로 한 줄씩 쌓이고 (`attendance_logs`) 있어서, 직원별로 한 달 동안 근태가 어땠는지 한눈에 파악하기가 힘듭니다.

**직원별로 '출근', '지각', '결근' 횟수가 각각 몇 번인지** 옆으로 펼쳐서 보여주는 보고서를 만들어 주세요.
(결과 데이터는 사원 번호 순으로 정렬되어야 합니다.)"

---

## 3. 원하는 결과 (Expected Output)

| emp_name | dept_name | attend_count | late_count | absent_count |
| --- | --- | --- | --- | --- |
| 김철수 | 영업팀 | 1 | 1 | 0 |
| 이영희 | 인사팀 | 1 | 0 | 1 |
| 박민수 | 영업팀 | 2 | 0 | 0 |

---

#### 힌트 (고민을 돕는 질문)

* **Pivot (행을 열로 변환):** 세로로 길게 나열된 데이터(`status`)를 가로로 눕혀서 집계해야 합니다. 이 기술을 **Pivot** 또는 **Cross Tab** 이라고 합니다.
* **조건부 집계 (Conditional Aggregation):** `COUNT` 나 `SUM` 함수 안에서 `CASE WHEN` 문을 사용할 수 있다는 것을 기억하시나요?
* 예: `SUM(CASE WHEN status = '지각' THEN 1 ELSE 0 END)`


* **JOIN:** 두 테이블에 나누어진 정보(이름/부서, 근태 기록)를 합치기 위해 `JOIN` 이 필요합니다. 어떤 컬럼이 연결 고리일까요?

---

# 7. 구매 이력이 존재하는 실사용 고객 명단 추출

---

### DB :

---

## 1. 테이블 정보 (Schema)

**1. `customers` (고객 정보)**

* `customer_id`: **고객 ID** (PK)
* `name`: **고객 이름**

| customer_id | name |
| --- | --- |
| 1 | 철수 |
| 2 | 영희 |
| 3 | 민수 |

**2. `orders` (주문 내역)**

* `order_id`: **주문 ID**
* `customer_id`: **고객 ID** (FK)
* `amount`: **주문 금액**

| order_id | customer_id | amount |
| --- | --- | --- |
| 101 | 1 | 5000 |
| 102 | 1 | 3000 |
| 103 | 3 | 7000 |

*(영희(ID 2)는 주문 내역이 없습니다.)*

---

## 2. 문제 상황 (Scenario)

**[상황]**
마케팅 팀에서 **"한 번이라도 주문을 한 적이 있는 고객"** 의 명단만 뽑아달라고 합니다.
주문 내역 (`orders`) 테이블에 ID가 존재하는 고객만 `customers` 테이블에서 조회하면 됩니다.

**[요구 사항]**
`customers` 테이블을 조회하되, **`EXISTS`** 키워드를 사용하여 주문 내역이 있는 고객의 `customer_id` 와 `name` 을 출력하세요.

* **주의:** `JOIN` 을 사용하지 마세요. (JOIN을 쓰면 철수가 2번 출력될 수 있습니다. 우리는 중복 없는 명단만 필요합니다.)

---

## 3. 원하는 결과 (Expected Output)

| customer_id | name |
| --- | --- |
| 1 | 철수 |
| 3 | 민수 |

---

#### 힌트 (고민을 돕는 질문)

* **상관 서브쿼리 (Correlated Subquery):** `EXISTS` 내부의 서브쿼리에서 메인 쿼리(`customers`)의 데이터를 참조해야 합니다.
* "주문 테이블에 **이 고객(customers.id)** 의 주문 내역이 존재하는가?"를 확인하려면 `WHERE` 절을 어떻게 작성해야 할까요?


* **작동 원리:** `EXISTS` 는 서브쿼리의 결과가 **한 건이라도 발견되면** 즉시 참(True)을 반환하고 더 이상 찾지 않습니다. 그래서 단순히 "존재 여부"만 파악할 때 `JOIN` 이나 `IN` 보다 성능상 유리할 때가 많습니다.