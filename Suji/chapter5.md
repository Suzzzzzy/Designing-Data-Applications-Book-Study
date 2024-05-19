# 복제

: 네트워크로 연결된 여러 장비에 동일한 데이터의 복사본을 유지
- 지리적으로 사용자와 가깝게 데이터 유지 -> 지연 시간 감소
- 일부 장애 발생해도 지속적 동작 가능
- 읽기 제공 장비 수 확장 -> 읽기 처리량 늘림
- 복제의 어려움 - 복제된 데이터의 변경 처리
- 노드 간 변경을 복제하기 위한 세가지 알고리즘 존재: 단일리더, 다중리더, 리더없는 복제
- 복제 시 고려할 트레이드오프: 동기식 복제 vs 비동기식 복제 / 잘못된 복제 처리 방법

## 리더와 팔로워 - 리더 기반 복제
- 복제 서버 중 하나를 리더로 지정
- 클라이언트는 쓰기 요청을 리더에게 전송
- 리더는 먼저 로컬 저장소에 새로운 데이터 기록 - 할 때마다 팔로워에게 복제로그 or 변경 스트림 전송
- 각 팔로워가 리더로부터 로그 받으면 동일한 순서로 모든 쓰기 적용 -> 로컬 복사본 갱신
- 읽기 질의는 팔로워에게 가능 / 쓰기는 리더에게만
- PostgreSQL, MySQL, Oracle,,,, MongoDB 등 비관계형 데이터베이스에서도 사용
- 카프카나 큐 같은 분산 메세지에도 사용

### 동기식 vs 비동기식 복제
1. 동기식 복제
- : 리더가 팔로워가 쓰기를 수신했는지 확인을 줄때까지 기다리고 확인이 끝나면 사용자에게 보고
- + 팔로워가 리더와 일관성 있게 최신 복사본을 가지는 것을 보장
- + 리더가 작동하지 않아도 팔로워에서 확신 가능
- - 팔로워가 응답하지 않으면 쓰기가 처리될 수 없다 -> 리더는 쓰기를 차단하고 서버 복구를 기다려야 한다
- 따라서 모든 팔로워가 동기식인 것은 비현실적

2. 비동기식 복제
- : 리더는 쓰기 메시지를 전송하지만 팔로워의 응답을 기다리지 않는다
- 보통 리더 기반 복제는 완전히 비동기식으로 구성
- 리더가 잘못되면 팔로워에 아직 복제되지 않은 모든 쓰기는 유실 -> 지속성 보장 X
- 모든 팔로워가 잘못되더라도 리더는 쓰기 처리 계속 가능

3. 반동기식 복제
- : 팔로워 하나는 동기식, 그 밖에는 비동기식으로
- 적어도 두 노드(리더와 동기 팔로워)는 최신 복사본 있는 것 보장
- 현실적으로 동기식 복제를 사용하려면 반동기식으로 구현

### 새로운 팔로워 설정
복제 서버 수를 늘리거나, 장애 노드 대체를 위해 새로운 팔로워를 설정할 경우 -> 새로운 팔로워가 리더의 것을 가지고 있을 것을 보장하는 방법은?
- 데이터베이스를 잠가서 중단한 후 일관성있게 만들어주는 것은 가용성 X
- 리더의 스냅샷을 일정 시점에 가져온다
- 스냅샷을 팔로워 노드에 복사
- 스냅샷 이후의 데이터 복제 로그를 리더로 부터 가져온다
- 팔로워가 모두 처리하면 따라 잡은 것! 이제부터 리더에 변경에 이어서 처리 가능

### 개별 노드의 장애(중단)에도 전체 시스템의 영향을 최소화 하는 방법 -> 고가용성 달성 방법
1. 팔로워 장애 - 따라잡기 복구
- 팔로워는 리더로 부터 수신한 변경로그를 로컬 디스크에 보관
- 리더와 네트워크가 일시 중단되어도 팔로워는 쉽게 복구 가능
- 마지막 트랜잭션을 알아내고, 연결이 끊어진 동안 리더에게 발생한 데이터 변경 모두 요청

2. 리더 장애 - 장애 복구
- 리더가 장애인지 판단 - 일정 시간 노드가 응답하지 않으면 죽은것으로 간주
- 새로운 리더를 선택 - 리더의 최신 데이터 변경사항을 가진 복제 서버로 선택
- 클라이언트가 새로운 리더에게 쓰기 요청을 보내도록 시스템 재설정
- 장애 복구 과정이 잘못되는 경우: 비동기식 복제로 인한 이전 리더의 복제되지 않은 쓰기 충돌, 유효하지 않은 팔로워(뒤쳐진 서버)가 리더로 승격, 두 노드가 모두 리더라고 믿는 상황, 리더가 죽은 것이 아니라 부하급증으로 인한 시간 지연인 경우 
- 따라서 수동으로 복구 하는 것이 선호됨

### 복제로그 구현
1. 구문 기반 복제
- 리더는 모든 쓰기 요청을 구문으로 기록하고 실행
- rand, now같은 비결정적 함수를 호출하는 경우 서버마다 다른 값을 생성할 수 도 있음
- 자동증가 칼럼을 사용하는 경우 복제서버에서 정확하게 같은 순서로 실행되어야 함
- 따라서 mySQL에서는 구문에 비결정성이 있다면 **로우 기반 복제**로 실행

2. 쓰기 전 로그(WAL) 배송
- 모든 쓰기는 로그에 기록되므로 리더가 팔로워에게 네트워크로 로그 전송
- 팔로워가 이 로그를 처리하면 리더와 동일한 복제본 완성
- 어떤 디스크 블록에서 어떤 바이트를 변경했는지와 같은 상세 정보 포함
- 데이터베이스 버전이 달라질 경우 - 팔로워를 먼저 업그레이드 하고 새로운 리더로 선정 후 장애 복구 수행(버전 불일치 허용할 경우 가능)

3. 논리적(로우 기반) 로그 복제
- 논리적 로그: sql문이 아닌 로우 단위의 데이터베이스 변경사항을 기록한 로그
- 로우 로그는 모든 칼럼의 새로운 값을 포함
- 삭제된 로우 로그는 로우의 고유 식별 정보를 포함 - 테이블의 기본키
- 여러 로우를 수정하는 트랜잭션은 여러 로그 레코드를 생성한 다음 -> 트랜잭션이 커밋됐음을 레코드에 표시