# XSS
## 능동적 공격과 수동적 공격
### 능동적 공격
- 공격자가 웹 애플리케이션에 직접 공격 코드를 보내는 공격 유형
- SQL 인젝션, OS 명령 인젝션 등

### 수동적 공격
- 공격자가 준비한 피싱 사이트를 이용해 웹 애플리케이션에 방문한 사용자가 공격 코드를 실행하도록 하는 공격
- 페이지 접근, 링크 클릭 등에 의해 공격 트리거
- XSS, CSRF, 클릭재킹, 오픈 리다이렉트 등
- **능동적 공격은 서버에서 대책을 세워야하는 반면 수동적 공격은 프론트엔드에서만 가능한 부분 존재**

## XSS
> XSS(Cross Site Scripting)는 웹 애플리케이션의 취약점을 이용해 악성 스크립트를 실행하는 공격임.
- 교차 출처 페이지에 의한 자바스크립트는 SOP에 의해 실행되지 않는 반면 **XSS는 공격 대상 페이지에서 실행됨으로 막을 수 X**

### XSS 구조
- 공격자가 페이지 HTML에 악성 스크립트를 삽입해 사용자가 스크립트를 실행하도록 만드는 공격
- 예를 들어 하단 URL을 쇼핑 사이트 제품 검색 화면 이동 화면 URL이라고 가정
```text
https://site.example/search?keyword=보안
```
- 'keyword=보안'은 HTML에 삽입
```html
<div id="keyword">검색 키워드: 보안</div>
<div id="result">...</div>
```
- 이때 아래와 같은 키워드로 요청했다고 가정
```text
https://site.example/search?keyword=<img src onerror="location.href=`https://attacker.example`" />
```
- 이미지 요소의 src가 제대로 설정되지 않아 에러 처리를 위해 onerror 속성에 설정된 자바스크립트가 실행되며 강제로 다른 웹사이트로 이동

### XSS 위협
- XSS는 아래와 같은 문제를 일으킴
1. 보안 정보의 유출
2. 웹 애플리케이션 변조
3. 의도치않은 조작
4. 사용자로 위장
5. 피싱

### 세 가지 XSS
#### 1) 반사형 XSS
- 공격자가 준비한 함정에서 발생하는 요청에 잘못된 스크립트를 포함하는 HTML을 서버에서 생성해 발생하는 XSS
- 요청 내용에 잘못된 스크립트가 포함된 경우에만 발생
- 지속성이 없으므로 비지속성 XSS라고도 함

#### 2) 저장형 XSS
- 공격자가 폼 등으로부터 제출한 악성 스크립트를 포함하는 데이터가 서버에 저장되고 페이지에 반영되어 발생하는 XSS
- 데이터베이스에 등록된 데이터가 반영되는 페이지를 보는 모든 사용자에게 영향을 미침
- 반사형 XSS와는 달리 정상적인 요청을 하는 사용자에게도 영향을 줄 수 있음 (지속형 XSS)

#### 3) DOM 기반 XSS
- 자바스크립트로 DOM을 조작할 때 발생하는 XSS
- 서버를 통하지 않아 감지가 어려우며 개발자 도구로 코드를 볼 수 있다는 취약점을 이용

##### DOM
- DOM은 HTML을 조작하기 위한 인터페이스이며 브라우저는 HTML 구문을 해석해 DOM 트리를 생성함
- DOM 트리를 변경할 때는 자바스크립트를 사용
```js
document.body.innerHTML = `<a href="https://attacker.example">새로운 사이트로 이동</a>`;
```

##### DOM 기반 XSS 예시
- `https://site.example/#hello` URL이 있다고 가정
- 여기서 #을 제거하고 hello라는 문자열을 DOM으로 삽입
```js
const message = decodeURIComponent(location.hash.slice(1));
document.getElementById("message").innerHTML = message;
```
- `https://site.example/#<img src=x onerror="location.href=`https://attacker.example`" />` URL을 요청한다고 가정
```html
<div id="message">
    <img src=x onerror="location.href=`https://attacker.example`" />
