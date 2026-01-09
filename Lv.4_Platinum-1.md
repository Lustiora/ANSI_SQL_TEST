# 1. 제품 라인별 고수익 프리미엄 상품군 식별

---

### DB : MySQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`products`** (제품 정보)

* `productCode`: **제품 코드** (PK)
* `productName`: **제품 이름**
* `productLine`: **제품 라인** (FK) - 카테고리 역할
* `buyPrice`: **제품 구입 가격**

**`productlines`** (제품 라인 정보)

* `productLine`: **제품 라인** (PK) - 카테고리 역할
* `textDescription`: **라인에 대한 상세 설명**

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
제품 라인 (**Product Line**) 별로 가격 경쟁력이 높은 제품을 분석하려고 합니다.
각 제품 라인 내에서 **'해당 라인의 평균 구입 가격 (buyPrice)'보다 비싼 제품들** 만 추출하여 리스트를 만들어 주세요.

문제를 해결하기 위해 쿼리를 단계별로 나누어 작성해야 하며, **중첩 CTE** (**Nested CTE**, 또는 **Chained CTE**) 구조를 사용해야 합니다.

**[정렬 기준]**

* `productLine` 기준으로 오름차순 정렬하고, 같은 라인 내에서는 `buyPrice` 가 비싼 순서대로 내림차순 정렬하세요.

---

## 3. 원하는 결과 (Expected Output)

* 결과는 3행까지만 표시합니다.

| productline | productname | textdescription | buyprice |
| --- | --- | --- | --- |
| Classic Cars | 1952 Alpine Renault 1300 | Attention car enthusiasts: Make your wildest car ownership | 98.58 |
| Classic Cars | 1952 Citroen-15CV | Attention car enthusiasts: Make your wildest car ownership | 72.82 |
| Classic Cars | 1956 Porsche 356A Coupe | Attention car enthusiasts: Make your wildest car ownership | 98.3 |
| ... | ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **CTE의 연쇄 작용:** 중첩 CTE란, `WITH` 절 안에서 정의된 첫 번째 가상 테이블을 두 번째 가상 테이블이 참조하여 사용하는 방식을 말합니다. 이 방식을 사용하면 복잡한 서브쿼리를 여러 번 중첩하는 것보다 가독성이 훨씬 좋아집니다.
* **문법 구조:** CTE를 여러 개 정의할 때 `WITH` 키워드는 맨 처음에 한 번만 쓰고, 각 CTE 사이는 **쉼표(,)** 로 구분한다는 점을 기억하시나요?
* 예: `WITH CTE1 AS (...), CTE2 AS (...) SELECT ...`


* **비교 조건:** 두 번째 CTE나 메인 쿼리에서 조인을 할 때, `products.buyPrice > LineAverage.avg_price` 와 같은 형태의 조건절이 필요합니다.

---

# 2. VIP 고객 세그먼트별 구매 성향 심층 분석

---

### DB : MySQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`customers`** (고객 정보)

* `customerNumber`: **고객 번호** (PK)
* `customerName`: **고객(회사)명**

**`orders`** (주문 정보)

* `orderNumber`: **주문 번호** (PK)
* `customerNumber`: **고객 번호** (FK)
* `orderDate`: **주문 일자**
* `status`: **주문 상태** (예: 'Shipped', 'Cancelled')

**`orderdetails`** (주문 상세)

* `orderNumber`: **주문 번호** (PK, FK)
* `productCode`: **제품 코드** (PK, FK)
* `quantityOrdered`: **주문 수량**
* `priceEach`: **판매 단가**

**`products`** (제품 정보)

* `productCode`: **제품 코드** (PK)
* `productLine`: **제품 라인** (FK) - 카테고리

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
2003년 매출을 견인한 **VIP 고객들** 을 식별하고, 그들의 **구매 다양성** 을 분석하려 합니다.

* **2003년** 에 주문을 완료한 고객들을 대상으로 **총 주문 금액** 을 계산하고, **'전체 평균 주문 금액보다 더 많이 지출한 VIP 고객'** 들을 선별해 주세요.
* 그리고 선별된 VIP 고객들이 **총 몇 종류 (Product Line) 의 제품군** 을 구매했는지 집계해야 합니다.

**[정렬 기준]**

