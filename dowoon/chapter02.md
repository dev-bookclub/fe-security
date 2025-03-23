# 실습 준비

## 실습에 사용하는 소프트웨어
- 브라우저 (Chrome 권장)
- VSCode
- terminal
- Node.js
- npm
- Express

## 실습 서버 구축
### Express 설치
- 프록시와 방화벽이 설정된 네트워크의 경우 에러가 발생할 수 있음
```text
npm ERR! code UNABLE_TO_VERIFY_LEAF_SIGNATURE
```
- 에러 발생 시 아래 커맨드 실행 후 다시 Express 설치 (SSL/TLS 검증을 무효화)
```text
npm config set strict-ssl false
```

### HTTP 서버 구축
```js
const express = require('express');
const app = express();
const port = 3000;

app.get("/", (req, res, next) => {
  res.send("Top Page");
});

app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```

> **CommonJS vs ES Module**
> - CommonJS: Node.js에서 주로 사용되는 모듈 시스템으로 require()와 module.exports를 사용하여 모듈을 가져오고 내보냄
> - ES Module: ES6에서 도입된 모듈 시스템으로 import와 export를 사용하여 모듈을 가져오고 내보냄

