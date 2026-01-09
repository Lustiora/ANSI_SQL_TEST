# 1. 전체 조직 구조의 계층(Depth) 측정

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`employees`** (직원 조직도)

* `emp_id`: **사번** (PK)
* `name`: **이름**
* `manager_id`: **직속 상사의 사번** (FK)
* *CEO는 상사가 없으므로 `NULL` 입니다.*

| emp_id | name | manager_id |
| --- | --- | --- |
| 1 | King | NULL |
| 2 | Scott | 1 |
| 3 | Adams | 2 |

---

## 2. 문제 상황 (Scenario)

**[상황]**
King(CEO)부터 시작해서, 그 밑에 누가 있고, 또 그 밑에 누가 있는지 **조직의 계층 (Level)** 을 파악하고 싶습니다.

**[요구 사항]**
**`WITH RECURSIVE`** 구문을 사용하여 아래와 같은 결과를 출력해 주세요.

1. `name`: **직원 이름**
2. `lvl`: **조직 계층** (CEO=1, 팀장=2, 사원=3 ...)

---

## 3. 원하는 결과 (Expected Output)

| name | lvl |
| --- | --- |
| King | 1 |
| Scott | 2 |
| Adams | 3 |

---

#### 힌트 (고민을 돕는 질문)

* **재귀 쿼리의 공식 (Template):** 재귀 쿼리는 문법이 생소하므로 **기본 틀 (Skeleton)** 을 드립니다. 빈칸을 채워보세요.
1. **Anchor (닻):** 시작점(CEO)을 먼저 찾습니다. (Level 1)
2. **UNION ALL:** 위 결과와 아래 결과를 합칩니다.
3. **Recursive (재귀):** **"방금 찾은 사람(`cte`)의 부하직원(`emp`)"** 을 찾아서 Level을 +1 합니다.



```sql
WITH RECURSIVE OrgChart AS (
    -- 1. 시작점 (Anchor): 상사가 없는 사람(CEO) 찾기, 레벨은 1
    SELECT
        emp_id,
        name,
        1 as lvl
    FROM employees
    WHERE manager_id IS ______

    UNION ALL

    -- 2. 재귀 (Recursive): 윗사람(OrgChart)과 아랫사람(employees)을 조인
    SELECT
        e.emp_id,
        e.name,
        o.lvl + 1  -- 부모 레벨에 1을 더함
    FROM employees e
    JOIN OrgChart o ON e.______ = o.______ -- 자식의 매니저ID = 부모의 ID
)
SELECT name, lvl FROM OrgChart;

```

---
# 2. 조직 계층을 반영한 팀별 누적 실적 집계

---

### DB : MySQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`employees`** (직원 정보)

* `employeeNumber`: **사원 번호** (PK)
* `lastName`, `firstName`: **성, 이름**
* `jobTitle`: **직급**
* `reportsTo`: **직속 상사의 사원 번호** (FK, 자기 참조, NULL이면 최상위 관리자)

**`customers`** (고객 정보)

* `customerNumber`: **고객 번호** (PK)
* `salesRepEmployeeNumber`: **담당 영업 사원 번호** (FK)

**`orders`** (주문 정보)

* `orderNumber`: **주문 번호** (PK)
* `customerNumber`: **고객 번호** (FK)
* `orderDate`: **주문 일자**
* `status`: **주문 상태**

**`orderdetails`** (주문 상세)

* `orderNumber`: **주문 번호** (PK, FK)
* `quantityOrdered`: **주문 수량**
* `priceEach`: **판매 단가**

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
회사의 모든 관리자 (**Manager**) 가 자신을 포함한 **하위 조직 전체** 에서 발생시킨 **총매출** 을 파악하고 싶습니다.
단순히 본인의 영업 실적뿐만 아니라, 자신에게 보고하는 모든 부하 직원 (직계 및 그 하위의 모든 직원) 들의 실적까지 합산하여 **'팀 총괄 매출 (Total Team Revenue)'** 을 계산하는 보고서를 작성하세요.

**[1. 개별 실적 계산 (Base Sales)]**