* 총 지출액 (`total_spent`) 이 높은 순서대로 내림차순 정렬하세요.

---

## 3. 원하는 결과 (Expected Output)

* 결과는 3행까지만 표시합니다.

| customername | total_spent | distinct_lines |
| --- | --- | --- |
| 141 | 189,840.15 | 7 |
| 124 | 167,783.08 | 6 |
| 148 | 150,123.15 | 6 |
| ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **집계의 집계 (Average of Sums):** 이 문제의 핵심은 "평균"을 구하는 기준입니다. '주문 건당 평균'이 아니라, **'고객별 총지출액의 평균'** 을 구해야 합니다.
* 따라서 첫 번째 CTE에서 고객별 총액을 먼저 구하고, 두 번째 CTE에서 `SELECT AVG(total_spent) FROM CustomerSpending` 과 같은 형태로 비교해야 합니다.


* **평균 비교:** `WHERE` 절에서 집계 함수 (`AVG`) 를 바로 쓸 수 없다는 것을 아시나요? 조건절에서 `WHERE total_spent > (SELECT AVG(...) FROM ...)` 형식을 사용해야 합니다.
* **다양성 집계:** "몇 종류의 제품 라인"을 샀는지 셀 때는 단순히 `COUNT(productLine)` 을 하면 중복이 포함됩니다. 중복을 제거하고 세려면 어떤 키워드가 필요할까요? (`DISTINCT`)

---

# 3. 옵티마이저 실행 계획 이상(Anomaly) 점검

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`orders`** (주문 정보)

* **데이터 건수**: **약 1,000만 건**
* `order_id`: **주문 번호** (PK)
* `order_status`: **주문 상태** (VARCHAR)
* **데이터 분포 (Data Distribution):**
* `'COMPLETED'` (배송 완료): **990만 건 (99%)**
* `'REFUNDED'` (환불됨): **10만 건 (1%)**


* `amount`: **주문 금액**

**[인덱스 정보]**

* `CREATE INDEX idx_status ON orders (order_status);`
* `order_status` 컬럼 하나에 대해 단일 B-Tree 인덱스가 생성되어 있습니다.

---

## 2. 문제 상황 (Scenario)

**[상황]**
개발자가 두 가지 쿼리를 실행해보고 실행 계획 (Explain Plan) 을 확인했습니다.
그런데 **동일한 인덱스 (`idx_status`)** 가 있음에도 불구하고, DB 옵티마이저가 **서로 다른 선택** 을 했습니다.

**[쿼리 A: 환불 내역 조회]**

```sql
SELECT * FROM orders WHERE order_status = 'REFUNDED';

```

* **실행 결과:** 인덱스를 사용함 (**Index Range Scan**)
* **속도:** 매우 빠름 (0.05초)

**[쿼리 B: 완료 내역 조회]**

```sql
SELECT * FROM orders WHERE order_status = 'COMPLETED';

```

* **실행 결과:** 인덱스를 **사용하지 않음** (**Full Table Scan**)
* **속도:** 느림 (3초)

**[문제]**

1. DB는 왜 **쿼리 B** 에서 멀쩡한 인덱스를 무시하고 **테이블 전체 스캔 (Full Table Scan)** 을 선택했을까요? (단순히 "데이터가 많아서"라고 답하기보다, **I/O 방식의 차이** 를 언급해 주세요.)
2. 이 현상을 설명하는 데이터베이스 용어인 **'이것'** 은 무엇일까요? (힌트: 전체 데이터 중 특정 조건에 해당하는 데이터의 비율)

---

#### 힌트 (고민을 돕는 질문)

* **랜덤 액세스 (Random Access) vs 순차 액세스 (Sequential Access):**
* **인덱스 스캔:** 인덱스에서 주소를 찾은 뒤, 데이터 블록을 하나씩 **띄엄띄엄 (Random)** 찾아갑니다.
* **풀 스캔:** 데이터 블록을 처음부터 끝까지 **쭉 (Sequential)** 읽어버립니다.
* 만약 여러분이 책에서 **특정 단어 하나** 를 찾는다면 색인(Index)을 보는 게 빠르겠지만, 책 내용의 **99%** 를 형광펜으로 칠해야 한다면? 색인을 왔다 갔다 하는 게 빠를까요, 아니면 그냥 첫 페이지부터 쭉 칠하는 게 빠를까요?


