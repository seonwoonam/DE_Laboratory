# AQE, BroadCast join, DPP 테스트

- 해당 실험은 AQE, BroadCast join, DPP를 활성화 또는 비활성화 해보면서 스파크 내부에서 어떤 조인 전략을 취하는지 어떤 방식으로 최적화가 진행되는지를 알아보기 위한 테스트
- AQE& Broadcast & DPP 비활성화

## 실험조건
  - 2024 택시 데이터
  - 디멘션 테이블과 Join 후에 맨허턴 Borugh에서의 택시 픽업 Zone 수 찾기

## 실험결과
  - all off : 5.04초
    - sort merge join
  - AQE on / Broadcast join off / DPP off : 6.38초 
    - sort merge join 일어남
  - AQE off / Broadcast join on / DPP on : 1.8초 
    - BroadcastHashJoin
  - all on : 1.58초
    - BroadcastHashJoin

## 결론
  - DPP나 Broadcast join 기능을 꺼버리고 AQE만 실행해놓으면 조인전략개선 같은 기능을 사용할 수 없다. 그래서 위에 상황에서는 AQE 오버헤드만 늘어서 시간이 오히려 증가
  - AQE를 아무 상황에나 사용한다해서 나아지지 않는다. 파티션이 어느정도 많아야 적절한 파티션 병합기능이 활성화될 것이고, skew 혹은 join과정도 없다면 overhead만 늘어날 수 있다.
  - 데이터 쿼리가 아주 작고, 파티션 적으며, 플랜을 개발자가 알아야 할 때는 AQE 꺼도된다.