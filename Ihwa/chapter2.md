# 2장. 데이터 모델과 질의 언어

## 관계형 모델과 문서 모델

### 관계형 모델

- **`데이터`**(테이블)은 관계로 구성 / 각 관계는 순서 없는 **`튜플`**(로우)의 모음
- 목표 : 정리된 인터페이스 뒤로 **구현 세부 사항을 숨기는 것**
- 일상적으로 사용하는 경우
    - 트랜잭션 처리(영업이나 은행 거래, 항공 예약, 창고에 재고 보관)
    - 일괄 처리(고객 송장 작성, 급여 지불, 보고)

### NoSQL

- Not Only SQL
- NoSQL 데이터베이스를 채택하는 원동력
    - 대규모 데이터셋이나 매우 **높은 쓰기 처리량 달성**을 관계형 데이터베이스보다 쉽게 할 수 있는 뛰어난 확장성
    - 상용 데이터베이스 제품보다 **무료 오픈소스 소프트웨어에 대한 선호도** 확산
    - 관계형 모델에서 지원하지 않는 **특수 질의 동작**
    - 관계형 스키마의 제한에 대한 불만과 **더욱 동적이고 표현력이 풍부한 데이터 모델**에 대한 바람

**다중 저장소 지속성**
애플리케이션의 요구사항의 다양성으로 인해 여러 사용 사례에 맞는 최적화를 위해서는 **관계형 데이터베이스**와 **비관계형 데이터스토어**가 **함께 사용**될 것이다.

<br/>

### 객체 관계형 불일치

**임피던스 불일치**
OOP 데이터를 관계형 테이블에 저장하기 위해서 애플리케이션 코드와 데이터베이스 모델 객체(테이블, 로우, 칼럼) 사이의 전환 계층 필요

**객체 관계형 매핑**(**ORM** - ex.액티브레코드나 하이버네이트) 프레임워크는 전환 계층 필요없이 상용구 코드의 양을 줄이지만 두 모델간의 차이를 완벽히 숨길 수는 없다.

문서형 데이터는 JSON 표현에 적합(JSON은 XML보다 간단)  
- 다중 테이블 스키마보다 더 나은 지역성을 제공
    - 다중 조인을 수행할 필요 없이 모든 관련 정보가 한 곳에 있어 질의 하나로 충분
- 문서 지향 데이터베이스(몽고DB, 리싱크DB, 카우치DB, 에스프레소)
    - JSON데이터 모델을 지원

<br/>

### 다대일과 다대다 관계

**ID나 텍스트 문자열의 저장 여부** ⇒ `중복 문제`  
- ID : 사람에게 의미있는 정보는 한 곳에만 저장하고 그것을 참조하는 모든 것은 ID를 사용한다.
- 텍스트 : 그것을 사용하는 모든 레코드에서 사람을 의미하는 정보를 중복해서 저장하게 된다.

<br/>  

**ID를 사용하는 이유**    
- ID 자체에는 아무런 의미가 없기 때문에 변경할 필요가 없다.
- 식별 정보를 변경해도 ID는 동일하게 유지할 수 있다.


중복된 데이터를 정규화하려면 **다대일 관계**가 필요  
→ But! 문서 모델에 적합하지 않다.  
- 관계형 데이터베이스에서는 조인이 쉬움(ID로 다른 테이블의 로우를 참조)
- 문서형 데이터베이스에서는 조인에 대한 지원이 약함
- **계층 모델** (IBM 정보관리시스템)
    - 모든 레코드는 정확하게 하나의 부모가 존재
    - JSON구조와 비슷하게 모든 데이터를 레코드 내에 중첩된 레코드 트리로 표현
    - 다대다 관계 표현 어려움
    - Join을 지원하지 않음

<br/>

위 계층 모델의 한계를 해결하기 위한 해결책 2가지  
- **네트워크 모델(코다실 모델)**
    - 계층모델을 일반화 : 레코드는 `다중 부모`가 있을 수 있다.
    - 레코드 간 연결은 외래 키보다는 프로그래밍 언어의 포인터와 더 비슷
    - 레코드 접근을 위해서는 최상위 레코드서부터 연속된 연결 경로를 따라야 한다. → **접근 경로**
    - 수동 접근 경로 선택
        - 매우 제한된 하드웨어 성능을 가장 효율적으로 사용 가능
        - 데이터베이스 질의와 갱신을 위한 코드가 복잡 & 유연하지 않음
        - 원하는 데이터에 대한 경로가 없다면 아주 수많은 데이터베이스 질의 코드를 살펴봐야 하고 새로운 접근 경로를 다루기 위해 재작성 해야한다.
- **관계형 모델**
    - 알려진 모든 데이터를 배치하는 것.
    - `관계`(테이블)는 `튜플`(로우)의 컬렉션이 전부이다.
    - 복잡한 접근 경로 없이 임의 조건과 일치하는 로우를 선택해서 읽을 수 있다.
    - 질의 최적화기는 질의의 어느 부분을 어떤 순서로 실행할지를 결정하고 사용할 색인을 자동으로 결정한다. → 접근 경로

<br/>

**문서 모델**
- 한가지 측면에서 계층 모델로 되돌아감.
    - 별도 테이블이 아닌 상위 레코드 내에 중첩된 레코드를 저장
- 문서 내 중첩 항목 참조 불가능
- 미흡한 조인 지원
→ 다대다 관계 사용 시 문서 모델의 매력 하락

<br/>

### **관계형 모델과 문서 모델의 비교**

