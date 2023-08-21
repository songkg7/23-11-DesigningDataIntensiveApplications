# Ch3 저장소와 검색 (트랜잭션 처리나 분석?, 칼럼 지향 저장소)

## 트랜잭션 처리나 분석?

트랜잭션이라는 용어는 논리 단위 형태로서 읽기와 쓰기 그룹을 나타낸다.

> 트랜잭션이 반드시 ACID 속성을 가질 필요는 없다. 트랜잭션 처리는 일괄 처리 작업과 달리 클라이언트가 지연시간이 낮은 읽기와 쓰기를 가능하게 한다는 의미다.

- OLTP: Online Transaction Processing, 사용자 입력을 기반으로 데이터를 읽고 쓰는 작업을 수행하는 시스템
- OLAP: Online Analytical Processing, 데이터를 분석하는 시스템

데이터 분석은 트랜잭션과 접근 패턴이 매우 다르다. 보통 분석 질의는 사용자에게 원시 데이터를 반환하는 것이 아니라 많은 수의 레코드를 스캔해 레코드당 일부 칼럼만 읽어 집계 통계를 계산해야 한다.

| 특성       | OLTP                         | OLAP                                  |
|----------|------------------------------|---------------------------------------|
| 주요 읽기 패턴 | 질의당 적은 수의 레코드, 키 기준으로 가져옴    | 많은 레코드에 대한 집계                         |
| 주요 쓰기 패턴 | 임의 접근, 사용자 입력을 낮은 지연 시간으로 기록 | 대규모 불러오기(bulk import, ETL) 또는 이벤트 스트림 |
| 주요 사용처   | 웹 애플리케이션을 통한 최종 사용자/소비자      | 의사결정 지원을 위한 내부 분석가                    |
| 데이터 표현   | 데이터의 최신 상태(현재 시점)            | 시간이 지나며 일어난 이벤트 이력                    |
| 데이터셋 크기  | 기가바이트에서 테라바이트                | 테라바이트에서 페타바이트                         |

처음에는 두가지 모두 같은 데이터베이스에서 수행했지만 시간이 지나며 개별 데이터베이스에서 분석을 수행하는 경향을 보였다. 이 개별 데이터베이스를 **데이터 웨어하우스(data warehouse)** 라고 불렀다.

### 데이터 웨어하우징

OLTP 시스템은 대개 사업 운영에 대단히 중요하기 때문에 일반적으로 높은 가용성과 낮은 지연시간의 트랜잭션 처리를 기대한다. 그래서 데이터베이스 관리자는 OLTP 데이터베이스를 철저하게 보호하려 한다. 비즈니스
분석가가 OLTP 데이터베이스에 즉석 분석 질의(ad hoc analytic query)를 수행하면 데이터셋의 많은 부분을 스캔해 이와 동시에 실행되는 트랜잭션의 성능을 저하시킬 가능성이 있다.

반대로 데이터 웨어하우스는 분석가들이 OLTP 작업에 영향을 주지 않고 마음껏 질의할 수 있는 개별 데이터베이스다. 데이터 웨어하우스는 회사 내의 모든 다양한 OLTP 시스템에 있는 데이터의 읽기 전용 복사본이다.
데이터 웨어하우스로 데이터를 가져오는 이 과정을 **ETL(Extract-Transform-Load)** 이라 한다.

#### OLTP 데이터베이스와 데이터 웨어하우스의 차이점

각각 다른 질의패턴에 맞게 최적화됐기 때문에 시스템의 내부는 완전히 다르다.

### 분석용 스키마: 별 모양 스키마와 눈꽃 송이 모양 스키마

많은 데이터 웨어하우스는 별 모양 스키마로 알려진 상당히 정형화된 방식을 사용한다.

스키마의 중심에 소위 사실 테이블(fact table)이 있다.

## 칼럼 지향 저장소

사실 테이블은 칼럼이 보통 100개 이상이지만 일반적인 데이터 웨어하우스 질의는 한 번에 4개 또는 5개의 칼럼만 접근한다.

대부분의 OLTP 데이터베이스에서 저장소는 **로우 지향** 방식으로 데이터를 배치한다. 특정 칼럼을 가져오기 위해서는 모든 로우를 메모리로 적재한 다음 구문을 해석해 필요한 조건을 충족하지 않은 로우를 필터링해야 한다. 이 작업은 오랜 시간이 걸릴 수 있다.

**칼럼 지향 저장소**의 기본 개념은 간단하다. 모든 값을 하나의 로우에 함께 저장하지 않는 대신 각 칼럼별로 모든 값을 함께 저장한다. 각 칼럼을 개별 파일에 저장하면 질의에 사용되는 칼럼만 읽고 구분 분석하면 된다.

칼럼 지향 저장소 배치는 각 칼럼 파일에 포함된 로우가 모두 같은 순서인 점에 의존한다. 그러므로 로우의 전체 값을 다시 모으려면 개별 칼럼 파일의 23번째 항목을 가져온 다음 테이블의 23번째 로우 형태로 함께 모아 구성할 수 있다.

### 칼럼 압축

각 칼럼은 많은 값이 반복해서 나타나는 경향이 있다. 압축을 하기에 이런 경향성은 매우 좋은 징조다. 칼럼의 데이터에 따라 다양한 압축 기법을 사용할 수 있다. 그 중 한 가지 기법은 데이터 웨어하우스에서 특히 효과적인 **비트맵 부호화(bitmap encoding)** 이다.

#### 메모리 대역폭과 벡터화 처리

수백만 로우를 스캔해야 하는 데이터 웨어하우스 질의는 디스크로부터 메모리로 데이터를 가져오는 대역폭이 큰 병목이다.

칼럼 압축을 사용하면 같은 양의 L1 캐시에 칼럼의 더 많은 로우를 저장할 수 있다. AND 와 OR 같은 연산자는 압축된 칼럼 데이터 덩어리를 바로 연산할 수 있게 설계할 수 있다. 이런 기법을 **벡터화 처리(vectorized processing)** 라고 한다.

### 칼럼 저장소의 순서 정렬

### 칼럼 지향 저장소에 쓰기

### 집계: 데이터 큐브와 구체화 뷰

## 정리