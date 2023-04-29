# 2주차 

## 숙제 리뷰

데이터 웨어하우스에서 데이터를 로드에서 추출하는 (ELT) 작업을 통해서 Summary Table 을 1시간마다 만든다. 

- 조인, Count, Distinct, GroupBy 의 동작 원리에 대해서 공부해보자.

## 실습 전에 기억할 점. 

- 현업에서 항상 깨끗한 데이터는 존재하지 않으니 이게 사용할만한 데이터인지 의심하는 것. 
  - **실제 레코드를 살펴보고 판단하는게 제일 좋다.**

- **데이터 일을 한다면 데이터의 품질을 확인하는 테스트를 해봐야한다.** 
  - 중복된 레코드가 있는지 확인하는 것. 
  - 최근 데이터의 존재 여부를 확인하는 것. (freshness)
  - 값이 비어있는 칼럼이 있는지 확인하는 것. 
  - Primary Key 가 Uniqueness 지켜지는지 확인하는 것. 
  - 위의 체크를 Unit Test 를 통해서 확인하는 것.

- 시간이 지나면 테이블들이 엄청나게 많아진다. (무슨 데이터가 있는지 확인하는 것도 어려워진다.) 
  - 이게 **data discovery** 라고한다. 이런 문제들도 생긴다.  
  - 그래서 중요 테이블들이 머고, 이 데이터가 무엇인지 나타내는 메타 정보를 관리하는게 중요해진다. 

## SQL 기본 

다수의 SQL 을 사용한다면 `;` 을 통해서 구별해야한다. 
- `SQL1; SQL2; SQL3;`
- SQL 키워드는 컨벤션을 맞추는게 좋다. 대문자를 쓴다던지. 
- 테이블과 필드 이름은 컨벤션을 맞추는게 좋다. 
  - 단수형을 쓰는지 복수형을 쓰는지.
  - 카멜 케이스를 쓸건지, 스네이크 케이스를 쓸건지

Primary Key 는 빅데이터 웨어하우스에서는 지켜지지 않는다.
- SNOW FLAKE, Big Query, RedShift

CTAS (CREATE TABLE schema_name.table_name AS SELECT)
- 대문자로 표시된 것들의 줄임말.
- SELECT 한 것들을 통해서 table 을 만드는 것을 말한다.
- CTAS 를 하면 ELT 를 하는것. (summary table 을 만드는 것)

DROP TABLE IF EXIST table_name 
- 테이블이 존재하면 지우는 것
- vs DELETE FROM: DELETE 는 레코드만 지우는 것.

WHERE 절에서 LIKE 와 ILIKE 
- ILIKE 는 대소문자를 구별 안함.

String Functions 
- LEFT(str, n)
- REPLACE(str, exp1, exp2)
- UPPER(str)
- LOWER(str)
- LEN(str)
- LPAD, RPAD
  - LPAD(input_string, target_length, pad_string)
  - SELECT LPAD('SQL', 10, '0') AS PaddedString;
    - 0000000SQL
- SUBSTRING

ORDER BY
- multiple column ORDER BY
  - `ORDER BY 1 DESC 2,3`

- NULL value ordering 
  - 기본 ORDER BY 는 오름차순 (ASC). 이 경우에는 NULL VALUE 가 가장 마지막에 옴. 
  - ORDER BY DESC 로 한다면 내림차순이다. 이 경우에는 NULL VALUE 가 가장 첫 번째에 온다.
    - ORDER BY 1 DESC NULLS LAST; -- NULL 값이 맨 뒤에옴.

Type Cast and Conversion 
- Type Cast 
  - cast 를 쓰거나 :: operator 를 쓰거나 
- TO CHAR, TO_TIMESTAMP 
  - TO CHAR 은 date -> String
  - TO_TIMESTAMP 는 String -> Date 
- Date Conversion 
  - CONVERT TIMEZONE
  - DATE, TRUNCATE
  - DATE_TRUNC

NULL
- IS NULL or IS NOT NULL 로 비교한다. 
- Boolean 타입도 IS TRUE or IS FALSE 로 비교하기도 한다. 
- LEFT JOIN 시에 매칭되는 것이 있는지 확인하는데 유용하다.
  - LEFT JOIN 시에 매칭되는 값이 없는 경우에는 NULL 이 나오기 때문에 확인할 수 있음.
- NULL 값을 다른 값으로 바꾸고 싶다면 `COALESCE` 를 사용하면 된다. 

JOIN 

![](./image/join%20type.png)

![](./image/join%20sql.png)

JOIN 고려할 때 주의할 점 
- 중복 레코드가 없고, PRIMARY KEY 가 정말 unique 한지 확인해야한다. 보통 primary key 를 가지고 조인을 하니까.
  - 이걸 확인해야하는 이유가 중복 레코드이거나 PRIMARY KEY 가 unique 하지 않다면 JOIN 결과도 여러개가 나올 수 있어서 그렇다. 그걸 원하지 않는 결과일 수 있으니까. 
  - 예로 ORDER 테이블 레코드에 CUSTOMER_ID 가 중복인 경우가 있다면 조인을 했을 때 하나의 결과가 나오는게 아니라 중복된 결과가 나올 것이니까. 
  - 중복을 제거하려면 GROUP_BY 나 DISTINCT 를 사용해야한다. 

- 조인할 때 대상의 관계를 명확히 인지해야한다.
  - OneToOne, OneToMany, ManyToOne, ManyToMany. 이 관계에 맞춰서 조인 결과도 나올 것이니.

- 어느 테이블을 드라이빙 테이블로 잡을지 생각해야한다.

## 복습 체크리스트 

<details>
<summary> 조인할 때 신경써야 할 점들은? </summary>

</details>


<details>
<summary> Order By 를 사용할 때 주의할 점은? </summary>



</details>