* **손익분기점 (Tipping Point):** 보통 DB 옵티마이저는 전체 데이터의 **몇 퍼센트 (%)** 이상을 읽어야 할 때 인덱스가 더 손해라고 판단할까요? (보통 20~30% 정도입니다.)

---

# 4. 대용량 과거 로그 데이터 조회 속도 개선

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`logs`** (시스템 로그)

* **데이터 건수**: **약 1억 건** (초대용량)
* `log_id`: **로그 ID** (PK, Auto Increment)
* `created_at`: **생성 일시** (인덱스 있음: `idx_created_at`)
* `message`: **로그 내용**

---

## 2. 문제 상황 (Scenario)

**[상황]**
관리자 페이지에서 로그 목록을 최신순으로 보여줍니다.
1페이지 (`LIMIT 10`) 는 방금 배우신 인덱스 덕분에 아주 빠릅니다.
그런데 관리자가 **"아주 옛날 로그를 보겠다"** 며 게시판 하단의 **[10,000페이지]** 버튼을 클릭했습니다.
시스템이 멈춘 듯이 느려지더니 10초 뒤에야 결과가 떴습니다.

**[실행된 쿼리]**

```sql
SELECT * FROM logs
ORDER BY created_at DESC
LIMIT 100000, 10; -- 10만 번째부터 10개를 가져와라

```

**[문제]**

1. `created_at` 에 인덱스가 잘 걸려 있어서 정렬도 필요 없는데, **왜 `OFFSET` 이 커질수록 (뒷 페이지로 갈수록) 쿼리가 미친 듯이 느려질까요?** (DB가 내부적으로 데이터를 읽는 과정을 묘사해 주세요.)
2. 이 문제를 해결하기 위해, **`OFFSET` 을 사용하지 않는 방식 (No-Offset)** 으로 쿼리를 튜닝하려고 합니다. 직전 페이지의 마지막 `created_at` 값이 `'2024-01-01 12:00:00'` 이었다면, 다음 페이지를 **0.01초 만에** 로딩하기 위해 `WHERE` 절을 어떻게 구성해야 할까요?

---

#### 힌트 (고민을 돕는 질문)

* **OFFSET의 배신:**
* `LIMIT 100000, 10` 은 "10만 번째 위치로 뿅! 이동해서 10개 읽어라"가 **아닙니다.**
* 인덱스는 "몇 번째"라는 위치 정보를 가지고 있지 않습니다. (배열이 아니라 링크드 리스트와 비슷합니다.)
* 그렇다면 DB는 10만 번째 데이터를 찾기 위해 1번째부터 99,999번째까지를 어떻게 해야 할까요?


* **No-Offset 전략:**
* "앞에서부터 10만 개 건너뛰어!"라고 시키는 대신, **"아까 본 마지막 놈 (`2024-01-01...`) 보다 더 과거인 놈들 중에서 10개만 줘!"** 라고 시키면 어떨까요?
* 인덱스 탐색 관점에서 이 둘의 차이는 무엇일까요?

---

# 5. 영업 사원별 주력 판매 카테고리 분석

---

### DB : Oracle Sample Database

---

## 1. 테이블 정보 (Schema)

**`employees`** (직원 정보)

* `employee_id`: **사원 번호** (PK)
* `first_name`: **이름**
* `last_name`: **성**
* `job_title`: **직급**

**`orders`** (주문 정보)

* `order_id`: **주문 번호** (PK)
* `customer_id`: **고객 번호** (FK)
* `salesman_id`: **해당 주문을 담당한 영업 사원** (FK) - `employee_id` 참조
* `status`: **주문 상태** (예: 'Shipped', 'Pending', 'Canceled')
* `order_date`: **주문 일자**

**`order_items`** (주문 상세)

* `order_id`: **주문 번호** (PK, FK)
* `item_id`: **아이템 순번** (PK)
* `product_id`: **제품 번호** (FK)
* `quantity`: **주문 수량**
* `unit_price`: **판매 단가**

**`products`** (제품 정보)

* `product_id`: **제품 번호** (PK)
* `product_name`: **제품명**
* `category_id`: **카테고리 번호** (FK)