* **2004년** 한 해 동안 각 직원이 담당하는 고객들로부터 발생한 매출액 (`quantityOrdered * priceEach`) 의 합계를 먼저 구합니다.
* 제외 조건: `status` 가 **'Cancelled'** 인 주문은 제외합니다.

**[2. 조직도 구성 (Recursive Structure)]**

* `WITH RECURSIVE` 를 사용하여 최상위 관리자 (`reportsTo` 가 NULL) 부터 말단 직원까지 내려가는 계층 구조를 만듭니다.
* **Level**: 최상위 사장님을 0 또는 1로 시작하여 아래로 내려갈수록 1씩 증가합니다.
* **HierarchyPath**: 상위 관리자부터 해당 직원까지의 이름을 연결한 경로 문자열을 만듭니다. (예: `Diane -> Mary -> Jeff`)

**[3. 매출 누적 (Revenue Roll-up)]**

* 각 직원에 대해 **(본인 실적 + 모든 하위 조직원들의 실적 합계)** 를 계산하여 `Total_Team_Revenue` 를 구하세요.
* 예를 들어, 사장님 (President) 의 `Total_Team_Revenue` 는 회사 전체의 매출 합계와 같아야 합니다.

---

## 3. 원하는 결과 (Expected Output)

* 결과는 3행까지만 표시합니다.

| employeenumber | firstname | jobtitle | hierarchypath | level | individual_revenue | total_team_revenue |
| --- | --- | --- | --- | --- | --- | --- |
| 1,002 | Diane | President | Diane | 0 | 0 | 4,344,182 |
| 1,076 | Jeff | VP Marketing | Diane -> Jeff | 1 | 0 | 0 |
| 1,056 | Mary | VP Sales | Diane -> Mary | 1 | 0 | 4,344,182 |
| ... | ... | ... | ... | ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **데이터 타입 일치:** `HierarchyPath` 를 만들 때 `CAST(firstName AS VARCHAR(1000))` 처럼 데이터 타입을 명시하지 않으면, 재귀 과정에서 텍스트 길이가 달라져 에러가 발생할 수 있습니다.
* **집계 전략:** 재귀 CTE 안에서 무리하게 합계를 구하려고 하면 복잡해집니다.
1. `CTE_Sales`: 직원별 2004년 개별 매출을 미리 구한다.
2. `CTE_Org`: 재귀를 이용해 모든 직원의 계층 정보와 경로를 구한다.
3. **최종 조회:** `CTE_Org` 를 (A)와 (B)로 두 번 사용하여, "A(상사)의 경로에 B(부하)의 경로가 포함되는 경우"를 찾아 조인하고 `SUM` 을 하면 어떨까요? (혹은 모든 자손 관계를 리스트업 한 뒤 Join)

---

# 3. 관리자별 관할 지역 시장 점유율 산출

---

### DB : MySQL Sample Database

---

## 1. 테이블 정보 (Schema)

**`employees`** (직원 정보)

* `employeeNumber`: **사원 번호** (PK)
* `reportsTo`: **직속 상사 번호** (FK)
* `firstName`: **이름**
* `lastName`: **성**
* `jobTitle`: **직급**

**`customers`** (고객 정보)

* `customerNumber`: **고객 번호** (PK)
* `salesRepEmployeeNumber`: **담당 영업 사원 번호** (FK)
* `country`: **고객 소재 국가**

**`orders`** (주문 정보)

* `orderNumber`: **주문 번호** (PK)
* `customerNumber`: **고객 번호** (FK)
* `orderDate`: **주문 일자**
* `status`: **주문 상태**

---

## 2. 문제 상황 (Scenario)

**[문제 사양]**
각 매니저가 이끄는 조직이 전 세계적으로 얼마나 넓은 시장을 커버하고 있는지 분석하려 합니다.
본인을 포함하여 자신의 **모든 하위 조직원 (부하 직원의 부하 직원 포함)** 이 담당하는 **'고유한 (Unique) 고객'** 과 **'진출 국가'** 를 집계해 주세요.

**[1. 조직도 구성 (Recursive Structure)]**

