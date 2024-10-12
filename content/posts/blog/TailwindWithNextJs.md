---
title: "Tailwind의 동작 방식과 Next.js의 서버 컴포넌트"
date: 2024-10-11T18:55:03+09:00
draft: false
tags: ["Tailwind", "Emotion", "Next.js", "CSS-in-JS"]
categories: ["CSS", "Next.js"]
---

# **✓** Trigger

Next.js 13+ 프로젝트를 진행하면서 디자인 시스템을 구축할 때 어떤 스타일 라이브러리를 사용하면 좋을까 고민했다. 팀원들과 합의한 결과, Emotion을 사용하는 것으로 결정했다. 주된 이유는 Inline style처럼 사용하는 방식이 지저분하다는 것이었고 나도 동의했다.

그러나, 어떻게 사용하는지 알아보던 과정에서 Emotion이 서버 컴포넌트 지원이 어렵다는 글들을 보게 되었다. 또한 Next의 공식문서에서도 Tailwind를 추천하고 있길래 그 이유가 매우 궁금해졌다.

어떤 이유에서 Tailwind를 추천하고, Emotion은 서버 컴포넌트 지원이 어려울까?

우선 T**ailwind의 Inline style같은 방식에 대해 동작 방식과 장점**이 궁금했고, 최종적으로는 **Emotion과의 어떤 차이로 인해 서버 컴포넌트 지원이 가능한지** 궁금했다.

# **✓** Tailwind의 동작 방식

## Utility first CSS란?

Utility-First-CSS는 개별 CSS 속성(ex. padding, margin, color)에 해당하는 특수한 CSS 클래스를 사용해 스타일을 작성하는 것을 말한다. 이를 통해 CSS 파일을 별도로 작성하지 않고도 매우 빠르게 UI를 구성할 수 있다. 이러한 방식은 CSS 파일의 크기를 줄이고 HTML 페이지를 벗어나지 않고도 쉽게 스타일링이 가능하도록 한다.

```jsx
<h2 class="font-16 font-bold font-purple">Stranger Things</h2>
<p class="font-13 font-italic">Stranger Things is an American science fiction-horror web television...</p>
<h2 class="font-16 font-bold font-purple">Game of Thrones</h2>
<p class="font-13 font-italic">Game of Thrones is an American fantasy drama television...</p>
```

```jsx
/* Font sizes */

.font-13 { font-size: 13px }
.font-16 { font-size: 16px }
...

/* Font styles */

.font-bold { font-weight: bold }
.font-italic { font-style: italic }
...

/* Font colors */

.font-purple { color: purple }
...
```

### Inline style과 같은 개념인가?

어느정도 사실이다. 왜냐면 Tailwind class도 HTML 요소에 직접 적용되기 때문이다. 그러나 Tailwind는 단순 Inline style로 할 수 없는 반응형 UI를 만들 수 있고 가상 선택자 등을 사용할 수 있다. 무엇보다 특정 디자인 시스템을 기반으로 모든 것을 만들 수 있다.

### Tailwind도 일종의 Bootstrap인가?

흔히 Tailwind를 Bootstrap과 비교하는 경우가 많다. 하지만 추구하는 방향성은 다를 수 있다. Bootstrap과 같은 CSS Framework를 사용해도 결과는 Bootstrap의 컴포넌트들을 조합한, 비슷한 외관을 가지게 된다. 그러니까, Bootstrap은 자신들만의 디자인 시스템을 제공하는 것이다.

그러나 Tailwind는 CSS를 작성하는 방식만 제공할 뿐, 그들만의 스타일 자체를 제공하는 것은 아니다. Tailwind가 제공하는 utility들은 CSS에 1:1로 대응하는 저수준의 API이기 때문에 자신들만의 특정한 외관이 강제되지 않는다.

## CSS-in-JS: Runtime vs Zero-Runtime

> **_Runtime_**이란 코드가 실행되는 환경을 말한다. 즉, JavaScript 코드를 해석하고 실행하는 과정이 일어나는 시점이다. 이는 브라우저 환경에서 작동한다.