**`product_categories`** (카테고리 정보)

* `category_id`: **카테고리 번호** (PK)
* `category_name`: **카테고리 이름** (예: 'CPU', 'Video Card')

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
영업 사원 (**Salesman**) 들의 판매 성향을 분석하기 위한 보고서를 작성하세요.
각 영업 사원별로 **가장 많은 매출 (Revenue)** 을 올린 **제품 카테고리 (Category)** 가 무엇인지 파악해야 합니다.

**[매출 집계 기준]**

* `orders` 테이블의 `status` 가 **'Shipped'** 인 주문만 집계에 포함하세요.
* 매출액 (**Revenue**) 은 `order_items` 테이블의 `quantity * unit_price` 로 계산합니다.

**[영업 사원별 1위 선정]**

* 각 영업 사원 (`salesman_id`) 별로 모든 카테고리에 대한 총 매출액을 구한 뒤, **매출액이 가장 높은 단 하나의 카테고리** 만 출력해야 합니다.
* 만약 매출액 1위 카테고리가 여러 개일 경우, **카테고리 이름의 알파벳 순서** 가 빠른 것을 우선으로 합니다.

**[제외 조건]**

* 주문 실적이 아예 없는 직원은 결과에서 제외합니다.

---

## 3. 원하는 결과 (Expected Output)

* 결과는 3행까지만 표시합니다.

| first_name | last_name | top_category_name | category_revenue |
| --- | --- | --- | --- |
| Chloe | Cruz | CPU | 1,479,691.16 |
| Daisy | Ortiz | Video Card | 213,601.66 |
| Evie | Harrison | Video Card | 367,198.64 |
| ... | ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **순위 지정 및 필터링:** 단순히 `GROUP BY` 만 해서는 사원별 '최대 매출'을 가진 행만 뽑아내기 어렵습니다. 사원별로 카테고리들의 매출 순위를 매긴 후, 랭킹이 1위인 행만 필터링하는 전략이 필요합니다.
* **랭킹 매기기:** 그룹 (Salesman) 내에서 매출액 순으로 순번을 붙여주는 윈도우 함수 `ROW_NUMBER()` 또는 `RANK()` 를 어떻게 활용할 수 있을까요?
* **계산의 순서:** 먼저 "사원-카테고리별 총 매출"을 구하는 쿼리를 만들고 (서브쿼리 혹은 CTE), 그 결과 위에서 다시 **"사원별 1등"** 을 뽑는 2단계 구조로 생각해보세요.

---

# 6. 월간 매출 성장률(MoM) 지표 산출

---

### DB : MySQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`orders`** (주문 정보)

* `orderNumber`: **주문 번호** (PK)
* `orderDate`: **주문 일자**
* `status`: **주문 상태**

**`orderdetails`** (주문 상세)

* `orderNumber`: **주문 번호** (FK)
* `quantityOrdered`: **주문 수량**
* `priceEach`: **판매 단가**

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
회사의 매출이 매달 어떻게 변화하고 있는지 **성장세** 를 추적하려고 합니다.
**2004년도** 의 데이터를 기준으로, 매월 매출액을 집계하고 지난달 대비 매출이 **몇 퍼센트 (%) 성장했는지** (Growth Rate) 계산해 주세요.

**[매출 집계]**

* `orders` 의 `status` 가 **'Shipped'** 인 주문만 포함합니다.
* 매출액은 `quantityOrdered * priceEach` 의 합계입니다.
* 2004년 데이터 (`orderDate` 기준 `2004-01-01` ~ `2004-12-31`) 만 추출하세요.

**[전월 매출 비교 (Previous Month Sales)]**

* 현재 달의 매출뿐만 아니라, **바로 직전 달 (Previous Month)** 의 매출을 옆에 나란히 표시해야 합니다.
* 이를 위해 윈도우 함수 `LAG()` 를 사용하는 것이 핵심입니다.

**[성장률 계산 (Growth Rate)]**

* 공식: `((당월 매출 - 전월 매출) / 전월 매출) * 100`
* **소수점 둘째 자리** 까지 반올림하여 표시해 주세요. (예: 15.25%)
* 전월 데이터가 없는 (2004년 1월) 경우는 성장률을 `0.00%` 혹은 `NULL` 로 표시해도 무방합니다.