* `WITH RECURSIVE` 를 사용하여 `employees` 테이블의 계층 구조를 전개합니다.
* **HierarchyPath**: 최상위 관리자부터 해당 직원까지의 이름을 연결한 경로를 생성합니다. (예: `Diane -> Mary -> Gerard`)

**[2. 고객 및 국가 정보 결합 (Active Client Filtering)]**

* 분석 대상 고객은 **2004년 1월 1일 ~ 2005년 12월 31일** 사이에 주문 (`orders`) 실적이 있는 고객으로 한정합니다.
* 제외 조건: 주문 상태 (`status`) 가 **'Cancelled'** 인 건은 실적으로 인정하지 않습니다.
* 위 조건을 만족하는 고객들을 담당 영업 사원 (`salesRepEmployeeNumber`) 과 연결해야 합니다.

**[3. 글로벌 영향력 집계 (Aggregation)]**

* 모든 직원(매니저 포함)에 대해 다음 지표를 계산하세요. (**하위 조직 전체 포함**)
* **Total_Unique_Customers**: 조직이 관리하는 중복 없는 총 고객 수.
* **Total_Served_Countries**: 조직이 진출해 있는 중복 없는 총 국가 (Country) 수.
* **Territory_Reach**: 조직이 담당하는 국가들의 이름을 알파벳 순으로 정렬하여 **쉼표(,)** 로 구분된 하나의 문자열로 나열하세요. (예: `France, Japan, USA`)



---

## 3. 원하는 결과 (Expected Output)

* 결과는 3행까지만 표시합니다.

| employeenumber | firstname | jobtitle | reportsto | hierarchypath | total_unique_customers | total_served_countries | territory_reach |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1,002 | Diane | President | [NULL] | Diane | 97 | 21 | Australia, Austria, Belgium... |
| 1,056 | Mary | VP Sales | 1,002 | Diane -> Mary | 97 | 21 | Australia, Austria, Belgium... |
| 1,102 | Gerard | Sale Manager (EMEA) | 1,056 | Diane -> Mary -> Gerard | 45 | 14 | Austria, Belgium, Denmark... |
| ... | ... | ... | ... | ... | ... | ... | ... |

---

#### 힌트 (고민을 돕는 질문)

* **조인 전략:** "나(Manager)"를 기준으로 집계하려면, 재귀 쿼리 결과에서 **"나의 HierarchyPath로 시작하는 모든 하위 직원(Subordinates)"** 들을 찾아 조인(Self-Join)하는 것이 가장 직관적입니다. (`LIKE path || '%'`)
* **중복 제거:** A 사원이 'USA'를 담당하고, 부하 직원 B도 'USA'를 담당한다면, 매니저 입장에서 진출 국가는 1개여야 합니다. `COUNT(country)` 와 `COUNT(DISTINCT country)` 의 차이를 기억하시죠?
* **문자열 합치기:** 여러 행에 흩어진 국가 이름들을 한 줄로 합치기 위해 PostgreSQL의 `STRING_AGG(DISTINCT column, separator)` 함수가 유용합니다.

---

# 4. 쇼핑몰 카테고리의 최상위 카테고리(Root) 찾기

---

### DB :

---

## 1. 테이블 정보 (Schema)

**`categories`** (상품 카테고리 구조)

* `category_id`: **카테고리 ID**
* `category_name`: **카테고리명**
* `parent_id`: **상위 카테고리 ID** (NULL이면 최상위)

| category_id | category_name | parent_id |
| --- | --- | --- |
| 1 | Electronics | NULL |
| 2 | Laptops | 1 |
| 3 | Gaming Laptops | 2 |
| 4 | Clothing | NULL |
| 5 | T-Shirts | 4 |

*(구조: Electronics > Laptops > Gaming Laptops)*

---

## 2. 문제 상황 (Scenario)

**[개발팀장의 요청]**
"현재 상품 페이지에 **'Gaming Laptops'** 라고만 표시되는데, 고객들이 헷갈려합니다.
이 카테고리가 원래 어디 소속인지 **최상위 (Root) 카테고리명** 을 같이 보여주고 싶습니다.

모든 카테고리에 대해, 자신의 이름과 **자신의 뿌리가 되는 최상위 카테고리 이름** 을 찾아주세요."

