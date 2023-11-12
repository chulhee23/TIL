# Redis Cluster

분산 시스템에 따라오는 문제

- 부분장애
- 네트워크 실패
- 데이터 동기화
- 로드밸런싱 (또는 Discovery)
- 개발 및 관리의 복잡성

그럼에도 불구하고 분산 시스템으로 인한 trade-off 고려해서 적합하다면 사용한다.



## Redis Cluster 가 제공하는 것

- 여러 노드에 자동적인 데이터 분산
- 일부 노드의 실패나 통신 단절에도 계속 작동하는 가용성
- 고성능을 보장하면서 선형 확장성을 제공

### 특징

- Full-mesh 구조 (모든 노드끼리 통신)

- cluster bus 라는 추가채널(port) 사용

  - 노드 간에 상태 정보와 메시지를 교환하는 데 사용되는 내부 통신 채널
  - 이 클러스터 버스를 통해 노드들은 서로 통신하며 클러스터 전체의 상태를 동기화하여 고가용성을 유지하고 데이터의 일관성을 보장

- gossip protocal 사용

  - 너무 많은 노드가 서로 통신할 때 통신의 갯수를 줄이기 위함
  - 각 노드는 주기적으로 몇몇 다른 노드들과 랜덤하게 정보를 교환. 
    이때 노드들은 상대 노드들로부터 가지고 있지 않은 정보를 받아오거나, 자신이 가지고 있는 정보를 전달합니다. 이러한 정보 교환을 통해 시스템 전체에 정보가 분산되고, 모든 노드들이 서로의 정보를 알게 됩니다.
  - 

- Hash slot 을 사용한 키 관리

- DB0만 사용가능 

  - 레디스 단일 노드는 하나의 노드로만 구성되어 있으며, 여러 개의 DB를 사용 가능. 
    ```
    SELECT 0  # 0번 DB 선택
    SET key1 value1
    GET key1
    
    SELECT 1  # 1번 DB 선택
    SET key2 value2
    GET key2
    ```

    

- multi key 명령어가 제한됨

- 클라이언트는 모든 노드에 접속






