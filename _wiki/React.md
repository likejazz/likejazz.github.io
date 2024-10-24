---
layout: wiki 
title: React
tags: ["Software Engineering"]
last_modified_at: 2022/03/13 00:42:22
---

<!-- TOC -->

- [특징](#특징)
- [기타](#기타)
- [Code from Scratch](#code-from-scratch)
  - [Flask Backend](#flask-backend)
- [클래스 vs 함수](#클래스-vs-함수)
- [React Native 개발 정리](#react-native-개발-정리)
- [Books](#books)
  - [리액트를 다루는 기술 <sup>2019</sup>](#리액트를-다루는-기술-sup2019sup)

<!-- /TOC -->

# 특징
- 리액트의 핵심은 컴포넌트 단위로 조립
- HTML과 유사한 문법의 JSX 제공으로 직관적. 실제로는 JavaScript로 출력.
- Virtual DOM으로 빠른 업데이트. 개인적으로 검색 페이지 등에서 너무 빠른 업데이트는 사용성 저해. 물론 실시간으로 다양한 요소가 업데이트되는 어플리케이션에는 큰 도움이 될 듯.
- SEO는 SSR로 할 수 있다고 한다(확인 필요).
- 수정 사항 실시간 반영(SSE; Server-Sent Events?)
- 재밌게도 `npm start`를 하면 애플 스크립트를 실행해 브라우저까지 구동한다.
    - 수정 사항 실시간 반영
- 크롬 익스텐션 제공. Components 탭이 추가된다.

# 기타
- 2016년에 React Native 먼저 경험. 당시에 Nuclide IDE(Atom 기반)를 사용했으나 더 이상 업데이트 되지 않음.
- `node-sass` 패키지를 이용하는 예제가 있는데, 역시나 버전 이슈[^fn-node-sass] 때문에 설치가 가장 힘들다.

[^fn-node-sass]: <https://stackoverflow.com/questions/70281346/node-js-sass-version-7-0-0-is-incompatible-with-4-0-0-5-0-0-6-0-0>

# Code from Scratch
```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
</head>
<body>
<h1>Hello</h1>
<div id="root"></div>
<script crossorigin src="https://unpkg.com/react@17/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
<script>
    const Component = () => React.createElement('p', null, '안녕하세요');
    ReactDOM.render(
        React.createElement(Component, null, null),
        document.querySelector('#root')
    );
</script>
</body>
</html>
```
[^fn-1]

[^fn-1]: <https://velog.io/@kim-jaemin420/React-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%EC%82%AC%EC%9A%A9%EB%B0%A9%EB%B2%95>

JSX를 이용하는 방법. 바벨로 컴파일.
```jsx
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
</head>
<body>
<h1>Hello</h1>
<div id="root"></div>
<script crossorigin src="https://unpkg.com/react@17/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<script type="text/babel">
    const Component = () => (
        <div>
            <h1>목록</h1>
            <ul>
                <li>자바스크립트</li>
                <li>타입스크립트</li>
            </ul>
        </div>
    );
    ReactDOM.render(<Component/>, document.querySelector('#root'));
</script>
</body>
</html>
```

## Flask Backend
그런데 백엔드를 flask 등을 사용할 때 보면 저렇게 바벨로 하지 않고 노드 서버를 구동하고 호출은 백엔드로 따로 한다. 즉 2종류 서버를 모두 구동하는 것. flask를 nginx proxy[^fn-flask-react]로 설정해서 호출한다.

[^fn-flask-react]: <https://github.com/miguelgrinberg/react-flask-app>

# 클래스 vs 함수
클래스형 컴포넌트와 함수형 컴포넌트 모두 지원. 클래스형은,
- state 및 lifecycle 사용
- anonymous method 지원
- `render()`가 반드시 있어야 한다.

함수형도 hook을 통해(2019년 2월 릴리즈) useState 지원. 자바스크립트는 ES6 이전에는 클래스가 없다.

# React Native 개발 정리
*참고: 아래 내용은 2016년 여름 기준이며 현재는 개발 도구 및 버전이 변경 되었습니다.*

- AngularJS(2010)에 이어 React(2013) 등장. One-way data flow([Angular is known for it’s powerful two-way data-binding](https://toddmotto.com/one-way-data-binding-in-angular-1-5/)), Virtual DOM(고성능) 을 주요 기능으로 소개. 2015년 React Native 등장. 역사가 길지 않지만 폭발적인 인기를 끌고 있다. HTML을 완전히 사용할 수 있는건 아니지만 디자인 레이아웃은 HTML/CSS와 유사한 형태로 구현한다. 제한적인 컴포넌트를 사용하며 API를 이용하면 추가도 가능하다. JSX라는 js에 XML syntax를 추가한 preprocessor 단계를 사용한다. iOS/안드로이드 다른 구현이 존재한다.
- XCode로 프로젝트를 열어 Release 모드로 설정하면 모든 리소스를 로컬에서 네티이브 빌드가 가능하다. 해보진 않았으나 안드로이드도 비슷할듯. 동시에 두 플랫폼을 개발하고 풍부한 node 라이브러리를 사용할 수 있는 점이 장점이다. 그러나 리액트에 익숙하지 않다면 다양한 시행착오로 상당한 시간을 소비하게되는 점 또한 유사하다. 다행히 후술할 디버깅 툴의 도움으로 개발 생산성이 향상되었다.
- XCode로 빌드할때도 node가 필요하다. Build Phases를 보면 맨 마지막에 sh 파일이 있고 node로 실행되는 js 파일이다. 리액트가 디폴트 생성한 파일이며 따라서 필수적이다. 원격 서버에서 빌드할때 node가 없어서 실패한 적이 있다.
- 시행착오를 줄여주는 [강력한 디버깅 툴 제공](https://facebook.github.io/react-native/docs/debugging.html). 디버깅 툴의 기능을 알아갈수록 개발 시간이 점점 단축됐다. `^ + ⌘ + Z` 디버깅 메뉴. Hot Reloading에 매우 만족했다. 그러나 UI가 아닌 전체 코드의 변경사항을 보려면 전체 reload가 필요하다. 결국 Live Reload만 사용하게 됐다. Remote JS Debugging은 별도 크롬 브라우저에서 콘솔 창(개발자 모드)으로 디버깅이 가능한 최고의 기능이며 이 기능을 활용하고 부터 개발 생산성이 획기적으로 향상됐다.
- ~~Nuclide IDE 환경은 매우 좋다. 리액트를 위한 최적의 환경을 제공한다. 크롬으로 보는 것도 편리하지만 디버깅 윈도우를 우측에 띄울 수 있어 매우 편하다. React Native server도 Nuclide 내에서 바로 띄울 수있다. 덩달아 Atom에도 익숙해졌다. [단축키를 IntelliJ 기준으로 맞춘](https://atom.io/packages/intellij-idea-keymap) 이후 더욱 편해졌다. Atom을 주로 사용하다보니 상당히 트렌디한 느낌이다. 반면 Sublime Text는 점점 트렌드에서 밀리는 느낌.~~ DEPRECATED.
- 디버거는 하나의 도구만 붙을 수 있다. 따라서 크롬/Nuclide 동시 디버깅은 불가능하며 Nuclide에 붙이는게 편하다. 크롬에서 이미 보고 있다면 Nuclide에 붙지 않는데, 별도 오류 메시지가 없어서 한동안 원인 파악이 힘들었다.
- JSON 파싱이 디폴트로 편리하게 가능한 것과 달리 XML 파싱은 아예 제공조차 하지 않는다. 그러나 npm 모듈 중 [xmldom 라이브러리](https://stackoverflow.com/questions/29805704/react-native-fetch-xml-data)가 있어 [편리하게 이용](https://github.com/kaich/ASReact/blob/master/App/Main/TypePage.js)했다. node 기반인 React Native의 장점 중 하나다.
- 생각처럼 모든 HTML 컴포넌트들이 한 번에 찰싹 붙는건 아니었다. 마음에 드는 형태로 보이게 하려면 여러차례 수정을 해야 했고 특히 HTML 만으로는 어디를 수정해야 되는지 명확히 파악하기 힘들었다. 네이티브를 알고 있는 상태에서는 오히려 더 헷갈리는 부분이 많았다.

# Books
## 리액트를 다루는 기술 <sup>2019</sup>
★★★★☆  
설명도 친절하고 국내서라 읽기도 편하다. 차근차근 원리를 설명해주어 초보자가 리액트에 입문하기에 좋다. 그러나 이 책만 해도 분량이 엄청나고, 왜 리액트를 사용해야 하는지는 도입부에 충분히 설명한다. 중반 이후에는 다양한 도구와 실습 예제들이고 분량이 상당하다. 굳이 FE 실무를 하지 않는 이상 더 이상 시간 투자는 어렵다고 보여 10장까지만 읽고 학습 중단.