---

## 3. 원하는 결과 (Expected Output)

* `Gaming Laptops` 의 부모는 `Laptops` (2), 그 부모는 `Electronics` (1)입니다. 1번은 부모가 없으므로 `Electronics` 가 Root입니다.
* 모든 카테고리에 대해 아래와 같이 출력하세요.

| category_name | root_category_name |
| --- | --- |
| Electronics | Electronics |
| Laptops | Electronics |
| Gaming Laptops | Electronics |
| Clothing | Clothing |
| T-Shirts | Clothing |

---

#### 힌트 (고민을 돕는 질문)### DB :

---

## 1. 테이블 정보 (Schema)

**1. `user_logs` (유저 접속 정보)**

* `user_id`: **회원 ID**
* `info`: **접속 환경 정보** (JSON)
* 예: `{"browser": "Chrome", "os": "Windows"}`

| user_id | info |
| --- | --- |
| 1 | `{"browser": "Chrome", "os": "Mac"}` |
| 2 | `{"browser": "Safari", "os": "iOS"}` |
| 3 | `{"browser": "Chrome", "os": "Windows"}` |

**2. `orders` (주문 내역)**

* `order_id`: **주문 번호**
* `user_id`: **회원 ID**
* `order_date`: **주문 일자** (DATE)
* `amount`: **주문 금액**

| order_id | user_id | order_date | amount |
| --- | --- | --- | --- |
| 101 | 1 | 2024-01-05 | 5000 |
| 102 | 1 | 2024-01-20 | 15000 |
| 103 | 2 | 2024-01-15 | 3000 |
| 104 | 3 | 2024-02-10 | 40000 |

---

## 2. 문제 상황 (Scenario)

**[경영진의 요청]**
"우리 서비스의 핵심 고객층인 **Chrome 브라우저 사용자** 들의 구매력이 궁금합니다.
이 사람들이 **월별로 평균 얼마치 (총액 기준)** 를 샀는지 보고서를 만들어 주세요."

**[분석 논리]**

1. **타겟팅:** 브라우저가 **'Chrome'** 인 유저만 골라내야 합니다.
2. **개인별 집계:** 먼저 각 유저가 **월별로 총 얼마를 썼는지 (`SUM`)** 계산해야 합니다. (철수: 1월 2만원)
3. **전체 평균:** 그 다음, 그 월별 유저 총액들의 **평균 (`AVG`)** 을 구해야 합니다. (Chrome 유저들은 1월에 평균 2만원씩 씀)
4. **포맷팅:** 날짜는 **"YYYY년 MM월"** 형태로 출력해야 합니다.

---

## 3. 원하는 결과 (Expected Output)

| report_month | avg_monthly_spend |
| --- | --- |
| 2024년 01월 | 20000 |
| 2024년 02월 | 40000 |

*(해설: 1월에 Chrome 유저인 1번은 총 20,000원(5000+15000)을 썼습니다. 2번은 Safari라 제외됨. 따라서 1월 평균은 20,000원입니다.)*

---

#### 힌트 (고민을 돕는 질문)

* **JSON 텍스트 추출 (PostgreSQL):** `info` 컬럼에서 'browser'라는 키(Key)에 해당하는 값(Value)을 텍스트로 꺼내려면 `->>` 연산자를 사용해야 합니다.
* 예: `info ->> 'browser'`


* **집계의 집계:** SQL에서는 `AVG(SUM(amount))` 처럼 집계 함수를 두 번 중첩해서 쓸 수 없습니다.
* 해결책: **서브쿼리**나 **CTE**를 사용하여 1단계(유저별 월 합계)를 먼저 구한 뒤, 메인 쿼리에서 2단계(그 합계들의 평균)를 구해야 합니다.


* **날짜 포맷팅:** PostgreSQL에서 날짜를 특정 문자열 형식으로 바꿀 때는 `TO_CHAR(컬럼, '포맷')` 함수를 사용합니다. 한글을 포함하려면 큰따옴표(`"`) 처리가 필요할 수 있습니다.
* 예: `TO_CHAR(date_col, 'YYYY"년" MM"월"')`

---

