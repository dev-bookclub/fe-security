# 4. Origin에 의한 애플리케이션 간 접근 제한

## 4.1 애플리케이션 간 접근 제한의 필요성

- 개인 정보를 다루는 웹 애플리케이션 개발자는 외부에서 잘못된 방법으로 접속할 수 없도록 항상 주의해야 한다.

## 4.2 동일 출처 정책에 의한 보호

- 동일 출처 정책은 브라우저에 내장된 접근 제한 방식을 말한다.
- 브라우저는 웹 애플리케이션 사이에 출처라는 경계를 설정해 서로의 접근을 제한한다.

### 4.2.1 출처

- 다른 웹 애플리케이션 간의 접근을 제한하기 위한 경계를 `출처` 라고 한다.
  - 출처는 보통 스키마명, 호스트명, 포트 번호 의 구조를 가리킨다.
- 웹 애플리케이션의 출처가 같으면 동일 출처, 다르면 교차 출처라고 한다.

### 4.2.2 동일 출처 정책

- 일정한 조건에 따라 교차 출처 리소스에 접근을 제한하는 방식을 `동일 출처 정책`이라고 한다.

## 4.4 CORS

### 4.4.1 CORS 방식

- CORS는 교차 출처로 요청을 전송할 수 있는 방식.
- 응답에 포함된 HTTP 헤더에 접속해도 좋다는 허가가 주어진 리소스는 접근이 가능한데 이때 HTTP 헤더를 CORS 헤더라고 한다.

### 4.4.2 단순 요청

- CORS - safelisted로 정의된 HTTP 메서드와 HTTP 헤더
  - CORS - safelisted method
    - GET
    - HEAD
    - POST
  - CORS - safelisted request - header
    - Accept
    - Accept-Language
    - Content-Language
    - Content-Type
- 접근을 허가하는 출처를 브라우저에 전달하려면 Access-Control-Allow-Origin 헤더를 사용한다.
  - Access-Control-Allow-Origin 헤더에 하나 이상의 출처를 지정할 수 없다.
  - 다만 `*` 와일드카드를 사용하면 모든 출처의 접근을 허가할 수 있다.

### 4.4.3 Preflight Request

- 합의된 요청을 허가된 상태에서만 전송하는데 이 요청을 Preflight Request라고 한다.
- Preflight Request는 OPTIONS 메서드를 사용한다. 요청을 전송하는 출처 외에도 출처에서 이용하려는 메서드와 추가하고 싶은 HTTP 헤더를 전송해 교차 출처에서 사용 가능 여부를 확인한다.

### 4.4.4 쿠키를 포한하는 요청 전송

- 교차 출처의 서버로 쿠키를 전송할 때는 쿠키를 포함하는 요청을 전송한다고 명시해야만 한다. fetch 함수는 쿠키를 전송하기 위한 credentials 옵션이 있다.
- 교차 출처의 요청을 허가하기 위해 서버는 Access-Control-Allow-Credentials 헤더를 전송한다.

### 4.4.5 CORS 요청 모드

- 프런트엔드의 자바스크립트도 CORS를 설정할 수 있으며 브라우저에서도 CORS를 사용하지 않도록 지정 가능하다.

### 4.4.6 crossorigin 속성을 사용하는 CORS 요청

- HTML 요소도 crossorigin 속성을 부여하면 cors 모드로 요청을 전송할 수 있다.

## 4.7 프로세스 분리에 따른 사이드 채널 공격 대책

### 4.7.1 사이드 채널 공격을 방어하는 Site Isolation

- 컴퓨터의 CPU, 메모리 등 하드웨어에 대한 공격을 `사이드 채널 공격` 이라고 한다.
- 프로세스 분리는 사이트라는 단위로 이뤄지며 이 구조를 Site Isolation 이라고 한다.
- Site Isolation의 사이트는 출처와 다른 정의를 갖는 보안을 위한 경계이다.

### 4.7.2 출처마다 프로세스를 분리하는 구조

- CORP
  - CORP 헤더를 설정하면 헤더가 지정된 리소스를 가져올 때 동일 출처 또는 동일 사이트로 제한할 수 있다.
  - CORP를 활성화하려면 Cross-Origin-Resource-Policy 헤더를 리소스의 응답에 삽입한다.
- COEP
  - COEP 헤더를 페이지에 설정하면 페이지의 모든 리소스에 CORP 또는 CORS 헤더의 설정을 강제할 수 있다.
  - COEP를 활성화하려면 Corss-Origin-Embedder-Policy 헤더를 페이지의 응답에 삽입한다.
- COOP
  - COOP 헤더를 페이지에 설정하면 a 태그 요소와 `window.open` 함수로 오픈한 교차 출처 페이지의 접근을 제한할 수 있다.
  - COOP를 유효화하려면 Corss-Origin-Opener-Policy 헤더를 페이지의 응답에 추가한다.