- 다대일과 다대다 관계 표현 시 관계형 데이터베이스와 문서 데이터베이스 둘 다 관련 항목은 **고유한 식별자**로 참조
    - 고유한 식별자 : 관계형 모델- `외래키`  / 문서 모델 - `문서 참조`
- 문서 데이터 모델을 선호하는 주요 이유
    - 스키마 유연성, 지역성에 기인한 더 나은 성능
    - 일부 애플리케이션-애플리케이션에서 사용하는 데이터 구조와 더 가까움
- 관계형 모델을 선호하는 주요 이유
    - 조인, 다대일, 다대다, 관계를 더 잘 지원
- 문서와 같은 일대다 관계 트리 구조 → 문서 모델 사용
- 여러 테이블로 나누어 찢는 관계형 기법은
    - 다루기 힘든 스키마와 불필요하게 복잡한 애플리케이션 코드 발생

<br/>

**스키마가 없음(스키마리스)** 
- 임의의 키와 값을 문서에 추가 가능
- 읽을 때 클라이언트는 문서에 포함된 필드의 존재 여부를 보장하지 않음
- 그러나 데이터베이스에서는 강요하지 않지만 데이터를 읽는 코드는 보통 구조의 유형을 어느정도 가정하여 **`암묵적인 스키마가 존재`**한다.
    - 쓰기 스키마 : 정적(컴파일타임) 타입 확인과 비슷
    - 읽기 스키마 : 동적(런타임) 타입과 비슷
- ALTER TABLE  
    대부분의 관계형 데이터베이스 시스템은 `ALTER TABLE`을 수 밀리초 안에 수행한다. 하지만 MySQL은 `ALTER TABLE` 시에 전체 테이블을 복사하여 큰 테이블 변경 시 수 분에서 수 시간까지 중단시간이 발생한다는 의미다.
    

<br/>

### **질의를 위한 데이터 지역성**

문서의 일반적인 저장 형식  
- JSON, XML로 부호화된 단일 연속 문자열
- JSON 또는 XML의 이진 변형

<br/>

**저장소 지역성**
- 웹 페이지 사엥 문서를 보여주는 동작처럼 애플리케이션이 자주 전체 문서에 접근해야 할 ****때 이점이 있음
- 지역성의 이점 → 한 번에 해당 문서의 많은 부분을 필요로 하는 경우에만 적용

<br/>

### **문서 데이터베이스와 관계형 데이터베이스의 통합**  
 관계형 데이터베이스와 문서 데이터베이스는 각 데이터 모델이 서로 부족한 부분을 보완해 나가며 시간이 지남에 따라 점점 더 비슷해지고 있다.

<br/>

## 데이터를 위한 질의 언어
**SQL : 선언형 질의 언어**

- 일반적으로 명령형 API보다 더 간결하고 쉽게 작업할 수 있기 때문에 매력적임
- 종종 병렬 실행에 적합 → 결과의 패턴만 지정하기 때문

<br/>

**IMS : 명령형 코드를 사용** 

- 명령형 언어는 특정 순서로 특정 연산을 수행하게끔 컴퓨터에게 지시

<br/>

**맵리듀스 질의**   
- 많은 컴퓨터에서 대량의 데이터를 처리하기 위한 프로그래밍 모델
- 몽고DB, 카우치 DB를 포함한 일부 NoSQL 데이터 저장소는 제한된 형태의 맵리듀스를 지원 - 메커니즘은 많은 문서를 대상으로 읽기 전용 질의를 수행할 때 사용 → 10장에서 설명

[[하둡] 맵리듀스(MapReduce) 이해하기](https://12bme.tistory.com/154)

<br/>

## 그래프형 데이터 모델

데이터에서 다대다 관계가 매우 일반적이라면 그래프로 데이터를 모델링하기 시작하는 편이 더 자연스럽다.

**그래프의 두 유형 객체**
- 정점
- 간선

<br/>

**그래프의 종류**
- 소셜 그래프
- 정점은 사람이고 간선은 사람들이 서로 알고 있음을 나타낸다.
- 웹 그래프
- 정점은 웹 페이지고 간선은 다른 페이지에 대한 HTML 링크를 나타낸다.
- 도로나 철도 네트워크
- 정점은 교차로이고 간선은 교차로 간 도로나 철로 선을 나타낸다.

그래프는 동종 데이터에 국한되지 않는다.

<br/>

### 속성 그래프
ex) 네오포제이, 타이탄, 인피니티그래프

**정점 요소**
- 고유한 시별자
- 유출 간선 집합
- 유입 간선 집합
- 속성 컬렉션(키-값 쌍)

**간선 요소**
- 고유한 식별자
- 간선이 시작하는 정점(꼬리 정점)
- 간선이 끝나는 정점(머리 정점)
- 두 정잠 간 관계 유형을 설명하는 레이블
- 속성 컬렉션(키-값 쌍)

**사이퍼(Cypher)**  
속성 그래프를 위한 선언형 질의 언어

<br/>

### 트리플 저장소와 스파클
ex) 데이토믹, 알레그로그래프 등

**트리플 저장소 정보의 세 부분 구문 형식**
- 주어
- 서술어
- 목적어

**시맨틱 웹**
웹 사이트는 이미 많은 사람이 읽을 수 있는 텍스트와 그림으로 정보를 게시하고 있으니 컴퓨터가 읽게끔 기계가 판독 가능한 데이터로도 정보를 게시하는건 어떨까?   
-> **자원 기술 프레임워크**  

[[IT 상식사전] 시맨틱 웹에 관하여 | 요즘IT](https://yozm.wishket.com/magazine/detail/1377/)