# Vue.js의 XSS 방어 메커니즘

Vue.js는 [공식 보안 가이드](https://vuejs.org/guide/best-practices/security.html#what-vue-does-to-protect-you)에서 다음과 같은 XSS 방어 메커니즘을 제공합니다.

## 1. 콘텐츠 자동 이스케이프

### 1.1 템플릿 내용 이스케이프

Vue는 모든 템플릿 내용을 자동으로 이스케이프 처리합니다.

```javascript
// 원본 코드
const userProvidedString = '<script>alert("hi")</script>'
<h1>{{ userProvidedString }}</h1>

// 빌드 후 결과
<h1>&lt;script&gt;alert(&quot;hi&quot;)&lt;/script&gt;</h1>
```

실제 코드를보면 아래와 같이 el.nodeValue & el.textContent 사용합니다. [상세 코드](https://github.com/vuejs/core/blob/main/packages/runtime-dom/src/nodeOps.ts#L77)

```typescript
export const nodeOps: Omit<RendererOptions<Node, Element>, 'patchProp'> = {
  // ... 다른 코드들 ...

  setText: (node, text) => {
    node.nodeValue = text;
  },

  setElementText: (el, text) => {
    el.textContent = text;
  },

  // ... 다른 코드들 ...
};
```

### 1.2 속성 바인딩 이스케이프

Vue는 동적 속성 바인딩도 자동으로 이스케이프 처리합니다.

```javascript
// 원본 코드
const userProvidedString = '" onclick="alert(\'hi\')'
<h1 :title="userProvidedString">
  hello
</h1>

// 빌드 후 결과
<h1 title="&quot; onclick=&quot;alert('hi')">
  hello
</h1>
```

실제 코드를보면 아래와 같이 el.setAttribute를 사용합니다. [상세 코드](https://github.com/vuejs/core/blob/main/packages/runtime-dom/src/modules/attrs.ts#L28)

```typescript
export function patchAttr(...){
...
      el.setAttribute(
        key,
        isBoolean ? '' : isSymbol(value) ? String(value) : value,
      )

...
}
```

## 2. 네이티브 브라우저 API 사용

Vue는 `textContent`와 `setAttribute` DOM API를 사용하여 콘텐츠를 안전하게 처리합니다.

### 2.1 지원 브라우저

- Chrome 1+
- Firefox 3.5+
- Safari 4+
- Internet Explorer 9+
- Opera 10+

### 2.2 안전한 API 사용 예시

```javascript
// 안전한 방법 (Vue가 사용)
element.textContent = userInput; // 텍스트 콘텐츠 처리
element.setAttribute('title', value); // 속성 처리

// 위험한 방법
element.innerHTML = userInput; // HTML로 해석될 수 있음
```

### 2.3 네이티브 API의 장점

1. 자동 이스케이프 처리
2. HTML 파싱 없음 (성능상 이점)
3. XSS 공격 방지
4. 브라우저 네이티브 구현으로 안정성 보장
