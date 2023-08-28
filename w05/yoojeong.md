## 저장소와 검색

### 트랜잭션 처리나 분석?

보통 애플리케이션은 색인을 사용해 일부 키에 대한 적은 수의 레코드를 찾는다. 레코드는 사용자 입력 기반으로 추가 및 갱신된다. <br>
이런 애플리케이션은 대화식이기에 이런 접근 패턴을 온라인 트랜잭션 처리(online trrasaction processing, OLTP)이라 한다.


하지만 데이터베이스를 데이터 분석(data analytic)에도 점점 더 많이 사용하기 시작했고, 데이터 분석은 기존 트랜잭션과 접근 패턴이 매우 다르다. <br>
사용자에게 소수의 원시 데이터를 반환하는게 아닌 많은 수의 레코드를 스캔한 뒤 일부 칼럼만 읽어서 집계 통계(카운트, 합, 평균 등)를 계산해야 한다. <br>
이런 접근 패턴을 온라인 분석 처리(online analytic processing, OLAP)라고 부른다.



| 특성 | 트랜잭션 처리 시스템(OLTP)|분석 시스템(OLAP)|
|:--:|  :--:|:--:|
|주요 읽기 패턴|질의당 적은 수의 레코드, 키 기준으로 가져옴|많은 레코드에 대한 집계|
|주요 쓰기 패턴|임의 접근, 사용자 입력을 낮은 지연 시간으로 기록|대규모 불러오기(bulk import, ETL) 또는 이벤트 스트림|
|주요 사용처|웹 애플리케이션을 통한 최종 사용자/소비자|의사결정 지원을 위한 내부 분석가|
|데이터 표현|데이터의 최신 상태(현재 시점)|시간이 지나며 일어난 이벤트 이력|
|데이터셋 크기|기가바이트에서 테라바이트|테라바이트에서 페타바이트|

처음에는 둘 다 동일한 데이터베이스를 사용했으나, 최근에 들어서는 OLTP 시스템을 분석 목적으로 사용하지 않고 개별 데이터베이스에서 분석을 수행하는 경향을 보였으며, 이러한 개별 데이터베이스를 데이터 웨어하우스(data warehouse)라고 불렀다.


#### 데이터 웨어하우징

- OLTP 데이터베이스에 즉석 분석 질의(ad hoc analytic query)를 함부로 실행시켰다간 서비스의 전체 서비스 퀄리티 하락으로 이어질 수 있다.
- 데이터 웨어하우스는 분석가들이 OLTP 작업에 영향을 주지 않고 마음껏 질의할 수 있게 데이터의 읽기 전용 복사본을 저장하는 개별 데이터베이스이다.
- 데이터는 OLTP 데이터베이스에서 (주기적인 데이터 덤프나 지속적인 갱신 스트림을 사용해) 추출(extract)하고 분석 친화적인 스키마로 변환(transform)하여 데이터 웨어하우스에 적재(load)한다.(ETL)

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/429bce98-c470-4c12-9578-847556c0cabc)


#### OLTP 데이터베이스와 데이터 웨어하우스의 차이점
SQL은 일반적으로 분석 질의에 적합하기에 데이터웨어 하우스는 일반적인 관계형 모델을 사용한다.

- SQL 질의를 생성하고 결과를 시각화
- 분석가가 드릴 다운(drill-down), 슬라이싱(slicing), 다이싱(dicing)같은 작업을 통해 데이터를 탐색할 수 있게 해주는 여러 그래픽 데이터 분석 도구 존재
  
OLTP나 데이터 웨어하우스 둘 다 SQL 질의 인터페이스를 지원한다는 공통점이 있지만, 둘은 서로 매우 다른 질의 패턴에 맞게 최적화되었기 때문에 내부는 완전히 다르다.<br>
그렇기에 다수의 데이터베이스 벤더 역시 트랜잭션 처리와 분석 작업부하를 둘 다 지원하기보단 하나에 특화되도록 지원하는데 중점을 둔다. 

> 참고: 드릴 다운(drill-down), 슬라이싱(slicing), 다이싱(dicing)
드릴 다운은 요약된 정보에서 상세 정보까지 계층을 나눠 구체적으로 분석하는 작업.
> 슬라이싱과 다이싱은 상세한 분석을 위해 주어진 큰 규모의 데이터를 작은 단위로 나누고 원하는 세부 분석 결과를 얻을 때까지 반복한다는 의미.

