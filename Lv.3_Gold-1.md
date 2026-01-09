# 1. 해외 지사별 매출 효율성 및 기여도 분석

---

### DB : MySQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`offices`** (지사 정보)

* `officeCode`: **지사 코드** (PK)
* `city`: **도시** (지사명으로 사용)
* `country`: **국가**

**`employees`** (직원 정보)

* `employeeNumber`: **사원 번호** (PK)
* `officeCode`: **소속 지사 코드** (FK)

**`customers`** (고객 정보)

* `customerNumber`: **고객 번호** (PK)
* `salesRepEmployeeNumber`: **담당 영업 사원 번호** (FK)

**`orders`** (주문 정보)

* `orderNumber`: **주문 번호** (PK)
* `customerNumber`: **고객 번호** (FK)
* `status`: **주문 상태**

**`orderdetails`** (주문 상세)

* `orderNumber`: **주문 번호** (FK)
* `quantityOrdered`: **주문 수량**
* `priceEach`: **개당 판매 가격**

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
각 지사 (**Office**) 가 얼마나 효율적으로 운영되고 있는지 평가하려고 합니다.
전 세계에 퍼져 있는 각 지사별로 **총 매출액** 을 집계하고, 해당 지사의 직원 수 대비 매출액인 **인당 매출 효율** (Sales per Employee) 을 계산해 주세요.

**[매출액 집계 기준]**

* `orderdetails` 테이블의 `quantityOrdered * priceEach` 를 매출액으로 정의합니다.
* `orders` 테이블의 `status` 가 **'Shipped'** 인 주문만 실적으로 인정합니다.
* 지사의 매출은 해당 지사에 소속된 **직원** (`employees`) 이 담당하는 **고객** (`customers`) 들이 일으킨 주문의 총합입니다.

**[직원 수 집계 기준]**

* 매출 발생 여부와 관계없이, 해당 지사 (`officeCode`) 에 소속된 **모든 직원의 수** 를 카운트합니다.

**[계산할 지표]**

* **City**: 지사가 위치한 도시 이름
* **Country**: 지사가 위치한 국가 이름
* **Employee Count**: 해당 지사의 총 직원 수
* **Total Sales**: 해당 지사의 총 매출액
* **Sales per Employee**: `Total Sales / Employee Count` 결과값

---

## 3. 원하는 결과 (Expected Output)

* 인당 매출 효율 (**Sales per Employee**) 이 높은 순서대로 내림차순 정렬하세요.
* 금액 관련 수치는 소수점 이하 2자리까지 표시하거나, `TRUNC` 등을 사용하여 보기 좋게 다듬어 주세요.

| city | country | emp_count | total_sales | sales_per_emp |
| --- | --- | --- | --- | --- |
| London | UK | 2 | 1,324,325.9 | 662,162.95 |
| Paris | France | 5 | 2,812,295.95 | 562,459.19 |
| NYC | USA | 2 | 1,072,619.47 | 536,309.74 |
| ... | ... | ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **뻥튀기 방지:** 단순하게 `JOIN` 을 하면 주문 상세 내역만큼 직원 수가 중복되어 카운트될 수 있습니다. `COUNT(DISTINCT employeeNumber)` 를 쓰면 해결될까요? 아니면 아예 지사별 직원 수를 구하는 쿼리를 따로 만드는 게 나을까요?
* **경로 연결:** `offices` → `employees` → `customers` → `orders` → `orderdetails` 까지 이어지는 긴 조인 고리를 잘 연결할 수 있나요?

---

# 2. 대륙별 물류 창고 재고 자산 가치 평가

---

### DB : Oracle Sample Database

---

## 1. 테이블 정보 (Schema)

**`regions`** (대륙/지역 정보)

* `region_id`: **지역 ID** (PK)
* `region_name`: **지역 이름** (예: Asia, Europe, Americas)

**`countries`** (국가 정보)

* `country_id`: **국가 ID** (PK)
* `region_id`: **소속 지역 ID** (FK)

**`locations`** (위치/주소 정보)

* `location_id`: **위치 ID** (PK)
* `country_id`: **소속 국가 ID** (FK)

**`warehouses`** (창고 정보)

* `warehouse_id`: **창고 ID** (PK)
* `warehouse_name`: **창고 이름**
* `location_id`: **창고 위치 ID** (FK)

**`inventories`** (재고 수량)

