# React의 XSS 방어 메커니즘

React는 [공식 문서](https://legacy.reactjs.org/docs/dom-elements.html#security-notes)에서 설명하는 기본적인 XSS 방어 기능을 제공합니다.

## 1. JSX에서의 자동 이스케이프

### 1.1 텍스트 콘텐츠 이스케이프
React는 JSX 내의 모든 값을 렌더링하기 전에 이스케이프 처리합니다.

```javascript
// 원본 코드
const userProvidedString = '<script>alert("hi")</script>';
return <h1>{userProvidedString}</h1>;

// 렌더링 결과
<h1>&lt;script&gt;alert("hi")&lt;/script&gt;</h1>
```

### 1.2 속성 바인딩 이스케이프
React는 모든 속성 값도 자동으로 이스케이프 처리합니다.

```javascript
// 원본 코드
const userProvidedString = '" onclick="alert(\'hi\')';
return <h1 title={userProvidedString}>hello</h1>;

// 렌더링 결과
<h1 title="&quot; onclick=&quot;alert('hi')">hello</h1>
```

## 2. React의 DOM 처리

### 2.1 안전한 값 처리
```javascript
// 안전한 방법 (React의 기본 동작)
function SafeComponent({ userInput }) {
  return <div>{userInput}</div>;  // 자동으로 이스케이프
}

// 위험한 방법 (사용 주의)
function DangerousComponent({ userInput }) {
  return <div dangerouslySetInnerHTML={{ __html: userInput }} />;
}
```

### 2.2 React의 이벤트 처리
```javascript
// React의 안전한 이벤트 처리
function SafeButton({ onClick }) {
  // React는 이벤트 핸들러를 문자열이 아닌 함수로 처리
  return <button onClick={onClick}>Click me</button>;
}

// 안티패턴 (직접 HTML 문자열 사용)
function UnsafeButton({ onClickStr }) {
  // 절대 이렇게 하지 마세요!
  return <button onClick={eval(onClickStr)}>Click me</button>;
}
```

## 3. 주의가 필요한 기능

### 3.1 dangerouslySetInnerHTML
```javascript
// 위험! XSS 취약점 가능성 있음
function Component({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// 대신 DOMPurify 같은 라이브러리 사용 권장
import DOMPurify from 'dompurify';

function SafeComponent({ html }) {
  return <div dangerouslySetInnerHTML={{ 
    __html: DOMPurify.sanitize(html) 
  }} />;
}
```

### 3.2 href 속성 처리
```javascript
// React는 javascript: URLs을 자동으로 차단
function SafeLink({ url }) {
  return <a href={url}>Click me</a>;
}

// 추가 검증이 필요한 경우
function ValidatedLink({ url }) {
  const isValid = /^(https?:\/\/)/.test(url);
  return isValid ? <a href={url}>Click me</a> : null;
}
```

## 4. 추가 보안 조치

### 4.1 타입 검사로 보안 강화
```typescript
// TypeScript를 사용한 추가 보안
interface SafeProps {
  content: string;
  allowedTags?: string[];
}

function SafeComponent({ content, allowedTags }: SafeProps) {
  const sanitizedContent = sanitizeContent(content, allowedTags);
  return <div>{sanitizedContent}</div>;
}
```

### 4.2 런타임 보안 검사
```javascript
// 런타임에서 값 검증
function SafeRender({ content }) {
  const isSafe = React.useMemo(() => validateContent(content), [content]);
  
  if (!isSafe) {
    console.warn('Potentially unsafe content detected');
    return null;
  }

  return <div>{content}</div>;
}
```

## 5. 서버 사이드 렌더링(SSR) 보안

### 5.1 Next.js에서의 XSS 방지
```javascript
// pages/api/data.js
export default function handler(req, res) {
  // 데이터 검증
  const sanitizedData = sanitizeServerData(req.body);
  res.json({ data: sanitizedData });
}

// pages/index.js
export default function Page({ data }) {
  // 서버에서 받은 데이터도 자동 이스케이프
  return <div>{data}</div>;
}
```

### 5.2 hydration 보안
```javascript
// React 18의 안전한 hydration
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

React의 XSS 방어는 Vue와 마찬가지로 여러 계층에서 이루어지며, JSX를 통한 선언적 렌더링과 자동 이스케이프 처리가 핵심입니다. 특히 React는 `dangerouslySetInnerHTML`과 같은 위험한 기능을 사용할 때 명시적인 경고를 제공하여 개발자가 보안 위험을 인지하도록 합니다.