# 6. 앱 버전 v2.0 신규 유저의 익월 재방문율 (Retention) 분석

### DB :

---

## 1. 테이블 정보 (Schema)

**`access_logs`** (유저 접속 로그)

* `log_id`: **로그 ID**
* `user_id`: **유저 ID**
* `accessed_at`: **접속 일시** (TIMESTAMP)
* `meta_info`: **접속 환경 정보** (JSON)
* 예: `{"app_version": "v2.0", "device": "Android"}`

| log_id | user_id | accessed_at | meta_info |
| --- | --- | --- | --- |
| 1 | 1 | 2024-01-05 10:00:00 | `{"app_version": "v2.0"}` |
| 2 | 1 | 2024-02-01 12:00:00 | `{"app_version": "v2.0"}` |
| 3 | 2 | 2024-01-20 09:00:00 | `{"app_version": "v1.0"}` |
| 4 | 3 | 2024-01-25 14:00:00 | `{"app_version": "v2.0"}` |
| 5 | 3 | 2024-03-10 11:00:00 | `{"app_version": "v2.0"}` |

---

## 2. 문제 상황 (Scenario)

**[PO(Product Owner)의 요청]**
"최근 배포한 **앱 버전 'v2.0'** 으로 처음 유입된 유저들이 서비스에 만족하고 있는지 알고 싶습니다.
**'2024년 1월'에 v2.0 버전으로 '최초 접속(가입)'한 유저들 중**, 바로 다음 달인 **'2월'에도 접속한 유저의 비율 (Retention Rate)** 을 구해주세요."

**[상세 조건]**

1. **코호트 (모수) 정의:**
* 생애 첫 접속 기록이 **2024년 1월** 이어야 합니다.
* 그 첫 접속 당시의 앱 버전 (`app_version`) 이 **'v2.0'** 이어야 합니다.


2. **리텐션 (재방문) 정의:**
* 위 모수에 해당하는 유저가 **2024년 2월** 에 접속 기록이 **1건 이상** 존재해야 합니다.



---

## 3. 원하는 결과 (Expected Output)

* 비율은 소수점 첫째 자리까지 반올림하여 **%** 문자열로 출력하세요.
* 예: 1월 신규 유저가 2명(ID 1, 3)이고, 그중 1명(ID 1)만 2월에 접속했다면 **50.0%** 입니다. (ID 3은 3월 접속이라 2월 재방문 아님)

| cohort_month | retention_rate |
| --- | --- |
| 2024-01 | 50.0% |

---

#### 힌트 (고민을 돕는 질문)

* **최초 접속 찾기:** 유저별로 가장 처음 접속한 로그를 찾으려면 어떻게 해야 할까요?
* 방법 1: `MIN(accessed_at)`으로 날짜를 구하고 다시 조인하기.
* 방법 2: 윈도우 함수 `ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY accessed_at)`를 사용하여 1등 로그만 뽑아내기. (추천)


* **JSON 필터링:** `meta_info` 컬럼에서 버전을 추출할 때 PostgreSQL의 `->>` 연산자를 사용하세요.
* 조건: `meta_info ->> 'app_version' = 'v2.0'`


* **비율 계산:** `COUNT(리텐션 유저) / COUNT(코호트 전체 유저)`를 해야 합니다.
* 이때 정수끼리 나누면 소수점이 버려질 수 있으므로, 분자나 분모 중 하나를 `::NUMERIC`으로 형변환 해주는 것을 잊지 마세요.

* **Top-down 전략 (위에서 아래로):**
* 보통 "내 부모 찾기"보다 **"뿌리(Root)부터 시작해서 자식들에게 내 이름을 물려주기"** 가 구현하기 더 쉽습니다.
* **Anchor (시작점):** `parent_id IS NULL` 인 최상위 카테고리들을 먼저 찾습니다. 이때 `root_name` 컬럼을 자신의 이름으로 설정합니다.
* **Recursive (재귀):** 재귀 부분에서 자식(`categories`)을 찾을 때, 부모(`cte`)가 가지고 있던 `root_name` 을 그대로 **물려받습니다.**

---

# 5. Chrome 사용자들의 월별 구매력 분석 보고서

---