* `product_id`: **제품 ID** (PK, FK)
* `warehouse_id`: **창고 ID** (PK, FK)
* `quantity`: **현재 재고 수량**

**`products`** (제품 정보)

* `product_id`: **제품 ID** (PK)
* `list_price`: **제품 정가** (단가)

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
각 대륙 (**Region**) 별로 보유하고 있는 **재고의 총 자산 가치** 를 파악하라는 지시가 내려왔습니다.
물류 창고 (`warehouses`) 들이 위치한 대륙 (`regions`) 을 기준으로, 보관 중인 모든 제품의 **재고 총액** (Inventory Value) 을 산출해 주세요.

**[재고 자산 가치 계산]**

* 개별 재고의 가치는 `inventories.quantity * products.list_price` 로 계산합니다.
* 이를 대륙 (`region_name`) 단위로 모두 합산 (**SUM**) 해야 합니다.

**[테이블 연결 (Joins)]**

* `inventories` 에서 시작하여 `warehouses` → `locations` → `countries` → `regions` 까지 연결되는 긴 조인 경로를 완성해야 합니다.

**[계산할 지표]**

* **Region Name**: 대륙 이름
* **Warehouse Count**: 해당 대륙에 존재하는 창고의 수 (창고 이름 기준 중복 제거)
* **Total Inventory Value**: 해당 대륙의 총 재고 자산 가치

**[정렬 및 조건]**

* **Total Inventory Value** 가 높은 순서대로 내림차순 정렬하세요.
* 재고가 아예 없거나 자산 가치가 0인 대륙은 결과에서 제외하세요.

---

## 3. 원하는 결과 (Expected Output)

| Region Name | Warehouse Count | Total Inventory Value |
| --- | --- | --- |
| Americas | 6 | 73,466,624.94 |
| Asia | 3 | 33,727,659.38 |

---

#### 힌트 (고민을 돕는 질문)

* **조인 순서:** `regions` 테이블부터 시작해서 내려오는 게 편할까요, 아니면 `inventories` 부터 시작해서 올라가는 게 편할까요? (사실 결과는 같지만, 쿼리 작성 흐름을 잡아보세요.)
* **집계 대상:** 창고 개수를 셀 때 `COUNT(warehouse_id)` 와 `COUNT(DISTINCT warehouse_id)` 중 무엇이 적절할까요? `inventories` 테이블과 조인된 상태라면 창고 ID가 여러 번 등장할 수 있음을 유의하세요.

---

# 3. 우수 고객(VIP) 구매 패턴 기반 그룹핑

---

### DB : PostgreSQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`customer`** (고객 정보)

* `customer_id`: **고객 ID** (PK)
* `first_name`, `last_name`: **고객 이름**
* `email`: **이메일**

**`rental`** (대여 이력 - 빈도/Frequency 담당)

* `rental_id`: **대여 ID** (PK)
* `customer_id`: **고객 ID** (FK)
* `rental_date`: **대여 일자**

**`payment`** (결제 이력 - 금액/Monetary 담당)

* `payment_id`: **결제 ID** (PK)
* `customer_id`: **고객 ID** (FK)
* `amount`: **결제 금액**

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
고객의 구매 성향에 따라 **'등급'** (`Customer Tier`) 을 부여하고, 각 그룹의 특징을 파악하려 합니다.
모든 고객에 대해 **총 대여 횟수** (`Total Rentals`) 와 **총 결제 금액** (`Total Spent`) 을 집계한 뒤, 아래의 비즈니스 로직에 따라 고객 등급을 매겨주세요.

**[등급 부여 로직]** (CASE WHEN 사용)

1. **Elite**: 총 결제 금액이 **$150 이상** 이고, 총 대여 횟수가 **30회 이상** 인 고객
2. **High Spender**: 총 결제 금액은 **$150 이상** 이지만, 대여 횟수가 **30회 미만** 인 고객 (적게 빌리고 비싼 걸 보는 타입)
3. **Frequent User**: 총 결제 금액은 **$150 미만** 이지만, 대여 횟수가 **30회 이상** 인 고객 (많이 빌리지만 싼 걸 보는 타입)
4. **Standard**: 위 3가지 조건에 해당하지 않는 나머지 모든 고객

**[추가 분석 지표]**

* **Avg Price per Rental**: (`총 결제 금액 / 총 대여 횟수`). 소수점 둘째 자리까지 반올림하세요.

