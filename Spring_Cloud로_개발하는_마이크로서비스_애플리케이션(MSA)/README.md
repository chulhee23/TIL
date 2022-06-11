# MSA

외부 서비스와 통신 시 연쇄 오류가 발생했을 때 어떻게 처리해야할까?

A Service -> B Service 로 요청을 그만 보내야함. => Fall Back 메소드가 필요하다!



### CircuitBreaker

- 장애가 발생하는 서비스에 반복적인 호출이 되지 못하게 차단
- 특정 서비스가 정상적으로 동작하지 않을 경우 다른 기능으로 대체 수행하여 장애를 회피
  - CircuitBreaker open 되었다면? circuit breaker 에서 우회처리 한다!