</div>
```
- `innerHTML`을 이용하여 DOM 조작한 것이 XSS의 원인
- DOM 기반 XSS는 브라우저의 기능을 사용할 때 발생하며 브라우저 기능은 소스와 싱크로 분류
- 소스: DOM 기반 XSS의 원인이되는 `location.hash` 문자열과 같은 것
  - `location.href`, `location.search`, `location.hash`, `document.referrer`, `postMessage`, `Web Storage` 등
- 싱크: 소스의 문자열에서 자바스크립트를 생성하고 실행하는 것
  - `innerHTML`, `eval`, `location.href`, `document.write`, `jQuery()` 등

### XSS 대책
- 실제 애플리케이션을 개발할 땐 XSS를 자동으로 예방해주는 라이브러리나 프레임워크를 사용

#### 1) 문자열 이스케이프 처리
- 이스케이프 처리란 프로그램에 특별한 의미를 갖는 문자나 기호를 특별하지 않은 의미로 변환 처리하는 작업을 의미함
- '<', '>' 기호를 인식하여 <srcipt>를 HTML 요소로 해석  -> \&lt;srcipt&gt
```js
const escapeHTML = (str) => {
    return str.replace(/&/g, "&amp;")
              .replace(/</g, "&lt;")
              .replace(/>/g, "&gt;")
              .replace(/"/g, "&quot;")
              .replace(/'/g, "&#x27;");
};
```
#### 2) 속성값의 문자열을 쌍따옴표로 감싸기
- HTML 속성값에 문자열을 넣으면 이스케이프 처리로는 예방 X
- 퀴리 스트링 값에 onmouseover=alert(1)과 같은 문자열 저장 시 사용자가 마우스를 올리면 alert(1) 실행됨
```html
<input type="text" value=onmouseover=alert(1) />
```
- 쌍따옴표로 묶으면 keyword 값이 단순한 문자열로 처리됨
```html
<input type="text" value="onmouseover=alert(1)" />
```
- 그러나 "onmouseover='alert(1)'와 같이 적을 경우 value가 빈 배열로 들어가 여전히 공격 가능함
```html
<input type="text" value="" onmouseover='alert(1)' />
```
- 쌍따옴표를 &quot;로 이스케이프 처리하면 문제 해결 가능 
```html
<input type="text" value="&quot; onmouseover='alert(1)'" />
```

#### 3) 링크의 URL 스키마를 http/https로 제한하기
- \<a> 요소의 href 속성은 이스케이프와 쌍따옴표로 처리하는데 한계가 있음
```js
const url = new URL(location.href).searchParams.get("url");
const a = document.querySelector("#my-link");
a.href = url;
```
- 위와 같은 코드의 경우 `https://site.example/?url=javascript:alert(1)`와 같은 URL을 입력하면 공격 가능
```html
<a id="my-link" href="javascript:alert(1)">링크</a>
```
- a태그의 href 속성에 http 혹은 https로 좁혀 해결할 수 있음
```js
const url = new URL(location.href).searchParams.get("url");
if (url.match(/^https?:\/\//)) {
    const a = document.querySelector('#my-link');
    a.href = url;
}
```

#### 4) DOM 조작을 위한 메서드와 프로퍼티 사용하기
- DOM 기반 XSS는 `innerHTML` 등을 사용할 때 발생함
```js
const txt = document.querySelector("#txt").value;
const list = txt.split(",");

const el = "<ul>";

```
- 브라우저가 DOM을 조작할 때 HTML로 해석하는 API 사용을 피하면 DOM 기반 XSS를 예방할 수 있음

#### 5) 쿠키에 HttpOnly 속성 사용하기
- 로그인이 필요한 웹 서비스의 경우 세션 정보를 쿠키에 저장할 때가 많음
- XSS 취약성이 존재하면 쿠키 값이 유출돼서 공격자가 사용자로 위장할 수 O
- HttpOnly 속성 부여 시 자바스크립트로 쿠키 값을 가져올 수 X

#### 6) 프레임워크 기능
- 다양한 프로그래밍 언어나 프레임워크 중에는 XSS를 자동으로 예방 (React, Vue.js Angular 등)
- `dangerouslySetInnerHTML` 사용 시 XSS 취약성 발생

#### 7) DOMPurify 라이브러리
- XSS로부터 무해한 HTML을 제외하고 일부 문자열만 제거하는 라이브러리
- `santitize` 메서드를 사용해 XSS로 작성된 문자열 무효화 가능
```js
const clean = DOMPurify.sanitize(dirty);
```

#### 8) Sanitizer API
- 브라우저의 API로 DOMPurify와 비슷한 기능을 제공
```html
<script>
  const el = document.querySelector("div");
  const unsafeString = decodeURIComponent(location.hash.slice(1));
  const sanitizer = new Sanitizer();
  el.setHTML(unsafeString, sanitizer);
</script>
```

## Content Security Policy
- CSP(Content Security Policy)는 XSS와 같이 악성 코드를 퐇마한 인젝션 공격을 감지해 피해를 막는 브라우저 기능임.
### CSP 개요
- CSP는 Content-Security-Policy 헤더를 응답에 포함해 활성화할 수 있음.
```text
Content-Security-Policy: script-src *.trusted.example
```
- CSP 헤더에 지정된 script-src *.trusted.com와 같은 값을 **policy directive** 또는 **directive**라고 함
- directive에 지정되지 않은 호스트명의 서버에는 자바스크립트 파일을 불러오지 않음.

#### 대표적인 directive
| directive                 | 의미                                                 |
|---------------------------|----------------------------------------------------|
| script-src                | 자바스크립트 등 스크립트 실행 허용                                |
| style-src                 | CSS 등 스타일 적용 허용                                    |
| img-src                   | 이미지 불러오기 허용                                        |
| media-src                 | 사운드, 영상 불러오기 허용                                    |
| connect-src               | XHR과 fetch 함수 등 네트워크 접근 허용                         |
| default-src               | 지정되지 않은 directive 전체 허용                            |
| frame-ancestors           | iframe 등 현재 페이지에 삽입 허용                             |
| upgrade-insecure-requests | http://로 시작하는 URL 리소스를 https://로 시작하는 URL로 변환하여 요청 |
| sandbox                   | 콘텐츠를 샌드박스화하여 외부로부터 접근 제어                           |

#### 소스 키워드
- CSP에 지정할 수 있는 특별한 의미가 존재하는 소스 키워드들은 다음과 같음

| 키워드          | 설명                                                     |
|--------------|--------------------------------------------------------|
| self         | CSP로 보호하는 페이지와 동일 출처만 허용                               |
| none         | 모든 출처 허용하지 않음                                          |
| unsafe-inline | 인라인 스크립트 및 인라인 스타일 허용                                  |
| unsafe-eval  | eval 함수 허용                                             | 
| unsafe-hashes | DOM에 설정된 이벤트 실행 허용<br>그러나 인라인 스크립트, 자바스크립트:스키마 실행 허용 X |

### Strict CSP
- CSP를 적용한 페이지는 HTML 내 자바스크립트를 작성하는 인라인 스크립트 금지
- `unsafe-inline`을 사용하지 않고 안전하게 인라인 스타일 실행 허용을 위해 nonce-source, hash-source 사용 가능
- 구글은 호스트명을 지정하는 대신 nonce-source, hash-source를 사용한 **Strict CSP**를 추천

#### nonce-source
- `<script>` 요소에 지정된 랜덤 토큰이 CSP 헤더에 지정된 토큰과 일치하지 않으면 에러 발생

#### hash-source
- nonce-source와 비슷하지만 랜덤 토큰 대신 해시값을 사용
- HTML을 동적으로 변경할 수 없는 경우 요청마다 토큰을 바꿀 수 없으므로 nonce-source 대신 hash-source 사용

#### strict-dynamic
- `nonce-source`, `hash-source` 사용 시 인라인 스크립트 안전하게 실행 가능 하지만 동적인 `<script>` 요소 생성 금지
- strict-dynamic 키워드 사용 시 `<script>` 요소 동적 생성 가능
  - 단 DOM 기반 XSS의 싱크인 `innerHTML`과 `document.write` 기능 제한

#### object-src / base-uri
- object-src는 플래시와 같은 플러그인 제한하는 directive
- base-uri는 `<base>` 요소 제한하는 directive