---

## 3. 원하는 결과 (Expected Output)

* **1순위**: `total_spent` 내림차순 (많이 쓴 순서)
* **2순위**: `total_rentals` 내림차순

| customer_id | first_name | last_name | total_rentals | total_spent | customer_tier | avg_price_per_rental |
| --- | --- | --- | --- | --- | --- | --- |
| 148 | Eleanor | Hunt | 46 | 211.55 | Elite | 4.6 |
| 526 | Karl | Seal | 45 | 208.58 | Elite | 4.64 |
| 178 | Marion | Snyder | 39 | 194.61 | Elite | 4.99 |
| ... | ... | ... | ... | ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **데이터 뻥튀기 방지 (Fan-out Trap):** `customer` 테이블에 `rental` 과 `payment` 를 동시에 `LEFT JOIN` 하면, 대여 횟수와 결제 금액이 서로 곱해져서 엄청나게 부풀려질 위험이 있습니다.
* 가장 안전한 방법은 각 테이블에서 미리 집계 (`GROUP BY`) 를 끝낸 후, 고객 ID를 기준으로 합치는 것입니다. (**CTE** 활용 권장)


* **조건문 순서:** `CASE WHEN` 은 위에서부터 순차적으로 조건을 검사합니다. 가장 까다로운 조건 (**Elite**) 을 어디에 두어야 할까요?

---

# 4. 조회 시나리오별 최적 인덱스 선정 전략

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`orders`** (주문 정보)

* **데이터 건수**: **약 1,000만 건**
* `order_id`: **주문 번호** (PK)
* `customer_id`: **고객 번호** (카디널리티: 약 10만 명)
* `order_date`: **주문 일자** (기간: 지난 5년치 데이터)
* `amount`: **주문 금액**

---

## 2. 문제 상황 (Scenario)

**[상황]**
특정 고객의 최근 주문 내역을 조회하는 쿼리가 시스템에서 가장 빈번하게 실행됩니다.
개발팀은 성능 최적화를 위해 **복합 인덱스** (Multi-column Index) 를 생성하려고 합니다.

**[타겟 쿼리]**

```sql
SELECT * FROM orders 
WHERE customer_id = 5821 
  AND order_date >= '2025-01-01';

```

*(참고: `customer_id` 는 특정 값 일치(Equality) 조건이고, `order_date` 는 범위(Range) 조건입니다.)*

**[선택지]**
개발팀 내에서 인덱스 컬럼 순서를 두고 두 가지 의견이 대립하고 있습니다.
**어떤 인덱스를 생성해야 위 쿼리에서 불필요한 스캔(I/O)을 최소화할 수 있을까요?**

* **후보 A:** `CREATE INDEX idx_A ON orders (order_date, customer_id);`
* **후보 B:** `CREATE INDEX idx_B ON orders (customer_id, order_date);`

**[문제]**

1. **후보 A** 와 **후보 B** 중 더 효율적인 인덱스를 선택하세요.
2. 선택하지 않은 인덱스는 왜 비효율적인지 **"정렬(Sorting)과 스캔 범위(Scan Range)"** 의 관점에서 설명해 보세요.

---

#### 힌트 (고민을 돕는 질문)

* **전화번호부** 를 상상해 보세요.
* **방식 A:** "생년월일"로 먼저 정렬하고, 같은 날짜에 태어난 사람끼리 "이름"으로 정렬된 전화번호부.
* **방식 B:** "이름"으로 먼저 정렬하고, 동명이인끼리 "생년월일"로 정렬된 전화번호부.


* 우리의 목표는 "이름이 김철수(`customer_id`)이고, 생일이 2000년 이후(`order_date >=`)인 사람"을 찾는 것입니다.
* **방식 A** 전화번호부에서 2000년 1월 1일 페이지를 펼쳤습니다. 거기서 '김철수'만 쏙쏙 골라낼 수 있나요? 아니면 2000년 이후 페이지를 전부 넘겨가며 '김철수'를 찾아야 하나요?
* **인덱스 컬럼 선정 공식:** "동등 비교(`=`) 조건 컬럼과 범위 비교(`> <`) 조건 컬럼 중 누가 앞에 와야 할까?"

---

# 5. 게시판 목록 로딩 지연 현상 해결

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`posts`** (게시판 게시글)

