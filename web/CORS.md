# Cross Origin Resource Sharing
- SOP[(Same Origin Policy](./SOP.md))와 함께보자.
- SOP에 의한 불편함을 호소하는 사례가 많이 생기고 Cross Domain 요청을 허용해달라는 수요가 많아지자 우회하는 방법들이 생겨났는데 이것들을 그냥 두고 볼 수는 없으니까 공식적으로 스펙을 만들어버림. (W3C)
- 단순하게 요청 헤더에 CORS관련 옵션을 넣어주고 받는 서버에서는 응답에 CORS를 허용한다는 헤더를 담아 보내주는 것.
- 이 요청은 최초 OPTION 헤더를 달고 서버에 날아가게 되고, 그 응답이 허용할 때만 실제 요청을 보내는 식으로 이중 호출하도록 되어 있음.

## 실제
### 요청
클라이언트가 서버로
Origin 이라는 헤더에 출처의 정보를 담아서 보낸다.
```
Origin: https://my-blog.com
```

### 응답
서버가 클라이언트로
Access-Control-Allow-Origin 헤더에 리소스에 접근하는 것이 허용된 출처를 담아 보낸다.
```
Access-Control-Allow-Origin: http://my-blog.com
```

## CORS의 두가지 시나리오.
- 첫번째는 preflight request(예비 요청)을 보내서 CORS 지원 여부를 확인 후 보내는 것이다.
- 두번째는 Simple Request(단순 요청)을 바로 보내서 예비 요청없이 처리하는 것이다. 
- 예비 요청은 아마도 위험이 크지 않다고 판단하는 조건이 있는 것 같다 그 조건은 아래와 같다.
    - GET, HEAD, POST중 하나일 것.
    - Accept, Accept-Language, Content-Language, Content-Type, DPR, Downlink, Save-Data, Viewport-Width, Width를 제외한 헤더를 사용하지 않을 것.
    - 만약 Content-Type를 사용하는 경우에는 application/x-www-form-urlencoded, multipart/form-data, text/plain만 사용할 것.
