# CORS

## SOP

`SOP(Same Origin Policy)` 란 document의 Origin이 Cross Origin 관계에 있는 리소스와 상호작용하는 것을 제한하는 **보안 메커니즘**이다.

> Origin
> 

`Origin`이란 브라우저가 서버로 부터 다운로드 받은 document에 대한 출처이다.

- 브라우저는 `Protocol , Host, Port`를 기준으로 Origin을 구분한다.
    - `Same Origin` : Protocol, Host, Port 가 모두 동일한 두 Origin
    - `Cross Origin` : Protocol, Host, Port 셋 중 하나라도 다른 두 Origin

> SOP 정책의 적용 예시
> 
1. `A 서버의 document에서 A와 Cross-Origin 관계인 B 서버의 자원을 받아올 경우`
    - 브라우저가 `https://www.guswns.com` 서버로부터 다운받은 document이 아래와 같다
        
        ```jsx
        var oReq = new XMLHttpRequest();
        oReq.open("GET", "https://api.guswns.com/profile"); // SOP 정책이 적용된다.
        
        window.open("https://popup.guswns.com")
        
        <ifram src="https://iframe.guswns.com"></iframe>
        ```
        
    - XMLHttpRequest 메소드를 이용해서 `"https://api.guswns.com/profile"` 사이트에서 JSON 데이터를 받아올 때 SOP 정책이 적용된다. (`https://www.guswns.com` 와 `https://api.guswns.com/profile` 는 Cross-Origin 관계)
2. `브라우저의 localStorage는 Same-Origin 끼리만 접근할 수 있다.`

> SOP 정책이 없다면?
> 

`악성 사이트에서 사용자의 쿠키 속 인증 정보를 악용할 수 있기 때문이다`

1. 공격자가 만든 악성 사이트에 접속
2. 사용자의 브라우저는 악성 사이트의 document를 다운로드
3. 악성 document에서 사용자 브라우저 속 쿠키에 있는 인증 정보에 접근 
4. **사용자 인증 정보를 통해 Google 계정에 접근 가능! (SOP 정책이 적용되지 않는다면 제 3 사이트의 사용자 정보를 탈취당할 수 있다)**

> SOP는 보안 정책이다
> 
1. 브라우저가 로드한 document가 Cross-Origin 관계를 가진 Google 서버의 리소스에 접근한다.(request를 보낸다)
2. Google 서버에서 response를 보낸다
3. 브라우저는 현재 document의 Origin과 Google 서버의 response의 Origin을 비교한다.
4. SOP 정책에 의해 두 Origin이 Cross-Origin 관계라면 브라우저는 해당 response를 읽어들이지 못하고 콘솔에 `".. has been blocked by CORS policy"` 라는 에러 메시지를 출력한다.

> SOP는 CSRF 공격을 막지 못한다.
> 

**SOP 정책은 Cross-Origin 관계의 서버로 부터 전달받은 response를 read하지 못하게 막지만, Cross-Origin 관계의 서버로 request를 보내는 write연산을 막지 않는다.**

1. 공격자가 만든 악성 사이트에 접속
2. 사용자의 브라우저는 악성 사이트의 document를 다운로드
3. 악성 document에서 사용자 브라우저 속 쿠키에 있는 인증 정보에 접근 
4. **사용자 인증 정보를 통해 Google 계정에 정보를 수정하는 POST/PUT request를 보낼 수 있다. (`write를 막지 못한다`)**
5. SOP 정책에 의해 Google 서버의 response를 read하지 못한다. (`read를 막는다`) ****

Google 서버는 CSRF 공격에 대한 대책을 마련해야 한다.

## CORS

`CORS(Cross Origin Resource Sharing)` 이란 Cross-Origin끼리 자원을 공유할 수 있도록 하는 정책이다.

> CORS는 에러가 아니다
> 

콘솔의 에러 메시지는 사실 `“Cross-Origin간에 자원을 공유하기 위해서 CORS를 허용해야 한다"`라는 의미의 메시지이다.

- 오직 브라우저에서만 발생하는 문제이다. Cross-Origin관계의 서버로 request를 브라우저가 아닌 Postman이나 Backend에서 보내면 아무 문제없이 response를 읽어들일 수 있다)

> CORS 정책의 등장 배경
> 
1. SOP 보안 정책에 의해 Cross-Origin끼리의 자원 공유가 제한되었다.
2. 웹 생태계가 다양해지면서 여러 서비스(서버)들간에 자유로운 데이터 교환이 필요해졌다
3. CORS 정책이 등장하기 이전에는 JSONP 로 SOP 정책을 우회하는 방식을 사용했었다. (JSONP는 모든 Origin 대상으로 SOP 정책을 무력화시키기 때문에 보안상 문제가 있다)
4. **합의된 Cross-Origin들 간에 합법적으로 자원 공유를 허용해주는 CORS 정책이 등장하게 되었다**

> CORS 정책 활성화 하기
> 
1. 브라우저의 request를 받은 서버는 HTTP request 헤더의 `Access-Control-Allow-Origin` 필드에 자원 공유를 허가할 Cross-Origin 정보를 담아야 보내야 한다.
2. 브라우저는 response를 받고 헤더 속 `Access-Control-Allow-Origin` 필드에 현재 document의 Origin이 존재하다면 response를 읽어들일 수 있다. (존재하지 않다면 `".. has been blocked by CORS policy"` 라는 에러 메시지를 출력)

CORS 정책에서는 브라우저의 Request를 `Simple Request`, `Preflight Request` 로 분류한다.

> `Simple Request`
> 
- 브라우저가 서버에게 예비 request를 보내지 않고, 본 request를 바로 보내는 방식
    1. 브라우저가 서버에게 request를 보낸다 (HTTP Request Method는 `GET, HEAD, POST` 중 하나여야 한다.)
    2. 서버가 response를 보낸다.
    3. 브라우저는 서버의 response 헤더의 `Access-Control-Allow-Origin` 필드를 확인하여 CORS 정책이 활성화 되었는지 검사한다.
- Simple Request의 문제점
    - 브라우저가 Simple Request 방식으로 request를 보내면, 서버는 일단 request에 대한 요청을 수행하기 때문에 데이터가 변경될 수 있다.

> `Preflight Request`
> 
- 서버에게 본 request를 보내기 전에 예비 request(preflight request)를 보내는 방식
    1. 브라우저가 서버로 OPTIONS Method 방식으로 예비 request를 보낸다.
    2. 서버는 response 헤더의 `Access-Control-Allow-Origin` 필드 속에 자원 공유를 허가할 Cross-Origin 목록을 담아 브라우저로 보낸다
    3. 브라우저는 서버의 response 헤더의 `Access-Control-Allow-Origin` 필드에 예비 request를 보낸 Origin이 포함되는지 확인한다.
    4. request를 보낸 Origin에 대해 CORS 정책이 활성화된 서버인 경우, 본 request를 보낸다.

```jsx
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER
```

서버는 response에 위와 같이 접근을 허가할 HTTP Method, 헤더 정보, Cross-Origin 목록을 담아 보낸다.