* **데이터 건수**: **약 5,000만 건** (대용량)
* `post_id`: **게시글 번호** (PK)
* `board_id`: **게시판 ID** (예: 1=자유게시판, 2=유머게시판...)
* `title`: **글 제목**
* `content`: **글 본문** (LONG TEXT)
* *특이사항:* 본문 내용이 매우 길어서, 게시글 하나 (Row) 의 물리적 크기가 상당히 큽니다.


* `writer_name`: **작성자 이름**
* `created_at`: **작성 일시**

---

## 2. 문제 상황 (Scenario)

**[상황]**
사용자가 가장 많이 방문하는 **'자유게시판 (board_id=1) 의 글 목록'** 페이지가 느리다는 신고가 접수되었습니다.
이 페이지는 글의 **제목** 과 **작성자** 만 리스트로 보여줍니다. (본문은 클릭해서 들어갔을 때만 보입니다.)

**[실행되는 쿼리]**

```sql
SELECT title, writer_name
FROM posts
WHERE board_id = 1
ORDER BY created_at DESC
LIMIT 10;

```

**[현재 인덱스 상태]**

* **인덱스 A:** `CREATE INDEX idx_standard ON posts (board_id, created_at);`

**[분석]**
현재 **인덱스 A** 는 `WHERE` 조건 (`board_id`) 과 `ORDER BY` (`created_at`) 를 모두 만족하는 완벽한 순서로 구성되어 있습니다.
하지만 DBA는 **"인덱스를 수정하면 성능이 2배 이상 빨라질 것"** 이라며 아래와 같은 **인덱스 B** 를 제안했습니다.

* **인덱스 B (제안):** `CREATE INDEX idx_covering ON posts (board_id, created_at, title, writer_name);`

**[문제]**

1. `title` 과 `writer_name` 은 `WHERE` 절이나 `ORDER BY` 절에 나오지도 않는데, 굳이 인덱스 뒤에 붙였습니다. **왜 인덱스 B가 인덱스 A보다 훨씬 빠를까요?**
2. 인덱스 A를 사용했을 때 발생하는 **"추가적인 물리적 I/O 작업 (Table Access)"** 이 무엇인지, 테이블 구조 (`content` 컬럼의 크기) 와 연관 지어 설명해 보세요.

---

#### 힌트 (고민을 돕는 질문)

* **인덱스 A의 실행 과정:**
1. 인덱스 트리에서 `board_id=1` 인 위치를 찾습니다.
2. `created_at` 순서대로 정렬된 리프 노드를 읽습니다.
3. 근데 우리가 필요한 건 `title` 과 `writer_name` 입니다. 이 정보가 **인덱스 안에 있나요?** 없다면 어디로 가야 하나요?


* **랜덤 I/O (Random I/O):**
* 인덱스에 없는 데이터를 가져오기 위해 실제 데이터 테이블 (Heap) 을 찾아가는 과정을 **"랜덤 액세스"** 라고 합니다.
* 만약 `content` 때문에 데이터 테이블의 덩치가 엄청나게 크다면, 이 방문 비용은 쌀까요, 비쌀까요?


* **인덱스 B의 실행 과정:**
* 인덱스 리프 노드에 `board_id`, `created_at` 뿐만 아니라 `title`, `writer_name` 도 같이 적혀 있다면? 실제 데이터 테이블을 방문할 필요가 있을까요?

---

# 6. 복합 조건 조회 시 발생하는 속도 저하 개선

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`users`** (회원 정보)

* **데이터 건수**: **약 1,000만 건**
* `user_id`: **회원 번호** (PK)
* `email`: **이메일** (**Unique Index** 있음: `idx_email`)
* `phone`: **휴대전화 번호** (**Index** 있음: `idx_phone`)
* `name`: **이름**

**[인덱스 현황]**

* `idx_email`: `(email)` 컬럼 하나만으로 구성됨
* `idx_phone`: `(phone)` 컬럼 하나만으로 구성됨

---

## 2. 문제 상황 (Scenario)

**[상황]**
로그인 화면에서 사용자가 **"이메일"** 을 입력할 수도 있고, **"휴대전화 번호"** 를 입력할 수도 있습니다.
그래서 백엔드 개발자가 입력된 값이 이메일인지 전화번호인지 확인하기 귀찮아서, 아래와 같이 **`OR` 조건** 을 사용하여 회원을 찾는 쿼리를 작성했습니다.

