# CSRF, 클릭재킹, 오픈 리다이렉트

웹 애플리케이션이 점점 더 복잡해지고 사용자 데이터를 많이 다루게 되면서, 프론트엔드 개발자에게도 보안에 대한 이해는 필수가 되었다. 이 글에서는 프론트엔드 개발자가 알아야 할 세 가지 중요한 보안 위협 - CSRF, 클릭재킹, 오픈 리다이렉트에 대해 알아보고, 이를 방지하기 위한 실질적인 방법들을 소개한다.

## 1\. CSRF (Cross-Site Request Forgery)

### CSRF란 무엇인가?

CSRF(Cross-Site Request Forgery)는 사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위를 웹사이트에 요청하게 하는 공격이다. 이 공격은 웹사이트가 사용자의 브라우저에 저장된 쿠키와 같은 인증 정보를 신뢰한다는 점을 악용한다.

### 어떻게 작동하는가?

1.  사용자가 은행 웹사이트에 로그인하여 인증 쿠키를 받는다.
2.  로그아웃하지 않고 다른 웹사이트(악성 사이트)를 방문한다.
3.  악성 사이트는 은행 웹사이트로 돈을 이체하는 요청을 자동으로 전송하는 코드를 포함하고 있다.
4.  사용자의 브라우저는 은행 웹사이트에 요청을 보낼 때 자동으로 쿠키를 함께 전송한다.
5.  은행 웹사이트는 유효한 쿠키를 받았으므로 해당 요청이 사용자로부터 온 것으로 간주하고 처리한다.

### 실제 예시 코드

악성 웹사이트에 포함된 HTML:

```
<img src="https://bank.example.com/transfer?to=attacker&amount=1000" style="display:none" alt=""/>
```

또는 자동 제출되는 폼:

```
<body onload="document.forms[0].submit()">
  <form action="https://bank.example.com/transfer" method="POST">
    <input type="hidden" name="to" value="attacker"/>
    <input type="hidden" name="amount" value="1000"/>
  </form>
</body>
```

### 방어 방법

1.  **CSRF 토큰 사용하기**
2.  `<form action="/transfer" method="post"> <input type="hidden" name="csrf_token" value="랜덤_생성된_토큰_값"/> <!-- 다른 폼 요소들 --> </form>`
3.  **Same-Site 쿠키 속성 설정**
4.  `// 서버 측 코드 (예: Node.js) res.cookie('sessionId', 'value', { httpOnly: true, secure: true, sameSite: 'strict' // 또는 'lax' });`
5.  **사용자 인증 재확인**  
    중요한 작업(결제, 계정 설정 변경 등)에서는 암호 재입력이나 2FA를 요구한다.
6.  **Referer 검사**  
    요청의 출처를 확인하여 같은 도메인에서 온 것인지 검증한다.

## 2\. 클릭재킹 (Clickjacking)

### 클릭재킹이란 무엇인가?

클릭재킹은 사용자가 의도하지 않은 버튼이나 링크를 클릭하도록 속이는 기법이다. 공격자는 투명한 iframe을 사용하여 정상적인 웹사이트 위에 보이지 않는 레이어를 씌워 사용자의 클릭을 가로챈다.

### 어떻게 작동하는가?

1.  공격자는 매력적인 콘텐츠(예: "무료 상품 받기")가 있는 웹페이지를 만든다.
2.  그 페이지에 투명한 iframe으로 대상 웹사이트(예: 페이스북 "좋아요" 버튼)를 삽입한다.
3.  사용자가 "무료 상품 받기" 버튼을 클릭한다고 생각하지만, 실제로는 iframe 내의 "좋아요" 버튼을 클릭하게 된다.

### 실제 예시 코드

```
<style>
  .decoy-button {
    position: relative;
    z-index: 1;
    padding: 10px;
    background-color: green;
    color: white;
    cursor: pointer;
    opacity: 0.5; /* 유저가 뭔가 이상하다고 느끼지 않도록 반투명으로 설정 */
  }
  .hidden-iframe {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 2;
    opacity: 0.0001; /* 완전히 투명하지는 않게 설정 */
    pointer-events: auto; /* 클릭 이벤트 활성화 */
  }
</style>

<div style="position: relative; width: 300px; height: 200px;">
  <button class="decoy-button">무료 상품 받기</button>
  <iframe class="hidden-iframe" width="1000" height="500" src="https://target-site.com/sensitive-action-page" scrolling="no"></iframe>
</div>
```

### 방어 방법

