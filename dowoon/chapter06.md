# CSRF, 클릭재킹, 오픈 리다이렉트
대표적인 수동적 공격인 CSRF, 클릭재킹, 오픈 리다이렉트에 대해 알아본다.

## CSRF
> CSRF(Cross Site Request Forgery)는 공격자가 준비한 함정에 의해 웹 애플리케이션이 원래 갖고 있는 기능이 사용자의 의도와는 상관없이 호출되는 공격이다.
 
과거 트위터와 같은 CNS에서도 CSRF 공격에 의해 악의적으로 내용이 업로드된 사례가 있다.

### CSRF 구조
사용자가 은행사이트를 방문하는 상황을 가정해보자.
은행 사이트에 대한 CSRF 공격 절차는 다음과 같다.
1. 사용자가 은행 사이트에 로그인
2. 로그인에 성공하면 세션 ID가 쿠키에 기록
3. 사용자가 공격자에게 송금하기 위한 악성 폼이 삽입된 피싱 사이트로 유도
4. 사용자의 쿠키와 함께 피싱 사이트에서 은행 사이트로 요청이 자동으로 전송
5. 은행 사이트 서버는 전송된 쿠키를 정상으로 간주해 요청을 처리

### CSRF 대책

#### CSRF 토큰
가장 효과적인 대책은 토큰(문자열)을 사용하는 것이다.
페이지 접근 요청을 받은 서버는 랜덤 문자열 토큰을 생성해 세션별로 서버에 보관한다.
그리고 보유한 토큰을 HTML에 삽입한다.
```html
<input
    type="hidden"
    name="CSRF_TOKEN"
    value="123abc456def7890"
/>
```
토큰을 사용하는 CSRF 방지는 **세션마다 다른 값의 토큰을 발행**해야한다.
서버는 요청에 포함된 토큰과 세션에 보관된 토큰이 일치하는 지를 통해 정상 요청인지 확인한다.

#### Double Submit Cookie
Double Submit 쿠키는 세션용 쿠키와는 달리 **랜덤 토큰 값을 가진 쿠키를 발행**하고 이 토큰을 사용해 정상적인 요청인지 확인하는 방법이다.
로그인 시 **1) 세션용 쿠키**, **2) HttpOnly 속성이 없는 토큰 값을 가진 쿠키** 두 가지를 발행한다.

앞서 토큰 방식과는 달리 Double Submit 쿠키는 토큰을 브라우저에서 처리하기 때문에 요청을 받는 서버에서 토큰을 보관하기 어려울 때 유용하다.

### SameSite Cookie
SameSite 쿠키는 **쿠키 전송을 동일한 사이트로 제한**한다. 
- ex) `alice.example.com`, `bom.example.com` → eTLD+1이 같으므로 동일한 사이트 간주

SameSite 속성은 다음 값을 설정할 수 있다.

| 속성 값   | 의미                                            |
|--------|-----------------------------------------------|
| Strict | 쿠키를 요청한 사이트와 동일한 사이트에서만 쿠키 전송                 |
| Lax    | 쿠키를 요청한 사이트와 동일한 사이트에서만 쿠키 전송 (GET 요청에 한해 허용) |
| None   | 사이트에 관계 없이 모든 요청에 쿠키를 전송                      |

SameSite 속성을 지정하지 않을 시 크롬, 엣지 등은 Lax가 기본값으로 설정된다.
그러나 Lax가 기본값 지정 시 웹 사이트 기능에 영향(서드파티 로그인, 크로스사이트 호환성 등)을 주기 때문에 초기 2분간은 Lax가 설정되지 않도록 동작한다.

### Origin 헤더
API를 제공하는 서버에서 Origin 헤더를 확인하면 허가되지 않은 출처의 요청을 금지할 수 있다.
```js
app.post("/remit", (req, res) => {
    if (!req.headers.origin && req.headers.origin !== "https://site.example") {
        req.status(401);
        req.send("허가되지 않은 요청");
        return;
    }
})
```