![rendering](/images/rendering.png)

1. 브라우저에서 사용자가 URL을 입력하고 접속하면 웹 서버에서 HTML 파일을 브라우저로 보낸다.
2. 브라우저가 HTML을 파싱하면서 DOM을 생성한다.
3. `<link>` 태그를 만나면 외부 리소스를 다운로드한다.
4. CSS 외부 리소스를 다운로드하면 CSS 파일을 이용해 CSSOM을 생성한다.
5. `<script>` 태그를 만나면 JavaScript 파일을 다운로드한다.
6. DOM과 CSSOM을 합쳐 Render Tree를 만든다.
7. Element 위치와 간격을 계산하는 Layout 과정이 일어난다.
8. Layer 별로 실제 그리는 작업인 Paint 과정이 일어난다.

이때 CSS와 JS는 렌더링 차단 리소스이다. CSS가 다운로드 되지 않으면 CSSOM을 만들 수 없고 이 때문에 Render Tree가 만들어지지 않아 그 다음 순서를 진행할 수 없기 때문이다. JavaScript는 DOM 파싱도 차단한다. 물론 `defer` 또는 `async`를 사용하지 않았을 때의 말이다.

### CSS-in-JS

CSS-in-JS는 JavaScript 코드 내에 CSS를 작성하는 방식이다. JavaScript 코드 내 템플릿 리터럴을 통해 CSS 코드가 존재하며 이는 브라우저로 JavaScript가 번들링 된 소스 코드 내 포함되어 전달되고, 런타임에 해석되는 방식이다.

### Runtime CSS

내가 사용하려고 했던 **Emotion**은 대표적인 runtime CSS이다. 이는 JavaScript가 모두 로드된 후 런타임 환경에서 스타일을 생성하기 때문에 번들링 된 JavaScript 파일이 브라우저에서 실행 된 후 스타일이 생성된다.

### Zero-Runtime CSS

반면 Tailwind의 경우 zero-runtime CSS로 볼 수 있으며, 이는 서버에서 빌드 시점에 CSS를 생성하고 이를 브라우저에 전달하게 된다.

# **✓** Next.js와 Tailwind

React 18 version부터는 서버에서 `async`를 이용해 서버 컴포넌트를 만들 수 있다. 이를 React Server Component(RSC)라 부른다. 이를 사용할 때 주의할 점이 있는데, 서버 컴포넌트는 서버에서만 렌더링 되며 클라이언트에서 렌더링 되지 않는다. 따라서 클라이언트 쪽에서 돌아가는 `useState`나 `useEffect` 등을 사용할 수 없다.

그렇다면 어떻게 해야할까? **런타임 이전에 스타일이 결정**되면 간단히 해결될 문제다. 이러한 이유에서 Next.js가 Tailwind를 추천한다고 생각한다. 물론 다른 이유들도 존재하겠지만 내가 궁금해하던 문제에 초점을 두자면 그렇다.

# **✓ 결론**

현재 해당 프로젝트를 마친 시점이다. Inline style처럼 줄줄이 써야하는 것에 대한 거부감이 있었는데, 직접 config 파일로 디자인 토큰들을 지정해두고 내가 만든 시스템 내에서 빠르게 작업할 수 있었다.

또한 SSR 과정과 함께 CSS의 동작과 효율성을 함께 생각할 수 있어서 좋은 기회였다. 이전까지는 솔직하게 별 생각 없이 사용하기 편리해 보이거나 새로운 것을 써보고 싶은 마음으로 스타일링 도구를 선택하였는데, 이에 대해 돌이켜볼 수 있었다. 이때까지 사용해본 SCSS, Vanilla Extract 등 여러 도구들을 비교해보는 것도 재밌을 것 같다.

### references

https://tailwindcss.com/docs/content-configuration#class-detection-in-depth

https://shiwoo.dev/posts/next-13-and-css-in-js#css-in-js와-서버-사이드-렌더링

https://pozafly.github.io/css/explore-how-to-apply-modern-css/

https://frontstuff.io/in-defense-of-utility-first-css
