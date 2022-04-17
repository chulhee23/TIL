# 3. 도큐먼트 생성, 갱신, 삭제



- 컬렉션에 
  - 새 도큐먼트 추가하기
  - 삭제하기
  - 기존 갱신
- 연산 수행 시 안전성과 속도 중 맞는 수준 선택



## 3.1 도큐먼트 삽입

- insertOne
- insertMany
  - 486MB까지 일괄 삽입 제한
  - 넘으면 알아서 분할해서 삽입
  - 정렬 연산 여부에 따라 오류 대응이 다르다.
  - 정렬이 기본 값
    - 정렬된 삽입 -> 삽입 오류가 될 경우, 해당 insert 까지만 시도하고 끝
    - 비정렬 -> 모든 도큐먼트 삽입을 시도



### 3.1.2 삽입 유효성 검사

삽입된 데이터에 최소한의 검사를 수행

- _id 값이 없으면 추가,
- 모든 도큐먼트는 16MB보다 작아야한다.
  - 크기 제한은 상대적

### 

## 3.2 도큐먼트 삭제

- deleteOne

```
	db.movies.deleteOne({"_id": 2})
```

_id 는 유니크해서 하나만 삭제도 가능, 일치하는 필터 중 가장 먼저 나오는 도큐먼트를 삭제하게 되는데, 
이는 도큐먼트 삽입된 순서, 어떤 갱신이 이뤄졌는지, 어떤 인덱스 지정하는지 등에 따라 달라진다.

- deleteMany
  - 필터에 맞는 모든 도큐먼트 삭제

- drop
  - 컬렉션의 모든 도큐먼트 제거

## 3.3 도큐먼트 갱신

- updateOne
- updateMany
- replaceOne
  - 완전히 새로운 것으로 치환
  - _id 를 지정해주는 편이 좋다

갱신은 원자적이다. 2개가 동시에 발생하면, 서버에 먼저 도착한 요청이 먼저 적용!

이러한 동작을 원하지 않는다면 도큐먼트 버저닝 패턴 사용



### 3.3.2 갱신 연산자

특정 부분만 갱신하는 경우가 많은데, 부분갱신에는 **원자적 갱신 연산자**를 사용한다.

`$inc` 연산자를 사용해서 원자적으로 증가도 가능!

```
db.analytics.updateOne({"$inc", {"pageViews": 1}})
```

- **$set 제한자 사용하기**

  - 필드가 존재하지 않으면 새 필드가 생성된다. 
  - update or insert
  - 타입 변환도 가능!

  - 키를 추가, 변경, 삭제할 때는 $ 제한자를 사용해야한다. 갱신하다가 키 값을 다른 값으로 바꾼다면 오류가 발생한다!

- **증가와 감소**

  - $inc 연산자는 이미 존재하는 키의 값을 변경하거나 새 키를 생성.

- 배열 연산자

  - 배열을 다루는 데 갱신 연산자 사용 가능

  - 리스트에 대한 인덱스 지정 + 셋처럼 이중으로 사용가능

  - $slice 와 $push 를 결합하면 배열을 특정 크기로 만들 수도 있다.

  - ```
    db.movies.updateOne({"genre": "horror",
    	{"$push": 
    		{"top10": {$each: ["hello", "saw"], 
    		  "$slice", -10,
    	 	 "$sort": {"rating": -1}
         }
       }
     }
    })
    ```

  - $slice 나 $sort 를 $push 와 함께 쓰려면 $each 도 필요

  - 배열을 집합처럼 쓰려면 $ne 를 추가해서 사용
    - $addToSet 도 사용 가능 ($ne 사용 못하거나, 무슨 일이 일어났는지 더 잘 알고 싶을때?)

- 요소 제거하기

  - 배열을 큐나 스택으로 쓰려면 $pop을 사용 

    - (key: -1) 을 주면 큐처럼 사용 가능!

    - ```
      {"$pop": {"key": 1}} // 마지막 요소 제거
      {"$pop": {"key": -1}} // 첫 요소 제거
      ```

  - 배열의 위치 기반 변경

    - 게시글의 첫 번째 댓글 투표수 증가를 위해선?

      - ```
        db.blog.updateOne({"post": post_id},
                         {"$inc": {"comments.0.votes": 1}})
        ```

    - 그런데 쿼리해서 검사해보지 않는다면 배열의 몇 번째 요소를 변경할지 알 수 없다.

    - 위치 연산자를 활용해보자.

    - ```
      db.blog.updateOne({"comments.author": "John"},
                       {"$set": {"comments.$.author": "Jim"}})
      ```

- 배열 필터를 이용한 갱신

  - arratFilters 를 도입해 특정 조건에 맞는 배열 요소 갱신 가능!



### 3.3.3 갱신 입력 (Upsert)

갱신 입력은 특수한 형태를 갖는 갱신!

갱신 조건에 맞는 도큐먼트가 없다면, 쿼리도큐먼트와 갱신 도큐먼트를 합쳐 새로운 도큐먼트를 생성한다.

```javascript
blog = db.analytics.findOne({url: "/blog"})

if (blog) {
  blog.pageviews++;
	db.analytics.save(blog);
}
else {
	db.analytics.insertOne({url: "/blog", pageviews: 1})
}
```

가 아닌, 원자적으로 처리하자!

```
db.analytics.updateOne(
    {"url": "/blog"}, 
    {"$inc": {"pageviews": 1}},
    {"upsert": true}
)
```

$setOnInsert 를 사용한다면 기본값도 설정 가능!

- 저장 셸 보조자
  - `save` : save or update
  - return null or object



### 3.3.4 다중 도큐먼트 갱신

updateMany: 조건에 맞는 도큐먼트 모두 수정 



### 3.3.5 갱신한 도큐먼트 반환

findOneAndUpdate 메서드는 기본적으로 도큐먼트의 상태를 수정하기전에 반환하므로 경쟁상태를 예방할 수 있다.

#### 프로세스의 컬렉션 예시

```
{
    "id": ObjectId(),
    "status": "state", // "READY", "RUNNING", "DONE"
    "priority": N
}
```

우선순위가 높은 ready 상태의 프로세스를 실행하고, status를 ready -> done 으로 바꾼다. 

```javascript
var cursor = db.processes.find({"status": "READY"});
ps = cursor.sort({"priority": -1}).limit(1).next();

db.processes.updateOne({"_id": ps._id}, {"$set": {"status": "RUNNING"}});

// ~~~

db.processes.updateOne({"_id": ps._id}, {"$set": {"status": "DONE"}});
```

경쟁상태 가능..!

update 가 원자적으로 처리된다면 어떨까!

findOneAndUpdate -> 한 번의 연산으로 항목을 반환하고 갱신할 수 있다.

```javascript
ps = db.processes.findOneAndUpdate(
  {"status": "READY"},
	{"$set": {"status": "RUNNING"}},
	{"sort": {"priority": -1},
	"returnNewDocument": true}
)

// ~~~

db.process.updateOne({"_id": ps._id}, {"$set": {"status": "DONE"}})
```



이 외에도 findOneAndReplace, findOneAndDelete 등이 존재