1.  **X-Frame-Options 헤더 설정**
2.  `// 서버 측 코드 response.setHeader('X-Frame-Options', 'DENY'); // 또는 'SAMEORIGIN'`
3.  **Content-Security-Policy 사용**
4.  `// 서버 측 코드 response.setHeader('Content-Security-Policy', "frame-ancestors 'none'"); // 또는 'self'`
5.  **프레임 버스팅 기법**
6.  `<style>body { display: none; }</style> <script> if (self === top) { document.documentElement.style.display = 'block'; } else { top.location = self.location; } </script>`

## 3\. 오픈 리다이렉트 (Open Redirect)

### 오픈 리다이렉트란 무엇인가?

오픈 리다이렉트는 웹 애플리케이션이 사용자를 다른 URL로 리다이렉트할 때, 목적지 URL을 충분히 검증하지 않아 발생하는 취약점이다. 공격자는 이 취약점을 이용해 사용자를 피싱 사이트와 같은 악성 웹사이트로 리다이렉트할 수 있다.

### 어떻게 작동하는가?

1.  신뢰할 수 있는 웹사이트에 리다이렉트 기능이 있다(예: 로그인 후 특정 페이지로 돌아가기).
2.  URL 매개변수로 리다이렉트 대상을 지정할 수 있다.
3.  공격자는 신뢰할 수 있는 도메인을 사용하여 악성 URL을 만들고 이를 공유한다.
4.  사용자가 링크를 클릭하면 처음에는 신뢰할 수 있는 사이트로 이동하지만 즉시 악성 사이트로 리다이렉트된다.

### 실제 예시

취약한 URL:

```
https://trusted-site.com/redirect?url=https://malicious-site.com
```

취약한 코드:

```
// 서버 측 코드 (예: Express.js)
app.get('/redirect', (req, res) => {
  const redirectUrl = req.query.url;
  if (redirectUrl) {
    res.redirect(redirectUrl); // 취약점: URL 검증 없음
  } else {
    res.redirect('/');
  }
});
```

피싱 공격에 사용될 수 있는 URL:

```
https://trusted-site.com/redirect?url=https://evil-site.com/fake-login.html
```

### 방어 방법

1.  **화이트리스트 사용**
2.  `// 서버 측 코드 (예: Express.js) app.get('/redirect', (req, res) => { const redirectUrl = req.query.url; const allowedDomains = ['trusted-site.com', 'partner-site.com']; // URL 파싱 let parsedUrl; try { parsedUrl = new URL(redirectUrl); } catch (err) { return res.redirect('/'); } // 도메인 검증 if (allowedDomains.includes(parsedUrl.hostname)) { res.redirect(redirectUrl); } else { res.redirect('/'); } });`
3.  **상대 URL만 허용**
4.  `// 서버 측 코드 app.get('/redirect', (req, res) => { const redirectUrl = req.query.url; // 상대 URL인지 확인 if (redirectUrl && redirectUrl.startsWith('/')) { res.redirect(redirectUrl); } else { res.redirect('/'); } });`
5.  **사용자에게 리다이렉트 경고**
6.  `` // 클라이언트 측 코드 function safeRedirect(url) { if (isExternalUrl(url)) { if (confirm(`외부 사이트 ${url}로 이동합니다. 계속하시겠습니까?`)) { window.location.href = url; } } else { window.location.href = url; } } function isExternalUrl(url) { // 현재 도메인과 다른 도메인인지 확인 const currentDomain = window.location.hostname; try { const urlDomain = new URL(url).hostname; return urlDomain !== currentDomain; } catch (e) { // 상대 URL인 경우 return false; } } ``

## 결론

웹 보안은 프론트엔드 개발자에게도 필수적인 지식이다. CSRF, 클릭재킹, 오픈 리다이렉트와 같은 공격 기법을 이해하고 이에 대한 방어책을 알면 더 안전한 웹 애플리케이션을 구축할 수 있다.

보안은 한 번에 완료되는 작업이 아니라 지속적인 과정이라는 점을 명심하자. 새로운 보안 위협이 계속해서 등장하므로, 최신 보안 트렌드와 모범 사례를 항상 따라가는 것이 중요하다.

어떤 보안 조치도 완벽하지 않기 때문에, 여러 계층의 방어(Defense in Depth) 전략을 사용하여 보안을 강화하는 것이 좋다. 이는 단일 보안 계층이 실패하더라도 다른 계층이 공격을 방어할 수 있도록 해준다.

마지막으로, 보안은 팀 전체의 책임이라는 것을 기억하자. 백엔드 개발자, 프론트엔드 개발자, 데브옵스 엔지니어, QA 테스터 등 모든 사람이 보안에 대한 이해와 책임을 가져야 한다.
