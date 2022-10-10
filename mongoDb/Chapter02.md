# 몽고 DB 기본



- 데이터의 기본단위 - 도큐먼트
- 컬렉션 - 동적 스키마가 없는 테이블
- 몽고DB 단일 인스턴스는 자체적인 컬렉션을 갖는 여러 개의 독립적인 DB를 호스팅
- 모든 도큐먼트는 고유 특수키인 _id 를 갖는다
- 몽고 쉘과 함께 배포된다





## 2.1 도큐먼트

몽고 DB의 핵심은 정렬된 키와 연결된 값의 집합으로 이뤄진 도큐먼트

- 대부분의 언어는 맵, 해시, 딕셔너리 같이 도큐먼트를 자연스럽게 표현하는 자료구조를 갖는다. 
  Ex. 자바스크립트는 객체

```
{"greeting": "hello~", 
"count": 5,
"count": "5", // wrong
}
```



## 2.2 컬렉션

컬렉션: 도큐먼트의 모음 
도큐먼트가 RDB의 row 에 해당한다면 컬렉션은 테이블에 대응



### 2.2.1 동적 스키마

컬렉션은 동적 스키마를 갖는다. 
하나의 컬렉션 내 도큐먼트가 모두 다른 구조를 가질 수 있다! 

```
{"hello": 3}
{"bye": "good~"}
```

도큐먼트의 키, 키 갯수, 데이터형의 값 모두 다를 수 있다. 그렇다면 왜 하나 이상의 컬렉션이 필요한가?

- 같은 컬렉션에 다른 종류 도큐먼트를 저장하면 개발자와 관리자가 번거로움
  - 각 쿼리가 특정 스키마를 고수하는 도큐먼트를 반환하는지, 혹은 쿼리한 코드가 다른 구조의 도큐먼트를 다룰 수 있는지
- 컬렉션별 목록을 뽑으면 한 컬렉션 내 특정 데이터형 별로 쿼리한 것보다 빠르다.
- 데이터 지역성에 좋다
- 인덱스를 만들면 도큐먼트는 특정 구조를 가져야한다.

스키마를 만들고 관련 유형의 도큐먼트를 그룹화하는 것이 좋다! 
유효성 검사, 객체-도큐먼트 매핑 등 유용!


### 2.2.2 네이밍

컬렉션은 이름으로 식별되지만 일부 제약조건이 있다. (빈 문자열, 예약어 겹치거나, ~~~)

#### 서브 컬렉션

서브컬렉션은 네임스페이스에 `.`를 사용해서 컬렉션을 체계화환다. ex) `blog.authors`



## 2.3 데이터베이스

몽고 DB는 컬렉션에 도큐먼트를 그룹화 + DB에 컬렉션을 그룹지어 놓는다.

- admin
  - admin DB는 인증과 권한 부여 역할
- local
  - 단일 서버에 대한 데이터를 저장
  - 복제 셋에서 local 은 복제 프로세스에 사용된 데이터를 저장
- config
  - 샤딩된 몽고 DB는 config DB를 사용해 샤드의 정보를 저장한다.