### CORS
CORS의 Preflight Request에서 요청 내용을 확인하여 `fetch`함수와 XHR 요청을 방지할 수 있다.
다만 Preflight Request는 요청 횟수가 늘어나므로 성능에 좋지 않다는 의견도 있다.

서버 측에선 허가된 출처로부터의 요청인지 X-Request-With: XMLHttpRequest 헤더가 추가되어 있는지 확인하여 CSRF 가능성을 차단할 수 있다.

## 클릭재킹
> 클릭재킹(Clickjacking)은 사용자가 의도하지 않은 클릭을 하도록 유도하여 공격자가 원하는 작업을 수행하게 하는 공격이다.

### 클릭재킹 구조
클릭재킹 공격은 iframe을 사용한 교차 출처 페이지의 삽입과 사용자에 의한 클릭에 의해 발생한다.
아래 발생 절차를 살펴보자.
1. 공격 대상의 웹 페이지를 iframe을 사용해 피싱 사이트와 중첩
2. CSS를 사용해 iframe을 투명하게 해 사용자에게 보이지 않게 함
3. 공격 대상의 페이지에서 중요한 기능을 하는 버튼이 피싱 사이트상의 버튼 위치와 중첩되도록 CSS 조정
4. 피싱 사이트에 접속한 사용자가 피싱 사이트에서 버튼을 클릭하도록 유도함
5. 사용자는 피싱 사이트에서 버튼을 클릭해도 실제로는 투명하게 겹쳐진 공격 대상의 페이지 버튼 클릭

### 클릭재킹 대책
클릭재킹을 방지하려면 iframe과 같이 프레임에 페이지를 삽입하는 것을 제한해야 한다.

#### X-Frame-Options
X-frame-Options 헤더가 추가된 페이지는 프레임 내부에 삽입이 제한된다.
- DENY: iframe에 삽입 불가
- SAMEORIGIN: 동일 출처에 한해 iframe에 삽입 가능
- ALLOW-FROM: 지정한 출처에 한해 iframe에 삽입 가능

#### CSP frame-ancestors
CSP의 frame-ancestors directive 또한 프레임 내부 페이지 삽입을 제한한다.
- `frame-ancestors 'self'` : 동일 출처에 한해 iframe에 삽입 가능
- `frame-ancestors 'none'` : iframe에 삽입 불가
- `frame-ancestors https://example.com` : 지정한 출처에 한해 iframe에 삽입 가능

## 오픈 리다이렉트
> 오픈 리다이렉트(open redirect)는 웹 리다리덱트 기능을 이용해 피싱 사이트 등 공격자가 준비한 사이트로 강제 이동시키는 공격이다. 

### 오픈 리다이렉트 구조
로그인 성공 시 쿼리 스트링 URL에 지정된 URL로 이동한다고 가정해보자.
- https://example.com/login?url=/mypage

그러나 오픈 리다이렉트의 취약점이 존재할 경우 다음과 같이 공격자에 의한 사이트로 이동하도록 설정될 수 있다.
- https://example.com/login?url=https://attacker.example.com

오픈 리다이렉트는 **요청에 포함된 URL 파라미터를 서버에서 리다이렉트 URL로 사용하기 때문에 발생**한다.
```js
const url = new URL(location.href);
const redirectUrl = url.searchParams.get("url");

if (!redirectUrl.match(/^https?:\/\//)) throw new Error("잘못된 URL");
location.href = redirectUrl;
```

### 오픈 리다이렉트 대책
오픈 리다이렉트를 방지하기 위해선 위부에서 입력한 URL이 올바른지 검증해야한다.

1. 이동 대상이 특정 URL로 한정되어있는 경우
```js
if (redirectUrlStr === "/mypage" || redirectUrlStr === "/schedule") {
    ...
}
```
2. 동일 출처로만 이동을 제한하는 경우
```js
if (redirectUrlObj.origin === pageUrlObj.origin) {
    ...
}
```