**[수정 전 쿼리 (Bad SQL)]**

```sql
SELECT *
FROM users
WHERE email = 'user@example.com'
   OR phone = '010-1234-5678';

```

**[문제점]**
각각의 컬럼에 인덱스가 있음에도 불구하고, 이 쿼리는 종종 **Full Table Scan** 이 발생하거나, **Index Merge** (두 인덱스를 억지로 합침) 가 발생하더라도 성능이 기대만큼 빠르지 않습니다. (특히 구버전 DB나 데이터 분포에 따라 성능이 불안정합니다.)

**[요구 사항]**
위 쿼리를 **물리적으로 완전히 분리된 2번의 인덱스 탐색** 을 수행하도록 수정하여, 항상 **안정적인 최상의 속도** 를 보장하게 만드세요.
(힌트: 하나의 쿼리 안에서 **집합 연산자 (Set Operator)** 를 사용하세요.)

---

#### 힌트 (고민을 돕는 질문)

* **OR 연산의 딜레마:** `WHERE A = 1 OR B = 2` 는 마치 **"서울 지도책에서 '강남구'를 찾거나, 부산 지도책에서 '해운대구'를 찾아라"** 라는 명령과 같습니다. 하나의 지도책(인덱스)으로는 해결이 안 됩니다.
* **해결책:** 가장 확실한 방법은 개발자가 명시적으로 **"각각 따로 찾아서 합치는 것"** 입니다.
1. "이메일로 먼저 찾으세요." (쿼리 1)
2. "전화번호로 따로 찾으세요." (쿼리 2)
3. "그리고 두 결과를 합치세요."


* SQL에서 두 쿼리의 결과를 **위아래로 합치는 (Merge)** 명령어는 무엇일까요? (`JOIN` 이 아닙니다!)

---

# 7. 댓글 조회 서비스의 시스템 CPU 부하 최적화

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`comments`** (댓글 정보)

* **데이터 건수**: **약 2,000만 건** (대용량)
* `comment_id`: **댓글 번호** (PK)
* `article_id`: **게시글 번호** (FK)
* `created_at`: **작성 일시**
* `content`: **댓글 내용**

**[현재 인덱스]**

* `CREATE INDEX idx_article ON comments (article_id);`
* `article_id` 하나에만 인덱스가 걸려 있습니다.

---

## 2. 문제 상황 (Scenario)

**[상황]**
인기 있는 게시글 (`article_id = 999`) 에는 댓글이 **10만 개** 나 달려 있습니다.
사용자가 게시글을 클릭하면 **"가장 최근 댓글 10개"** 만 보여주려고 합니다.

**[실행 쿼리]**

```sql
SELECT * FROM comments 
WHERE article_id = 999 
ORDER BY created_at DESC 
LIMIT 10;

```

**[성능 문제]**

* 이 쿼리를 실행했더니 DB 서버 CPU가 치솟고 시간이 **1초 이상** 걸렸습니다.
* 이유: `idx_article` 인덱스를 통해 10만 개의 댓글을 빠르게 찾았지만(Search), 그 10만 개를 메모리에 올려놓고 **날짜순으로 다시 정렬 (Sorting)** 하는 과정에서 엄청난 부하가 발생했기 때문입니다. (실행 계획에 `Using filesort` 라고 뜹니다.)

**[문제]**

* 별도의 정렬 작업 없이, **인덱스에서 데이터를 읽자마자 바로 상위 10개만 꺼내고 끝낼 수 있도록** 새로운 인덱스를 설계해 주세요.

---

#### 힌트 (고민을 돕는 질문)

* **인덱스는 이미 정렬되어 있다:** B-Tree 인덱스는 생성할 때 지정한 컬럼 순서대로 항상 **오름차순(또는 내림차순) 정렬** 된 상태를 유지합니다.
* **이상적인 구조:** 만약 인덱스 자체가 `article_id` 끼리 모여 있고, 그 안에서 `created_at` 순서로 이미 정렬되어 있다면, DB는 굳이 데이터를 꺼내서 다시 정렬할 필요가 있을까요?
* **해결책:** 일단 `article_id` 로 찾고, 그 안을 들여다보니 **이미 `created_at` 순서로 줄을 서 있는** 구조를 만들어야 합니다.
* **정답 인덱스 구성:** `CREATE INDEX idx_best ON comments ( ??? , ??? );`