---

## 3. 원하는 결과 (Expected Output)

* 결과는 3행까지만 표시합니다.

| order_month | current_sales | prev_sales | growth_rate |
| --- | --- | --- | --- |
| 2004-01 | 292,385.21 | [NULL] | [NULL] |
| 2004-02 | 289,502.84 | 292,385.21 | -0.99% |
| 2004-03 | 217,691.26 | 289,502.84 | -24.81% |
| ... | ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **2단계 전략:** `GROUP BY` 로 월별 매출을 먼저 구한 뒤, 그 결과를 바탕으로 `LAG()` 를 써야 합니다. 즉, 서브쿼리 (혹은 CTE) 로 **'월별 매출'** 을 먼저 만들고, 메인 쿼리에서 성장률을 계산하는 전략이 필요합니다.
* **날짜 자르기:** `orderDate` 에서 '년-월' 포맷을 추출하거나, 1일로 맞추려면 `TO_CHAR(orderDate, 'YYYY-MM')` 또는 `DATE_TRUNC('month', orderDate)` 를 사용하면 편리합니다.
* **LAG 사용법:** `LAG(컬럼명) OVER (ORDER BY 정렬기준)` 문법을 기억하시나요? 여기서는 **'월 순서'** 대로 정렬해야겠죠?

---

# 7. 라인별 재고 부족 위험군 조기 탐지

---

### DB : MySQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`products`** (제품 정보)

* `productCode`: **제품 코드** (PK)
* `productName`: **제품 이름**
* `productLine`: **제품 라인** (FK) - 카테고리
* `quantityInStock`: **현재 재고 수량**
* `buyPrice`: **제품 구입 가격**

**`productlines`** (제품 라인 정보)

* `productLine`: **제품 라인** (PK) - 카테고리
* `textDescription`: **라인에 대한 상세 설명**

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
재고 관리를 위해 각 제품 라인별로 재고가 가장 부족한 **'비상 제품'** 들을 파악하려고 합니다.

* 각 제품 라인 (`productLine`) 별로 **재고 수량** (`quantityInStock`) 이 가장 적은 **상위 3개 제품** 을 추출하고, 해당 제품들의 **재고 총 가치** (`quantityInStock * buyPrice`) 를 계산해 주세요.
* 이 문제는 논리적 단계를 명확히 하기 위해 **중첩 CTE (Nested CTE)** 를 사용하여 작성해야 합니다.

**[정렬 기준]**

* 결과는 `productLine` 오름차순, 그리고 **재고 순위 (Ranking)** 오름차순으로 정렬하세요.

---

## 3. 원하는 결과 (Expected Output)

* 결과는 3행까지만 표시합니다.

| productline | productname | quantityinstock | inventory_value | textdescription |
| --- | --- | --- | --- | --- |
| Classic Cars | 1968 Ford Mustang | 68 | 6,483.12 | Attention car enthusiasts: Make... |
| Classic Cars | 1970 Chevy Chevelle SS 454 | 1,005 | 49,486.2 | Attention car enthusiasts: Make... |
| Classic Cars | 1969 Ford Falcon | 1,049 | 87,119.45 | Attention car enthusiasts: Make... |
| ... | ... | ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **윈도우 함수와 필터링의 시점:** SQL 표준 문법상 `WHERE` 절에서는 윈도우 함수 (`RANK`, `ROW_NUMBER` 등) 를 직접 사용할 수 없습니다. (예: `WHERE RANK() OVER(...) <= 3` 은 불가능)
* 따라서, **첫 번째 CTE** 에서 먼저 순위를 '컬럼'으로 만들어두고, **두 번째 CTE** 에서 그 컬럼을 기준으로 `WHERE rank_num <= 3` 처럼 필터링하는 것이 핵심 로직입니다.


* **순위 매기기:** "각 라인별로" 순위를 매기려면 `OVER` 절 안에서 `PARTITION BY` 를 써야 할까요? 아니면 `GROUP BY` 를 써야 할까요?
* **중첩의 이점:** 만약 CTE를 쓰지 않고 서브쿼리로만 작성한다면 괄호가 깊어져 가독성이 떨어집니다. CTE를 사용하여 논리의 흐름을 단계별로 표현해 보세요.