#### 분석용 스키마: 별 모양 스키마와 눈꽃송이 모양 스키마

많은 데이터 웨어하우스는 별 모양 스키마(star schema, 혹은 차원 모델링)로 상당히 정형화된 방식을 사용한다. 

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/c51d7fe2-6efb-462c-af48-6875a11c8bea)


- `사실 테이블(fact table)`: 각 로우는 특정 시각에 발생한 이벤트에 해당한다.(e.g. 고객의 제품 구매) 사실 테이블의 일부 컬럼은 제품이 판매된 가격과 공급자로부터 구매한 비용과 같은 속성이다. 나머지 다른 컬럼들은 차원 테이블을 가리키는 외래키 참조다.
- `차원테이블(dimension table)`: 이벤트의 속성인 누가, 언제, 어디서, 무엇을, 어떻게, 왜를 나타낸다. (e.g. 판매된 제품(dim_product))
  
이러한 템플릿의 변형을 눈꽃송이 모양 스키마(snowflake schema)라고 하며 차원이 하위 차원으로 더 세분화된다. <br>
쉽게 말해 하위 차원테이블이 모두 실제 값만을 저장하는게아닌 또다른 차원테이블의 외래키를 저장하여 세분화하는 것이다. <br>
이는 별 모양 스키마보다 더 정규화됐지만, 작업 난이도가 더 올라가기 때문에 분석가들은 별 모양 스키마를 더 선호한다.

### 칼럼 지향 저장소


```sql
SELECT
  dim_date.weekday, dim_product.category,
  SUM(fact_sales.quantity) AS quantity_sold
FROM fact_sales
  JOIN dim_date ON fact_sales.date_key = dim_date.date_key
  JOIN dim_product ON fact_sales.product_sk = dim_product.product_sk
WHERE
  dim_date.year = 2013 AND
  dim_product.category IN ('Fresh fruit', 'Candy')
GROUP BY
  dim_date.weekday, dim_product.category;
```

사실 테이블에는 엄청난 개수의 로우와 페타바이트 데이터가 있기 떄문에 모든 값을 하나의 로우에 함께 저장하지 않는 대신 각 컬럼 별로 모든 값을 함께 저장한다.<br>
각 컬럼을 개별 파일에 저장하면 질의에 사용되는 컬럼만 읽고 분석하면 된다.


![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/e223d259-0ddf-4a03-a575-0af208818fdf)


### 칼럼 압축
압축을 통해 데이터를 압축하면 디스크 처리량을 줄일 수 있는데, 칼럼 지향 저장소는 대게 압축에 적합하다.<br>
product_sk 처럼 많은 값이 반복될 경우 데이터 웨어하우스에서 특히 효과적인 비트맵 부호화(bitmap encoding) 압축을 사용할 수 있다.

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/c2c642be-91d9-4e9e-a760-f57fd5363b5e)


칼럼에서 고유 값의 수는 로우 수에 비해 적다. (product_sk : 18개의 데이터, 고유한 값은 6가지(29, 30, 31, 68, 69, 74) 뿐) <br>
그렇기에 n개의 고유 값을 가진 칼럼을 가져와 n개의 개별 비트맵으로 변환할 수 있다.  

위 그림과 같이 고유 값 하나가 하나의 비트맵이고 각 로우는 한 비트를 가지기에 로우가 해당 값을 가질 경우 비트는 1이고 아니면 0이다. <br>
여기서 고유 값(n)이 클수록 대부분의 비트맵은 0이 더 많아지는데 이를 희소(sparse)하다고 말하는데, 이 경우 위 그림의 하단처럼 런 렝스 부호화(run-length encoding)할 수 있다.


1. `WHERE product_sk IN (30, 68, 69)`
: `product_sk = 30`, `product_sk = 68`, `product_sk = 69` 에 비트맵 3개를 적재하고 3개 비트맵의 비트를 OR 로 연산한다. 해당 조건에 해당하는 값이 몇 번째 인덱스에 존재하는지 쉽게 확인할 수 있다. 

