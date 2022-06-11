# JWT(Json Web Token)

: JWT(Json Web Token)란 Json 포맷을 이용하여 사용자에 대한 속성을 저장하는 Claim 기반의 Web Token



## 구조

jwt 는 세 부분으로 이뤄져있으며, 각 부분은 Base64url 로 인코딩되어있다. 

(여기서 오해하면 안되는 점은, 암호화가 되어 있는 것이 아니다.)

구성부인 header, payload, signature 를 차례로 보자.

`Base64 인코딩의 경우 “+”, “/”, “=”이 포함되지만 JWT는 URI에서 파라미터로 사용할 수 있도록 URL-Safe 한 Base64url 인코딩을 사용`



### Header

헤더는 typ 과 alg 로 구성되어 있다.

- typ: 토큰의 타입 지정
- alg : 알고리즘 방식 지정 
  - signature 및 토큰 검증에 사용
  - **암호화를 위한 목적이 아닌, signature 해싱하기 위한 알고리즘**이다!

```
{ 
   "alg": "HS256",
   "typ": JWT
 }
```



### PayLoad

페이로드에는 토큰에서 사용할 정보의 조각인 Claim 이 담겨있다.

 클레임에는 JSON 형태로 다수의 정보를 넣을 수 있다. 

- Registered Claim (등록된 클레임)

  - 토큰의 정보를 표현하기 위해 이미 정해진 종류의 데이터로, 모두 선택적으로 작성이 가능

    ```
    iss: 토큰 발급자(issuer)
    sub: 토큰 제목(subject)
    aud: 토큰 대상자(audience)
    exp: 토큰 만료 시간(expiration), NumericDate 형식으로 되어 있어야 함 ex) 1480849147370
    nbf: 토큰 활성 날짜(not before), 이 날이 지나기 전의 토큰은 활성화되지 않음
    iat: 토큰 발급 시간(issued at), 토큰 발급 이후의 경과 시간을 알 수 있음
    jti: JWT 토큰 식별자(JWT ID), 중복 방지를 위해 사용하며, 일회용 토큰(Access Token) 등에 사용
    ```

- Public Claim 공개 클레임

  - 사용자 정의 클레임으로, 공개용 정보를 위해 사용된다. 

- Private Claim 비공개 클레임

  - 이 또한 사용자 정의 클레임이며, 서버와 클라 사이의 임의로 지정한 정보를 저장



### Signature(서명)

서명은 **토큰을 인코딩하거나 유효성 검증을 할 때** 사용하는 고유한 암호화 코드.

서명 과정은 비밀키를 가진 사람은 서명할 수 있고, 공개키를 갖고 있는 사람은 해당 데이터를 검증할 수 있다!





## 어떻게 사용할까?

- 클라이언트에서 인증서버로 권한을 요청
- 인증 서버에서 JWT 를 생성하여 클라이언트에게 반환
- 클라이언트는 JWT 를 가지고 권한에 맞는 리소스에 접근 가능
- 유효 기한을 두거나, 로그아웃 시 폐기 처리하여 유효하지 않은 JWT로 요청이 들어오는 것을 막을 수 있다.



## 장단점

- 장점
  - 인증에 필요한 모든 정보가 토큰에 포함되어 있음
  - 다른 토큰들에 비해 크기가 작아 부담이 낮음
  - 수평 스케일링에 용이
  -  msa 환경에서 개별 서비스에서 인증 해결 가능
- 단점
  - payload 에 싣는 데이터가 많아지면 JWT 크기가 커짐
  - 암호화가 아니기에 정보는 노출이 가능하기에, payload 에 중요 정보를 넣지 못한다.



## Access Token 과 Refresh Token

 refresh token 은 jwt 의 단점을 극복할 수 있다!

jwt 가 탈취당하면, 다른 사용자가 해당 토큰 사용자의 정보에 접근이 가능하기 때문!

-> access token 과 별도로 refresh token 을 둔다.

- access token 의 만료기간은 짧게, refresh token 은 길게!

#### 사용

- access token 으로 요청 보냈을 때
  - 만료되었다면
    - refresh token 을 확인 후 새로운 access token 발급