3. `WHERE product_sk = 31 AND store_sk = 3`
: `product_sk = 31`과 `store_sk = 3`인 비트맵을 적재해 비트 AND를 연산하면, 해당 계산은 각 칼럼에 동일한 순서로 로우가 포함되기 때문에 연산 결과의 비트맵 중 ‘1’의 위치가 곧 질의 결과에 해당하는 row 순서이다.

> 참고: 칼럼 지향 저장소와 칼럼 패밀리
카산드라, HBase 와 같은 빅테이블 개념에서 나오는 칼럼 패밀리는 row 키에 따라 row와 모든 칼럼을 함께 저장하며 칼럼 압축을 하지 않는다. 따라서 칼럼 지향 저장소와는 다른 개념이며, 빅테이블 모델은 여전히 대부분 로우 지향이다.


### 메모리 대역폭과 벡터화 처리

데이터 웨어하우스 질의는 디스크로부터 메모리로 데이터를 가져오는 대역폭뿐 아니라, 메인 메모리에서 CPU 캐시로 가는 대역폭을 효율적으로 사용하고 CPU 명령 처리 파이프라인에서 분기 예측 실패(branch misprediction)와 버블(bubble)을 피하며 최신 CPU에서 단일 명령 다중 데이터(SIMD)명령을 사용하게끔 신경써야 한다.

칼럼 저장소 배치는 CPU 주기를 효율적으로 사용하기에 적잡한데, 예를 들어 질의 엔진은 압축된 칼럼 데이터를 CPU의 L1캐시에 청크를 나눠 가져오고 이 작업을 타이트 루프(tight loop)에서 반복한다. <br>

- CPU는 함수 호출이 많이 필요한 코드나 각 레코드 처리를 위해 분기가 필요한 코드보다 타이트 루프를 훨씬 빠르게 실행할 수 있다.
- 칼럼 압축을 사용하면 같은 양의 L1 캐시에 칼럼의 더 많은 로우를 저장할 수 있다.
- 앞에서 설명한 비트 AND 와 OR같은 연산자는 압축된 칼럼 데이터 덩어리를 바로 연산할 수 있게 설계할 수 있다.


이런 기법을 벡터화 처리(vectorized processing)라고 한다.

> 분기 예측실패와 버블
분기 예측 실패를 알기위해선 분기 예측을 알 필요가 있다. 
분기 예측이란 CPU 처리성능 향상을 위해 명령(instruction)처리 과정을 여러 단계로 세분화 하는기법인 파이프라인을 통한 명령 실행 중 조건 분기 명령의 실행이 종료될 때까지 다음 명령을 대기하지 않고 분기를 예측 실행해 파이프라인 처리 성능 저하를 최소화하는 CPU 실행 기술이며, 이런 예측이 실패하는 것을 분기예측실패라 한다. 어떤 방식으로 분기를 예측하는지에 대해서는 주제와 벗어나기에 생략한다.  그리고 파이프라인에서 구조적 해저드를 해소하기 위해 버블을 넣는데, 이 때 명령의 처리를 진행하지 않는다. 

### 칼럼 저장소의 순서 정렬

칼럼 저장소에서 로우가 삽입된 순서로 저장하는 방식이 가장 쉽다.<br>
하지만, 이전의 SS 테이블에서 했던 것처럼 순서를 도입해 이를 색인 메커니즘으로 사용할 수 있다. 


각 칼럼을 독립적으로 정렬할수는 없으므로(칼럼의 그 순서에 있는 모든 항목이 동일한 row에 속한다는 것을 보장하기 위해) 칼럼별로 저장하더라도 데이터는 한번에 전체 row를 정렬해야 한다.

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/c76f1181-8385-477d-bc99-9ce8dc83eb40)

ex.) age칼럼만 단독 정렬하는 경우

공통 질의에 대한 지식이 있으면 그 값을 기반으로 테이블에서 정렬해야 하는 칼럼을 선택할 수 있고, 이런 특징을 이용해 그룹화와 필터링 질의에 도움을 줄 수 있다.


정렬된 순서의 또 다른 장점으로는 칼럼 압축이 있는데, 기본 정렬 칼럼에 고유 값을 많이 포함하지 않는다면 정렬된 칼럼은 연속해서 같은 값이 길게 반복되므로 압축에 효과적이다.<br>
이런 압축 효과는 보통 첫번째 정렬 키에서 가장 강력하다.


#### 다양한 순서 정렬

- 다양한 질의는 서로 다른 정렬 순서의 도움을 받는다.
  - 복제 데이터를 다양한 방식으로 정렬해서 저장하면 질의를 처리할 때 질의 패턴에 가장 적합한 버전을 사용할 수 있다.
- 칼럼 지향 저장소에서 여러 정렬 순서를 갖는 것은 row 지향 저장소에서 여러 2차 색인을 갖는 것과 비슷하다.
  - 그러나 row 지향 저장소는 한 곳(힙 파일이나 클러스터드 색인)에 모든 row를 유지하고 2차 색인은 일치하는 row의 참조만 포함한다는 점이 다르다.
  - 칼럼 저장에서는 데이터를 가리키는 참조는 없고 단지 값을 포함한 칼럼만 존재한다.


#### 칼럼 지향 저장소에 쓰기

데이터 웨어하우스에서 칼럼 지향 저장소 뿐만아니라 압축, 정렬 모두 읽기 질의를 빠르게 하는 방법이다.
<br>하지만, 반대로 쓰기를 어렵게 하는 방법이기도 하다. 

B 트리 같은 제자리 갱신(update-in-place)은 압축된 칼럼에서는 불가능하다. 
<br>정렬된 테이블의 중간에 있는 row에 삽입하는 경우 모든 칼럼 파일을 재작성해야 하기 때문이다.

반면 LSM 트리를 사용하면 로우 지향인지 칼럼 지향인지 상관없이 모든 쓰기는 먼저 메모리 저장소로 이동해 정렬된 구조에 추가하고 디스크에 쓸 준비를 한다. 
<Br>충분한 쓰기를 모으면 디스크 칼럼 파일에 병합하고 대량으로 새 파일에 기록할 수 있다.


#### 집계: 데이터 큐브와 구체화 뷰


데이터 웨어하우스는 구체화 집계(materialized aggregate)라는 측면을 가지고 있기에, 보통 질의에 집계 함수(`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`)가 포함되는 경우가 많다. <br>
동일한 집계를 다양한 질의에서 사용한다면 매번 원시 데이터를 처리하는 것은 낭비이다. <br>
질의가 자주 사용하는 카운트나 합을 캐시할 수 있는데 이런 캐시를 만드는 한가지 방법이 구체화 뷰이다.



관계형 데이터 모델에선 이런 (COUNT, SUM 같은)캐시를 대개 표준 (가상)뷰로 정의하는데,  표준 뷰는 테이블 같은 객체로 일부 질의의 결과가 내용이다. 

- 구체화 뷰 : 디스크에 기록된 질의 결과의 실제 복사본
- 가상 뷰(virtual view) : 단지 질의를 작성하는 단축키, 가상 뷰에서 읽을 때 SQL엔진은 뷰의 원래 질의를 즉석에서 확장하고 나서 질의를 처리


원본 데이터를 변경하면 구체화 뷰 역시 변경해야 한다.<br>
데이터베이스는 이 작업을 자동으로 할 수 있지만 이런 갱신을 위한 쓰기 비용이 비싸기 때문에 OLTP 데이터베이스에서는 자주 사용하지 않지만 데이터 웨어하우스는 읽기 비중이 크기 때문에 구체화 뷰를 사용한다.


데이터 큐브(data cube) 또는 OLAP 큐브라 알려진 구체화 뷰는 일반화된 구체화 뷰의 특별 사례다. 

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/abbd7a6c-4893-4cd9-93fa-1c70560d9872)

위 그림은 사실 테이블이 2차원 테이블에만 외래 키를 가진다고 가정할 때 그릴 수 있는 2차원 테이블이다.<br>
각각의 축은 날짜(date), 제품(product)이고 각 셀은 날짜와 제품을 결함한 모든 사실의 속성의 집계값(Ex: SUM)이다.  


구체화 데이터 큐브의 장점은 특정 질의를 효과적으로 미리 계산했기 때문에 해당 질의를 수행할 때 매우 빠르다.

단점은 원시 데이터에 질의하는 것과 동일한 유연성이 없다는 점이다. 따라서 데이터 웨어하우스는 가능한 한 많은 원시 데이터를 유지하려고 노력한다. 데이터 큐브와 같은 집계값은 특정 질의에 대한 성능 향상에만 사용한다.