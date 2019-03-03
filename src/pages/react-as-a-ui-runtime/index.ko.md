---
title: UI 런타임으로서의 React
date: '2019-02-02'
spoiler: React 프로그래밍 모델의 상세한 설명.
---

튜토리얼 대부분은 React를 UI 라이브러리로 소개합니다. 맞는 말이죠. 실제로 React는 UI 라이브러리니까요. 태그라인에도 그렇게 쓰여있고요!

![React homepage screenshot: "A JavaScript library for building user interfaces"](./react.png)

얼마 전 [유저 인터페이스](/ko/the-elements-of-ui-engineering/)를 구축할 때 고려할 점들에 대한 글을 썼습니다. 하지만 이번 글은 UI 라이브러리가 아닌, [프로그래밍 런타임](https://en.wikipedia.org/wiki/Runtime_system)으로서의 React에 대한 것입니다.

**이 글은 유저 인터페이스 구축에 대해서는 다루지 않습니다.** 그러나 React 프로그래밍 모델을 좀 더 깊이 이해하는 데에는 도움이 될 거예요.

---

**노트: React를 _배우기 시작한_ 분은 이 글보다는 [React 문서](https://reactjs.org/docs/getting-started.html#learn-react)를 읽으시길 바랍니다.**

<font size="60">⚠️</font>

**이 글은 React를 아주 깊이 다루는 글입니다 — 초심자를 위한 글이 아닙니다.** 이 글은 React 프로그래밍 모델의 원칙 거의 대부분을 다룹니다. 어떻게 사용하는지가 아니라, 어떻게 동작하는지만 설명합니다.

저는 숙련된 개발자들, 그리고 React 외의 다른 UI 라이브러리의 사용자 중 React에 존재하는 트레이드오프에 대해 궁금한 사람들을 대상으로 이 글을 썼습니다. 그들에게 이 글이 도움이 되길 바랍니다.

**대다수 사람들은 이 글에서 다룰 주제에 대한 고려 없이도 React를 다년간 성공적으로 사용해왔습니다.** 이 글은 [디자이너 관점](http://mrmrs.cc/writing/2016/04/21/developing-ui/)이 아니라, 완전히 프로그래머 관점으로 바라본 React에 대한 글입니다. 두 관점을 다 이해한다고 나쁠 건 없겠죠.

서론은 여기까지 하고, 시작해봅시다.

---

## 호스트 트리

어떤 프로그램은 숫자를 만들어냅니다. 어떤 프로그램은 시를 씁니다. 프로그래밍 언어와 그 언어의 런타임은 대개 특정 유즈케이스 집합에 최적화되어 있고, React도 예외는 아닙니다.

React 프로그램은 보통 **시간이 지남에 따라 변할 수 있는 트리**를 만들어냅니다. 그 결과물은 [DOM 트리](https://www.npmjs.com/package/react-dom)일수도 있고, [iOS 뷰 계층 구조](https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/View%20Hierarchy.html)나 [PDF 트리](https://react-pdf.org/), 아니면 심지어 [JSON 오브젝트](https://reactjs.org/docs/test-renderer.html)일수도 있습니다. 어쨌든, 어떠한 UI가 표현되길 원하는 경우가 대부분이죠. 이 UI는 React 바깥에 있는 *호스트 환경*의 일부니까, "*호스트* 트리"라고 부르겠습니다. 호스트 트리는 대개 [자기들만의](https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild) [필수 API](https://developer.apple.com/documentation/uikit/uiview/1622616-addsubview)를 가지고 있습니다. React는 그 API를 덮고 있는 레이어고요.

그러면 React는 어떨 때 유용할까요? 아주 추상적으로 말하자면, React는 유저 상호작용, 네트워크 응답, 타이머 등의 외부 이벤트가 발생했을 때, 복잡한 호스트 트리를 안정적으로 조작하는 프로그램을 작성할 수 있도록 돕습니다.

전문화된 도구는 어떤 가정 아래에서, 그리고 그 가정을 최대한 활용할 때 일반적인 도구보다 더 잘 동작합니다. React는 다음 두 가정을 주요 원칙으로 삼습니다:

- **안정성.** 호스트 트리는 상대적으로 안정적이며, 업데이트 대부분은 트리 전체 구조를 급격하게 바꾸지 않는다. 앱이 모든 요소를 매초마다 뒤죽박죽으로 보여준다면 사람이 사용하기 어려울 것이다. 이 버튼은 어디갔지? 왜 내 화면이 춤추고 있는거야?
- **규칙성.** 호스트 트리는 무작위한 형태가 아닌, 모양과 행동이 일관적인 UI 패턴(버튼, 목록, 아바타처럼)으로 나눠질 수 있다. 

**이제 이 원칙들은 거의 모든 UI에서 참인 명제가 됐습니다.** 특히 React는 UI 결과물에 안정적인 "패턴"이 없는 경우 사용하기에 적합하지 않습니다. 예를 들면, React로 트위터 클라이언트를 만들 수는 있어도 [3D 파이프 화면 보호기](https://www.youtube.com/watch?v=Uzx9ArZ7MUU)를 만들기는 어려울 거예요.

## 호스트 인스턴스

호스트 트리는 여러 노드로 이뤄져있습니다. 이 노드를 "호스트 인스턴스"라고 부르겠습니다.

DOM 환경의 호스트 인스턴스는 일반적인 DOM 노드입니다. `document.createElement('div')`로 만들어진 오브젝트가 그 예시죠. iOS 환경에서는 JavaScript로부터 생성된 네이티브 뷰를 호스트 인스턴스로 볼 수 있겠네요.

호스트 인스턴스는 각자 자신만의 속성(`domNode.className`이나 `view.tintcolor` 등)을 지닙니다. 다른 호스트 인스턴스를 자식으로 데리고 있을 수도 있고요.

(저는 지금 React와 상관없는, 호스트 환경에 대해 설명하고 있습니다.)

그리고 이런 속성을 제어하는 API도 함께 존재합니다. 예를 들어 DOM은 `appendChild`, `removeChild`, `setAttribute` 같은 API를 제공하는 식이죠. React 앱 안에서는 개발자가 이런 API를 직접 호출할 일이 드뭅니다. 그건 React가 할 일이니까요.

## 렌더러

*렌더러*는 React가 특정 호스트의 환경과 인스턴스를 다룰 수 있게 해줍니다. React DOM, React Native, 그리고 [Ink](https://mobile.twitter.com/vadimdemedes/status/1089344289102942211)조차도 React 렌더러입니다. 물론 [당신만의 React 렌더러](https://github.com/facebook/react/tree/master/packages/react-reconciler)를 만들 수도 있죠. *(역자 주: Ink는 CLI용 인터랙티브 앱을 React로 만들 수 있게 해주는 [프로젝트](https://github.com/vadimdemedes/ink/tree/next)입니다.)* 

React 렌더러는 두가지 모드로 동작할 수 있습니다.

대다수의 렌더러는 "변이mutating" 모드를 사용합니다. DOM도 이 방식으로 동작합니다: 노드 생성, 속성 설정, 자식 노드 삽입 또는 제거. 호스트 인스턴스는 완전히 가변적입니다mutable.

React에는 "지속persistent" 모드도 있습니다. 이 모드는 `appendChild()`같은 메서드를 제공하지 않는 호스트 환경을 위한 것입니다. 이런 환경에서는 자식을 동적으로 삽입하는 대신, 부모 트리를 복제해서 전체를 교체하는 식으로 동작합니다. 호스트 트리의 불변성immutability은 다중쓰레딩을 적용하기 쉽게 해주죠. [React Fabric](https://facebook.github.io/react-native/blog/2018/06/14/state-of-react-native-2018)이 다중쓰레딩을 잘 활용하는 예입니다.

사실 React 개발자는 이런 모드에 대해 생각할 필요가 전혀 없습니다. 저는 단지 React가 한 모드에서 다른 모드로 바꿔주는 어댑터 그 이상이라는 걸 강조하고 싶었어요. React는 호스트의 저수준 API 패러다임이 무엇이든 상관없이 유용합니다.

## React 엘리먼트

호스트 인스턴스는 호스트 환경을 이루는 가장 작은 빌딩 블록입니다. DOM 환경의 DOM 노드처럼요. React에서는 *React 엘리먼트*가 가장 작은 빌딩 블록입니다.

React 엘리먼트는 순수한 JavaScript 오브젝트이며, 호스트 인스턴스를 *묘사*하는 역할을 합니다.

```jsx
// 다음 JSX는 이 오브젝트를 짧게 쓴 것입니다.
// <button className="blue" />
{
  type: 'button',
  props: { className: 'blue' }
}
```

React 엘리먼트는 가볍고, 호스트 인스턴스에 종속되지 않습니다. 다시 말하지만, React 엘리먼트는 단지 개발자가 화면에서 보고 싶은 무언가를 *묘사*할 뿐입니다.

호스트 인스턴스처럼 React 엘리먼트도 트리를 구성할 수 있습니다:

```jsx
// 다음 JSX는 이 오브젝트를 짧게 쓴 것입니다.
// <dialog>
//   <button className="blue" />
//   <button className="red" />
// </dialog>
{
  type: 'dialog',
  props: {
    children: [{
      type: 'button',
      props: { className: 'blue' }
    }, {
      type: 'button',
      props: { className: 'red' }
    }]
  }
}
```

*(노트: 글 진행에 중요하지 않은 [몇몇 속성](/why-do-react-elements-have-typeof-property/)을 의도적으로 생략했습니다.)*

그러나, 호스트 인스턴스와 달리 **React 엘리먼트는 자신만의 영속적인 아이덴티티persistent identity를 지니고 있지 않습니다.** React 엘리먼트는 매번 버려지고 다시 만들어지도록 디자인되었습니다.

React 엘리먼트는 불변적입니다. 예를 들어, React 엘리먼트의 자식이나 속성은 변경할 수 없습니다. 뭔가 다른 화면을 렌더링하고 싶다면, 그 화면을 *묘사*하는 새로운 React 엘리먼트 트리를 처음부터 만들어야 합니다.

저는 React 엘리먼트를 영화 프레임으로 생각하면 편하더군요. UI가 특정 순간에 어떻게 보여야 하는지를 캡처해준 것과 같습니다. 캡처는 변하지 않죠.

## 진입점

각 React 렌더러는 "진입점Entry Point"을 지닙니다. 개발자는 진입점 API를 통해 컨테이너 호스트 인스턴스에 특정 React 엘리먼트 트리를 렌더링할 수 있습니다.

예를 들어 React DOM 렌더러의 진입점은 `ReactDOM.render`입니다:

```jsx
ReactDOM.render(
  // { type: 'button', props: { className: 'blue' } }
  <button className="blue" />,
  document.getElementById('container')
);
```

 `ReactDOM.render(reactElement, domContainer)`는 이런 뜻입니다: **“React야, `domContainer` 호스트 트리가 내 `reactElement`에 매칭되게 해주렴.”**

React는 `reactElement.type` (위 예시에서는 `'button'`)을 살펴보고, React DOM 렌더러가 호스트 인스턴스를 만들어서 다음과 같이 인스턴스 속성을 지정합니다:

```jsx{3,4}
// ReactDOM 렌더러 어딘가의 소스 코드(생략된 버전)
function createHostInstance(reactElement) {
  let domNode = document.createElement(reactElement.type);
  domNode.className = reactElement.props.className;
  return domNode;
}
```

즉 우리의 예시에서 React가 하는 일은 이렇게 됩니다:

```jsx{1,2}
let domNode = document.createElement('button');
domNode.className = 'blue';

domContainer.appendChild(domNode);
```

React 엘리먼트에 자식이 있으면(`reactElement.props.children`), React는 첫번째 렌더링에서 자식들에 대한 호스트 인스턴스도 재귀적으로 만들어냅니다.

## 조정

같은 컨테이너에 `ReactDOM.render()` 를 두 번 호출하면 무슨 일이 벌어질까요?

```jsx{2,11}
ReactDOM.render(
  <button className="blue" />,
  document.getElementById('container')
);

// ... later ...

// 버튼은 *교체*될까요?
// 아니면 기존 버튼의 속성을 업데이트할까요?
ReactDOM.render(
  <button className="red" />,
  document.getElementById('container')
);
```

다시 강조하자면, React의 역할은 *주어진 React 엘리먼트 트리를 호스트 트리에 매칭*시키는 것입니다. 새로 정보가 주어졌을 때,  호스트 인스턴스 트리에 *어떤* 작업을 해야 하는지 알아내서 수행하는 과정을 [조정reconciliation](https://reactjs.org/docs/reconciliation.html)이라고 부릅니다.

조정을 하는 방법은 두 가지입니다. 간단한 방법은 기존의 트리를 다 날려버리고 처음부터 다시 만드는 거죠.

```jsx
let domContainer = document.getElementById('container');
// 트리 삭제
domContainer.innerHTML = '';
// 새 호스트 인스턴스 생성
let domNode = document.createElement('button');
domNode.className = 'red';
domContainer.appendChild(domNode);
```

그러나 DOM에서는 이 방법은 느릴뿐 아니라 포커스, 셀렉트 인풋 선택 상태, 스크롤 상태 등 중요한 정보를 다 잃어버리게 만듭니다. 대신 이렇게 해야 합니다:

```jsx
let domNode = domContainer.firstChild;
// 기존의 호스트 인스턴스를 업데이트
domNode.className = 'red';
```

즉 React는 새 React 엘리먼트를 매칭하기 위해, 언제 기존 호스트 인스턴스를 *업데이트*하고 언제 *새것*을 생성할지 결정해야 합니다.

이로 인해 *동일성* 문제가 제기됩니다. React 엘리먼트는 매번 달라질 수 있는데, 어떨 때 이 엘리먼트들이 같은 호스트 인스턴스를 의미한다고 간주할 수 있을까요?

우리의 예시에서는 간단합니다. `<button>` 은 `document`의 첫 번째이자 유일한 자식이었고, 우리는 `<button>`을 그 위치에 렌더링하고자 합니다. `<button>` 호스트 인스턴스는 이미 있으니 다시 생성할 필요는 없겠죠? 그냥 재사용합시다.

사실 이 설명은 React의 실제 동작 방식과 상당히 유사합니다.

**트리의 동일 위치에 존재하는 엘리먼트의 타입이 두 렌더링 사이에 "매칭된다면", React는 기존 호스트 인스턴스를 재사용합니다.**

다음은 React가 무슨 일을 하는지를 대략적으로 보여주는 예시입니다:

```jsx{9,10,16,26,27}
// let domNode = document.createElement('button');
// domNode.className = 'blue';
// domContainer.appendChild(domNode);
ReactDOM.render(
  <button className="blue" />,
  document.getElementById('container')
);

// 호스트 인스턴스 재사용 가능? 그렇다! (button → button)
// domNode.className = 'red';
ReactDOM.render(
  <button className="red" />,
  document.getElementById('container')
);

// 호스트 인스턴스 재사용 가능? 아니다! (button → p)
// domContainer.removeChild(domNode);
// domNode = document.createElement('p');
// domNode.textContent = 'Hello';
// domContainer.appendChild(domNode);
ReactDOM.render(
  <p>Hello</p>,
  document.getElementById('container')
);

// 호스트 인스턴스 재사용 가능? 그렇다! (p → p)
// domNode.textContent = 'Goodbye';
ReactDOM.render(
  <p>Goodbye</p>,
  document.getElementById('container')
);
```

자식 트리에 대해서도 같은 휴리스틱이 쓰입니다. `<button>` 두 개를 자식으로 가진 `<dialog>`를 업데이트한다고 해보죠. React는 우선 `<dialog>`를 재사용할지 결정한 뒤, 같은 의사결정 과정을 각 자식마다 반복합니다.

## 조건

React가 두 업데이트 사이의 엘리먼트 타입이 매칭될 때만 호스트 인스턴스를 재사용한다면, 조건부로 렌더링되는 엘리먼트가 있으면 어떻게 될까요?

처음에는 인풋만 보여주고, 그다음 메시지를 함께 보여주는 예제를 보죠:

```jsx{12}
// 첫번째 렌더링
ReactDOM.render(
  <dialog>
    <input />
  </dialog>,
  domContainer
);

// 다음 렌더링
ReactDOM.render(
  <dialog>
    <p>나 방금 생겼음!</p>
    <input />
  </dialog>,
  domContainer
);
```

이 예제에서 `<input>` 호스트 인스턴스는 재사용되지 않고 새로 만들어집니다. React가 엘리먼트 트리를 타고 들어가 두 버전을 비교할 때 이렇게 하거든요:

* `dialog → dialog`: 호스트 인스턴스 재사용 가능? **그렇다 — 타입이 매칭된다.**
  * `input → p`: 호스트 인스턴스 재사용 가능? **타입이 바뀌었다! 재사용할 수 없다.** 기존 `input`을 삭제하고 새로 `p` 호스트 인스턴스를 만든다.
  * `(nothing) → input`: 새 `input` 호스트 인스턴스를 만들어야 한다.

즉 React는 실질적으로 다음과 같이 작동하게 됩니다:

```jsx{1,2,8,9}
let oldInputNode = dialogNode.firstChild;
dialogNode.removeChild(oldInputNode);

let pNode = document.createElement('p');
pNode.textContent = '나 방금 생겼음!';
dialogNode.appendChild(pNode);

let newInputNode = document.createElement('input');
dialogNode.appendChild(newInputNode);
```

이는 당연히 좋지 않은데, *개념상*  `<input>` 이 `<p>`로 *교체*된 게 아니기 때문입니다 — `<input>`은 단지 움직였을 뿐입니다. 우리는 DOM 재생성으로 인해 포커스 상태나 인풋 내용을 잃는 것은 원치 않습니다.

이 문제를 해결하는 것이 어렵진 않지만(곧 보여드릴 텐데), 사실 React 앱에서 이런 일이 자주 발생하지는 않습니다. 왜 그런지 살펴볼까요?

실무에서 개발자가 `ReactDOM.render` 를 직접 호출할 일은 아주 드뭅니다. 대신 React 앱은 다음과 같이 여러 함수로 쪼개져있기 마련이죠:

```jsx
function Form({ showMessage }) {
  let message = null;
  if (showMessage) {
    message = <p>나 방금 생겼음!</p>;
  }
  return (
    <dialog>
      {message}
      <input />
    </dialog>
  );
}
```

이 예제에는 우리가 겪었던 문제가 없습니다. JSX 대신 오브젝트 표현을 쓰면 좀 더 이해가 쉬워질 텐데요. `dialog`의 자식 엘리먼트 트리를 봅시다:

```jsx{12-15}
function Form({ showMessage }) {
  let message = null;
  if (showMessage) {
    message = {
      type: 'p',
      props: { children: '나 방금 생겼음!' }
    };
  }
  return {
    type: 'dialog',
    props: {
      children: [
        message,
        { type: 'input', props: {} }
      ]
    }
  };
}
```

**`showMessage`가 `true`든 `false`든 간에, `<input>`은 두 번째에 위치한 자식이며, 따라서 렌더링 사이에 트리에서의 위치가 변하지 않습니다.**

`showMessage`가 `false`에서 `true`로 변하면 React는 엘리먼트 트리를 타고 들어가 이전 버전과 비교합니다:

* `dialog → dialog`: 호스트 인스턴스 재사용 가능? **그렇다 — 타입이 매칭된다.**
  * `(null) → p`: 새로운 `p` 호스트 인스턴스를 삽입해야 한다.
  * `input → input`: 호스트 인스턴스 재사용 가능? **그렇다 — 타입이 매칭된다.**

그리고 React는 이런 식으로 작동하게 됩니다:

```jsx
let inputNode = dialogNode.firstChild;
let pNode = document.createElement('p');
pNode.textContent = '나 방금 생겼음!';
dialogNode.insertBefore(pNode, inputNode);
```

인풋 상태는 모두 보존됩니다.

## 리스트

호스트 인스턴스를 재사용할지 재생성할지 결정할 때, 대개 트리에서 같은 위치에 있는 엘리먼트 타입을 비교하는 것만으로도 충분합니다.

이는 자식들의 위치가 정적이며 변경되지 않을 때만 제대로 작동합니다. 위 예시에서, 우리는 `message`가 "구멍"이 될 수 있더라도 여전히 인풋이 메시지 뒤에 위치하며 다른 자식이 없다는 걸 알고 있습니다.

그러나 동적인 리스트에서는 위치의 순서를 보장할 수 없습니다:

```jsx
function ShoppingList({ list }) {
  return (
    <form>
      {list.map(item => (
        <p>
          구매 대상: {item.name}
          <br />
          구매 수량: <input />
        </p>
      ))}
    </form>
  )
}
```

쇼핑 물품 `list`의 정렬 순서가 변경되더라도, React는 모든 `p`와 `input` 엘리먼트가 같은 타입을 가졌기 때문에 움직였다는 사실을 알 수 없습니다. (React의 관점에서는 *item 자체*는 변했으나 순서는 변하지 않은 셈이죠.)

React에서 실제 실행되는 코드는 대략 이렇습니다:

```jsx
for (let i = 0; i < 10; i++) {
  let pNode = formNode.childNodes[i];
  let textNode = pNode.firstChild;
  textNode.textContent = '구매 대상 ' + items[i].name;
}
```

즉 React는 각 쇼핑 물품을 *재배열*하는 대신 *업데이트*합니다. 이는 성능 문제는 물론 버그도 야기할 가능성이 있는데, 예를 들어 첫번째 인풋에 입력해둔 내용이 순서 변경 *이후*에도 그자리에 남아있을 수 있습니다.

**이게 바로 개발자가 엘리먼트의 배열을 렌더링할 때, `key` 속성을 명시하라고 React가 항상 징징대는 이유입니다.**

```jsx{5}
function ShoppingList({ list }) {
  return (
    <form>
      {list.map(item => (
        <p key={item.productId}>
          구매 대상: {item.name}
          <br />
          구매 수량: <input />
        </p>
      ))}
    </form>
  )
}
```

어떤 엘리먼트의 위치가 렌더링 사이에 달라졌더라도, `key`가 같다면 React는 *개념적으로* 같은 엘리먼트로 판단합니다.

React가 `<form>` 안에서 `<p key="42">`를 발견하면, React는 이전 렌더링에서 같은 `<form>` 안에 `<p key="42">`가 있었는지 확인합니다. `<form>`의 자식들 사이에 순서가 바뀌었더라도요. 같은 키의 호스트 인스턴스가 존재하면 React는 그 인스턴스를 재사용하고, 그에 따라 자식들을 재배열합니다.

`key`는 위 예시에서의 `<form>`처럼, 동일 부모를 가진 React 엘리먼트 사이에서만 판단의 기준이 된다는 것을 기억하세요. 부모가 다르면 React는 같은 키를 가진 엘리먼트끼리의 "매칭"을 시도하지 않습니다. (React에서는 호스트 인스턴스를 다시 만드는 것이 인스턴스를 다른 부모로 이동시키는 유일한 방법입니다.)

`key`로는 어떤 값이 적절할까요? 간단한 답은: **순서가 바뀌더라도 아이템이 "같음"을 보장하려면 어떤 정보를 살펴봐야 하는가?** 입니다. 예를 들어 쇼핑 목록에서는 상품의 ID가 형제 엘리먼트들 사이에서의 유일성을 보장하는 구분자가 되겠죠.

## 컴포넌트

함수를 리턴하는 React 엘리먼트는, 몇번 보셨다시피 이렇게 생겼습니다:

```jsx
function Form({ showMessage }) {
  let message = null;
  if (showMessage) {
    message = <p>나 방금 생겼음!</p>;
  }
  return (
    <dialog>
      {message}
      <input />
    </dialog>
  );
}
```

이를 *컴포넌트*라고 부릅니다. 컴포넌트는 개발자만의 버튼, 아바타, 코멘트 등을 만들 수 있게 해주는 연장통입니다. 컴포넌트야말로 React의 핵심입니다.

컴포넌트는 오브젝트 해시 하나만을 인자로 받습니다. 이 오브젝트는 "props"를 지니고 있고요. ("properties"의 약자입니다.) 위 예시에서는 `showMessage`가 prop 중 하나입니다. 명명된 인자named arguments와 유사하다고 볼 수 있죠.

## 순수성

React 컴포넌트는 props에 대한 순수 함수로 간주됩니다.

```jsx
function Button(props) {
  // 🔴 이렇게는 쓸 수 없습니다.
  props.isActive = true;
}
```

React에서는 일반적으로 변이가 잘 사용되지 않습니다. (이벤트에 반응하여 UI를 업데이트하는 방법에 대해서는 조금 있다가 설명하도록 하죠.)

하지만 *지역적 변이(local mutation)*는 전혀 문제 없습니다:

```jsx{2,5}
function FriendList({ friends }) {
  let items = [];
  for (let i = 0; i < friends.length; i++) {
    let friend = friends[i];
    items.push(
      <Friend key={friend.id} friend={friend} />
    );
  }
  return <section>{items}</section>;
}
```

`items`는 *렌더링 도중*에 만들어졌고, 다른 컴포넌트는 `items`를 "볼" 수 없으므로, 렌더링 결과로 UI에 전달하기 전에 얼마든지 변경할 수 있습니다. 지역적 변이를 피하기 위해 코드를 뒤틀 필요는 없습니다.

비슷하게, 완전히 "순수"하지는 않지만 지연된 초기화도 괜찮습니다:

```jsx
function ExpenseForm() {
  // 다른 컴포넌트에 영향을 미치지만 않는다면 괜찮습니다:
  SuperCalculator.initializeIfNotReady();

  // 렌더링을 계속합니다...
}
```

컴포넌트를 여러 번 불러도 안전하고, 다른 컴포넌트의 렌더링에 영향을 미치지 않는다면, React는 엄격한 함수형 프로그래밍의 관점에서 컴포넌트가 100% 순수한지 아닌지는 신경쓰지 않습니다. React에서는 [멱등성](https://stackoverflow.com/questions/1077412/what-is-an-idempotent-operation)이 순수성보다 더 중요하죠. *(역자 주: 함수의 멱등성은 같은 인자로 함수를 여러 번 호출하더라도, 첫 번째 호출한 것과 같은 결과를 반환하는 성질입니다.)* 

즉 사용자에게 직접 노출되는 사이드 이펙트는 React 컴포넌트 안에서 발생하면 안 됩니다. 다른 말로 하면, 단순히 컴포넌트 함수를 *호출*하는 것만으로 스크린에 어떤 변화가 생겨서는 안 됩니다.

## 재귀

컴포넌트는 함수이므로, 한 컴포넌트가 다른 컴포넌트를 *사용*하려면 함수 부르듯 *호출*하면 되겠죠?: 

```jsx
let reactElement = Form({ showMessage: true });
ReactDOM.render(reactElement, domContainer);
```

하지만 실제로는 React 런타임에서 이런 방식으로 컴포넌트가 사용되지는 않습니다.

React가 컴포넌트를 사용하는 메커니즘은 React 엘리먼트를 사용하는 메커니즘과 같습니다. **즉, 개발자가 컴포넌트 함수를 직접 호출하는 대신, React가 알아서 하게 하는 방식이죠:**

```jsx
// { type: Form, props: { showMessage: true } }
let reactElement = <Form showMessage={true} />;
ReactDOM.render(reactElement, domContainer);
```

React 안 어딘가에서는 이런 식으로 컴포넌트가 호출됩니다:

```jsx
// React 안 어딘가
let type = reactElement.type; // Form
let props = reactElement.props; // { showMessage: true }
let result = type(props); // Form의 반환값
```

컴포넌트 함수의 이름은 첫 자를 대문자로 쓰는 것이 관행입니다. JSX가 html 문법을 변환할 때 `<form>`이 아닌 `<Form>` 을 만나면, 오브젝트의 `type`을 단순한 문자열이 아닌 구분자로 만듭니다.:

```jsx
console.log(<form />.type); // 'form' 문자열
console.log(<Form />.type); // Form 함수
```

거창한 전역 등록 메커니즘 같은 건 없습니다. React는 문자 그대로 `<Form />`을 `Form`이란 이름을 이용해 참조합니다. `Form`이 지역 스코프에 존재하지 않는다면, 개발자는 변수 이름을 잘못썼을 때처럼 자바스크립트 에러를 보게 됩니다.

**그러면 엘리먼트의 타입이 함수일 때 React가 하는 일은 무엇일까요? React는 컴포넌트를 호출해서, _그 컴포넌트가_ 무엇을 렌더링하길 원하는지 묻습니다.**

이 프로세스는 재귀적으로 반복됩니다. 자세한 설명은 이 [링크](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)를 참조하세요. 간단하게는 이런 식으로 진행됩니다:

- **개발자:** `ReactDOM.render(<App />, domContainer)`
- **React:** 이봐 `App`, 렌더링하고 싶은 게 뭐야?
  - `App`: 나는 `<Layout>` 안에 `<Content>`를 렌더링하고 싶어.
- **React:** 이봐 `Layout`, 렌더링하고 싶은 게 뭐야?
  - `Layout`: 난 `<div>` 안에 내 자식들을 렌더링하고 싶어. 내 자식이 `<Content>`였으니, 그게 `<div>` 안에 들어갈 거야.
- **React:** 이봐 `<Content>`, 렌더링하고 싶은 게 뭐야?
  - `Content`: 난 `<article>`  안에 텍스트 조금이랑 `<Footer>`를 렌더링할래.
- **React:** 이봐 `<Footer>`, 렌더링하고 싶은 게 뭐야?
  - `Footer`: 나는 `<footer>` 안에 텍스트를 렌더링할거야.
- **React:** 좋아, 결과물은 이거야:

```jsx
// DOM 구조물 결과
<div>
  <article>
    Some text
    <footer>some more text</footer>
  </article>
</div>
```

이것이 조정reconciliation 과정이 재귀적이라고 표현하는 이유입니다. React가 엘리먼트 트리를 지나가면서 `타입`이 컴포넌트인 엘리먼트를 만나면, 컴포넌트 함수를 호출한 다음 반환된 React 엘리먼트의 트리로 다시 탐색해 들어갑니다. 트리에 컴포넌트가 하나도 남지 않으면 이제 React는 호스트 트리를 어떻게 바꿔야 할지 알게 되는 것이죠.

이러한 조정 규칙이 여기서도 적용됩니다. (자식들 사이에서의 순서와 `key`로 판단된) 위치가 같은데 `type`이 다르면, React는 호스트 인스턴스를 삭제하고 다시 생성합니다.

## 제어 역전

이쯤에서 한 가지 의문이 생길지도 모르겠네요. 왜 컴포넌트를 직접 호출하지 않는 걸까? 왜 `Form()`이 아니라 `<Form />`이라고 쓰는 걸까?

**컴포넌트에 대해 좀 더 일찍 "알게" 되면 React가 좀 더 일을 효율적으로 할 수 있기 때문입니다. (함수를 재귀적으로 호출한 뒤에야 React 엘리먼트 트리에 대해 알게 되는 것과 비교해서요.) **

```jsx
// 🔴 개발자가 직접 호출하면, React는 Layout과 Article의 존재 여부를 알 수 없습니다.
ReactDOM.render(
  Layout({ children: Article() }),
  domContainer
)

// ✅ React가 호출하면, Layout과 Article이 존재함을 React가 알 수 있습니다.
ReactDOM.render(
  <Layout><Article /></Layout>,
  domContainer
)
```

이는 [제어 역전inversion of control](https://en.wikipedia.org/wiki/Inversion_of_control)의 전형적인 예입니다. React에게 컴포넌트 호출의 역할을 넘김으로써 우리가 얻을 수 있는 몇 가지 흥미로운 점은:

* **컴포넌트가 단순한 함수 그 이상이 됩니다.** React는 컴포넌트 함수에 *지역 상태local state* 와 같은 기능을 붙일 수 있습니다. 좋은 런타임은 그 런타임의 사용자가 직면할 만한 문제에 맞는 기본적인 추상화를 제공해야 합니다. 이미 설명했듯, React는 UI 트리를 렌더링하고 사용자 인터랙션에 반응하는 프로그램에 특화된 런타임입니다. 컴포넌트를 개발자가 직접 호출한다면 이런 기능도 직접 구축해야 합니다.

* **컴포넌트 타입을 조정에 이용할 수 있습니다.** React가 컴포넌트 호출을 하게 함으로써, 개발자는 트리의 개념적 구조에 대해 React에게 더 알려줄 수 있습니다. 예를 들어, `<Feed>`를 렌더링하는 페이지에서 `<Profile>`을 렌더링하는 페이지로 이동했다고 해봅시다. React는 이 컴포넌트들 내부에 있는 호스트 인스턴스를 재사용하려고 하지 않습니다. `<button>`이 `<p>`로 바뀌었을 때처럼 말이죠. 모든 상태는 초기화될텐데, 개발자가 다른 뷰를 렌더링하는 상황이기 때문에 이는 대체로 옳은 선택입니다. 트리 안에서 `<input>`의 위치가 우연히 일치하더라도, `<PasswordForm>`과 `<MessengerChat>` 사이에서 인풋 입력이 보존되길 원하진 않을 테니까요.

* **조정을 미룰 수 있습니다.** React가 컴포넌트 호출을 맡으면 여러 흥미로운 일을 할 수 있습니다. 예를 들면, 컴포넌트 호출과 호출 사이에 브라우저가 몇 가지 작업을 할 수 있게 함으로써, 거대한 컴포넌트 트리의 리렌더링이 [메인 쓰레드를 멈추지 않게](https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html) 할 수 있습니다. React를 여러 군데 재작성하지 않는 한, 이를 개발자가 직접 하는 건 무척 어렵습니다.

* **디버깅이 더 편해집니다.** React가 컴포넌트를 일급 객체first-class citizens로 인지할 수 있으면, [풍부한 개발자 도구](https://github.com/facebook/react-devtools)를 만들어낼 수 있습니다.

React가 컴포넌트 함수를 호출하게 할 때의 마지막 이득은 *지연 연산lazy evaluation*입니다. 이게 무슨 뜻인지 알아보시죠.

## Lazy Evaluation

When we call functions in JavaScript, arguments are evaluated before the call:

```jsx
// (2) This gets computed second
eat(
  // (1) This gets computed first
  prepareMeal()
);
```

This is usually what JavaScript developers expect because JavaScript functions can have implicit side effects. It would be surprising if we called a function, but it wouldn’t execute until its result gets somehow “used” in JavaScript.

However, React components are [relatively](#purity) pure. There is absolutely no need to execute it if we know its result won’t get rendered on the screen.

Consider this component putting `<Comments>` inside a `<Page>`:

```jsx{11}
function Story({ currentUser }) {
  // return {
  //   type: Page,
  //   props: {
  //     user: currentUser,
  //     children: { type: Comments, props: {} }
  //   }
  // }
  return (
    <Page user={currentUser}>
      <Comments />
    </Page>
  );
}
```

The `Page` component can render the children given to it inside some `Layout`:

```jsx{4}
function Page({ currentUser, children }) {
  return (
    <Layout>
      {children}
    </Layout>
  );
}
```

*(`<A><B /></A>` in JSX is the same as `<A children={<B />} />`.)*

But what if it has an early exit condition?

```jsx{2-4}
function Page({ currentUser, children }) {
  if (!currentUser.isLoggedIn) {
    return <h1>Please login</h1>;
  }
  return (
    <Layout>
      {children}
    </Layout>
  );
}
```

If we called `Comments()` as a function, it would execute immediately regardless of whether `Page` wants to render them or not:

```jsx{4,8}
// {
//   type: Page,
//   props: {
//     children: Comments() // Always runs!
//   }
// }
<Page>
  {Comments()}
</Page>
```

But if we pass a React element, we don’t execute `Comments` ourselves at all:

```jsx{4,8}
// {
//   type: Page,
//   props: {
//     children: { type: Comments }
//   }
// }
<Page>
  <Comments />
</Page>
```

This lets React decide when and *whether* to call it. If our `Page` component ignores its `children` prop and renders
`<h1>Please login</h1>` instead, React won’t even attempt to call the `Comments` function. What’s the point?

This is good because it both lets us avoid unnecessary rendering work that would be thrown away, and makes the code less fragile. (We don’t care if `Comments` throws or not when the user is logged out — it won’t be called.)

## State

We’ve talked [earlier](#reconciliation) about identity and how element’s conceptual “position” in the tree tells React whether to re-use a host instance or create a new one. Host instances can have all kinds of local state: focus, selection, input, etc. We want to preserve this state between updates that conceptually render the same UI. We also want to predictably destroy it when we render something conceptually different (such as moving from `<SignupForm>` to `<MessengerChat>`).

**Local state is so useful that React lets *your own* components have it too.** Components are still functions but React augments them with features that are useful for UIs. Local state tied to the position in the tree is one of these features.

We call these features *Hooks*. For example, `useState` is a Hook.

```jsx{2,6,7}
function Example() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

It returns a pair of values: the current state and a function that updates it.

The [array destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring) syntax lets us give arbitrary names to our state variables. For example, I called this pair `count` and `setCount`, but it could’ve been a `banana` and `setBanana`. In the text below, I will use `setState` to refer to the second value regardless of its actual name in the specific examples.

*(You can learn more about `useState` and other Hooks provided by React [here](https://reactjs.org/docs/hooks-intro.html).)*

## Consistency

Even if we want to split the reconciliation process itself into [non-blocking](https://www.youtube.com/watch?v=mDdgfyRB5kg) chunks of work, we should still perform the actual host tree operations in a single synchronous swoop. This way we can ensure that the user doesn’t see a half-updated UI, and that the browser doesn’t perform unnecessary layout and style recalculation for intermediate states that the user shouldn’t see.

This is why React splits all work into the “render phase” and the “commit phase”. *Render phase* is when React calls your components and performs reconciliation. It is safe to interrupt and [in the future](https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html) will be asynchronous. *Commit phase* is when React touches the host tree. It is always synchronous.


## Memoization

When a parent schedules an update by calling `setState`, by default React reconciles its whole child subtree. This is because React can’t know whether an update in the parent would affect the child or not, and by default React opts to be consistent. This may sound very expensive but in practice it’s not a problem for small and medium-sized subtrees.

When trees get too deep or wide, you can tell React to [memoize](https://en.wikipedia.org/wiki/Memoization) a subtree and reuse previous render result during shallowly equal prop changes:

```jsx{5}
function Row({ item }) {
  // ...
}

export default React.memo(Row);
```

Now `setState` in a parent `<Table>` component would skip over reconciling `Row`s whose `item` is referentially equal to the `item` rendered last time.

You can get fine-grained memoization at the level of individual expressions with the [`useMemo()` Hook](https://reactjs.org/docs/hooks-reference.html#usememo). The cache is local to component tree position and will be destroyed together with its local state. It only holds one last item.

React intentionally doesn’t memoize components by default. Many components always receive different props so memoizing them would be a net loss.

## Raw Models

Ironically, React doesn’t use a “reactivity” system for fine-grained updates. In other words, any update at the top triggers reconciliation instead of updating just the components affected by changes.

This is an intentional design decision. [Time to interactive](https://calibreapp.com/blog/time-to-interactive/) is a crucial metric in consumer web applications, and traversing models to set up fine-grained listeners spends that precious time. Additionally, in many apps interactions tend to result either in small (button hover) or large (page transition) updates, in which case fine-grained subscriptions are a waste of memory resources.

One of the core design principles of React is that it works with raw data. If you have a bunch of JavaScript objects received from the network, you can pump them directly into your components with no preprocessing. There are no gotchas about which properties you can access, or unexpected performance cliffs when a structure slightly changes. React rendering is O(*view size*) rather than O(*model size*), and you can significantly cut the *view size* with [windowing](https://react-window.now.sh/#/examples/list/fixed-size).

There are some kinds of applications where fine-grained subscriptions are beneficial — such as stock tickers. This is a rare example of “everything constantly updating at the same time”. While imperative escape hatches can help optimize such code, React might not be the best fit for this use case. Still, you can implement your own fine-grained subscription system on top of React.

**Note that there are common performance issues that even fine-grained subscriptions and “reactivity” systems can’t solve.** For example, rendering a *new* deep tree (which happens on every page transition) without blocking the browser. Change tracking doesn’t make it faster — it makes it slower because we have to do more work to set up subscriptions. Another problem is that we have to wait for data before we can start rendering the view. In React, we aim to solve both of these problems with [Concurrent Rendering](https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html).


## Batching

Several components may want to update state in response to the same event. This example is convoluted but it illustrates a common pattern:

```jsx{4,14}
function Parent() {
  let [count, setCount] = useState(0);
  return (
    <div onClick={() => setCount(count + 1)}>
      Parent clicked {count} times
      <Child />
    </div>
  );
}

function Child() {
  let [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      Child clicked {count} times
    </button>
  );
}
```

When an event is dispatched, the child’s `onClick` fires first (triggering its `setState`). Then the parent calls `setState` in its own `onClick` handler.

If React immediately re-rendered components in response to `setState` calls, we’d end up rendering the child twice:

```jsx{4,8}
*** Entering React's browser click event handler ***
Child (onClick)
  - setState
  - re-render Child // 😞 unnecessary
Parent (onClick)
  - setState
  - re-render Parent
  - re-render Child
*** Exiting React's browser click event handler ***
```

The first `Child` render would be wasted. And we couldn’t make React skip rendering `Child` for the second time because the `Parent` might pass some different data to it based on its updated state.

**This is why React batches updates inside event handlers:**

```jsx
*** Entering React's browser click event handler ***
Child (onClick)
  - setState
Parent (onClick)
  - setState
*** Processing state updates                     ***
  - re-render Parent
  - re-render Child
*** Exiting React's browser click event handler  ***
```

The `setState` calls in components wouldn’t immediately cause a re-render. Instead, React would execute all event handlers first, and then trigger a single re-render batching all of those updates together.

Batching is good for performance but can be surprising if you write code like:

```jsx
  const [count, setCounter] = useState(0);

  function increment() {
    setCounter(count + 1);
  }

  function handleClick() {
    increment();
    increment();
    increment();
  }
```

If we start with `count` set to `0`, these would just be three `setCount(1)` calls. To fix this, `setState` provides an overload that accepts an “updater” function:

```jsx
  const [count, setCounter] = useState(0);

  function increment() {
    setCounter(c => c + 1);
  }

  function handleClick() {
    increment();
    increment();
    increment();
  }
```

React would put the updater functions in a queue, and later run them in sequence, resulting in a re-render with `count` set to `3`.

When state logic gets more complex than a few `setState` calls, I recommend to express it as a local state reducer with the [`useReducer` Hook](https://reactjs.org/docs/hooks-reference.html#usereducer). It’s like an evolution of this “updater” pattern where each update is given a name:

```jsx
  const [counter, dispatch] = useReducer((state, action) => {
    if (action === 'increment') {
      return state + 1;
    } else {
      return state;
    }
  }, 0);

  function handleClick() {
    dispatch('increment');
    dispatch('increment');
    dispatch('increment');
  }
```

The `action` argument can be anything, although an object is a common choice.

## Call Tree

A programming language runtime usually has a [call stack](https://medium.freecodecamp.org/understanding-the-javascript-call-stack-861e41ae61d4). When a function `a()` calls `b()` which itself calls `c()`, somewhere in the JavaScript engine there’s a data structure like `[a, b, c]` that “keeps track” of where you are and what code to execute next. Once you exit out of `c`, its call stack frame is gone — poof! It’s not needed anymore. We jump back into `b`. By the time we exit `a`, the call stack is empty.

Of course, React itself runs in JavaScript and obeys JavaScript rules. But we can imagine that internally React has some kind of its own call stack to remember which component we are currently rendering, e.g. `[App, Page, Layout, Article /* we're here */]`.

React is different from a general purpose language runtime because it’s aimed at rendering UI trees. These trees need to “stay alive” for us to interact with them. The DOM doesn’t disappear after our first `ReactDOM.render()` call.

This may be stretching the metaphor but I like to think of React components as being in a “call tree” rather than just a “call stack”. When we go “out” of the `Article` component, its React “call tree” frame doesn’t get destroyed. We need to keep the local state and references to the host instances [somewhere](https://medium.com/react-in-depth/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-67f1014d0eb7).

These “call tree” frames *are* destroyed along with their local state and host instances, but only when the [reconciliation](#reconciliation) rules say it’s necessary. If you ever read React source, you might have seen these frames being referred to as [Fibers](https://en.wikipedia.org/wiki/Fiber_(computer_science)).

Fibers are where the local state actually lives. When state is updated, React marks the Fibers below as needing reconciliation, and calls those components.

## Context

In React, we pass things down to other components as props. Sometimes, the majority of components need the same thing — for example, the currently chosen visual theme. It gets cumbersome to pass it down through every level.

In React, this is solved by [Context](https://reactjs.org/docs/context.html). It is essentially like [dynamic scoping](http://wiki.c2.com/?DynamicScoping) for components. It’s like a wormhole that lets you put something on the top, and have every child at the bottom be able to read it and re-render when it changes.

```jsx
const ThemeContext = React.createContext(
  'light' // Default value as a fallback
);

function DarkApp() {
  return (
    <ThemeContext.Provider value="dark">
      <MyComponents />
    </ThemeContext.Provider>
  );
}

function SomeDeeplyNestedChild() {
  // Depends on where the child is rendered
  const theme = useContext(ThemeContext);
  // ...
}
```

When `SomeDeeplyNestedChild` renders, `useContext(ThemeContext)` will look for the closest `<ThemeContext.Provider>` above it in the tree, and use its `value`.

(In practice, React maintains a context stack while it renders.)

If there’s no `ThemeContext.Provider` above, the result of `useContext(ThemeContext)` call will be the default value specified in the `createContext()` call. In our example, it is `'light'`.


## Effects

We mentioned earlier that React components shouldn’t have observable side effects during rendering. But side effects are sometimes necessary. We may want to manage focus, draw on a canvas, subscribe to a data source, and so on.

In React, this is done by declaring an effect:

```jsx{4-6}
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

When possible, React defers executing effects until after the browser re-paints the screen. This is good because code like data source subscriptions shouldn’t hurt [time to interactive](https://calibreapp.com/blog/time-to-interactive/) and [time to first paint](https://developers.google.com/web/tools/lighthouse/audits/first-meaningful-paint). (There's a [rarely used](https://reactjs.org/docs/hooks-reference.html#uselayouteffect) Hook that lets you opt out of that behavior and do things synchronously. Avoid it.)

Effects don’t just run once. They run both after component is shown to the user for the first time, and after it updates. Effects can close over current props and state, such as with `count` in the above example.

Effects may require cleanup, such as in case of subscriptions. To clean up after itself, an effect can return a function:

```jsx
  useEffect(() => {
    DataSource.addSubscription(handleChange);
    return () => DataSource.removeSubscription(handleChange);
  });
```

React will execute the returned function before applying this effect the next time, and also before the component is destroyed.

Sometimes, re-running the effect on every render can be undesirable. You can tell React to [skip](https://reactjs.org/docs/hooks-effect.html#tip-optimizing-performance-by-skipping-effects) applying an effect if certain variables didn’t change:

```jsx{3}
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  }, [count]);
```

However, it is often a premature optimization and can lead to problems if you’re not familiar with how JavaScript closures work.

For example, this code is buggy:

```jsx
  useEffect(() => {
    DataSource.addSubscription(handleChange);
    return () => DataSource.removeSubscription(handleChange);
  }, []);
```

It is buggy because `[]` says “don’t ever re-execute this effect”. But the effect closes over `handleChange` which is defined outside of it. And `handleChange` might reference any props or state:

```jsx
  function handleChange() {
    console.log(count);
  }
```

If we never let the effect re-run, `handleChange` will keep pointing at the version from the first render, and `count` will always be `0` inside of it.

To solve this, make sure that if you specify the dependency array, it includes **all** things that can change, including the functions:

```jsx{4}
  useEffect(() => {
    DataSource.addSubscription(handleChange);
    return () => DataSource.removeSubscription(handleChange);
  }, [handleChange]);
```

Depending on your code, you might still see unnecessary resubscriptions because `handleChange` itself is different on every render. The [`useCallback`](https://reactjs.org/docs/hooks-reference.html#usecallback) Hook can help you with that. Alternatively, you can just let it re-subscribe. For example, browser’s `addEventListener` API is extremely fast, and jumping through hoops to avoid calling it might cause more problems than it’s worth.

*(You can learn more about `useEffect` and other Hooks provided by React [here](https://reactjs.org/docs/hooks-effect.html).)*

## Custom Hooks

Since Hooks like `useState` and `useEffect` are function calls, we can compose them into our own Hooks:

```jsx{2,8}
function MyResponsiveComponent() {
  const width = useWindowWidth(); // Our custom Hook
  return (
    <p>Window width is {width}</p>
  );
}

function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  });
  return width;
}
```

Custom Hooks let different components share reusable stateful logic. Note that the *state itself* is not shared. Each call to a Hook declares its own isolated state.

*(You can learn more about writing your own Hooks [here](https://reactjs.org/docs/hooks-custom.html).)*

## Static Use Order

You can think of `useState` as a syntax for defining a “React state variable”. It’s not *really* a syntax, of course. We’re still writing JavaScript. But we are looking at React as a runtime environment, and because React tailors JavaScript to describing UI trees, its features sometimes live closer to the language space.

If `use` *was* a syntax, it would make sense for it to be at the top level:

```jsx{3}
// 😉 Note: not a real syntax
component Example(props) {
  const [count, setCount] = use State(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

What would putting it into a condition or a callback or outside a component even mean?

```jsx
// 😉 Note: not a real syntax

// This is local state... of what?
const [count, setCount] = use State(0);

component Example() {
  if (condition) {
    // What happens to it when condition is false?
    const [count, setCount] = use State(0);
  }

  function handleClick() {
    // What happens to it when we leave a function?
    // How is this different from a variable?
    const [count, setCount] = use State(0);
  }
```

React state is local to the *component* and its identity in the tree. If `use` was a real syntax it would make sense to scope it to the top-level of a component too:


```jsx
// 😉 Note: not a real syntax
component Example(props) {
  // Only valid here
  const [count, setCount] = use State(0);

  if (condition) {
    // This would be a syntax error
    const [count, setCount] = use State(0);
  }
```

This is similar to how `import` only works at the top level of a module.

**Of course, `use` is not actually a syntax.** (It wouldn’t bring much benefit and would create a lot of friction.)

However, React *does* expect that all calls to Hooks happen only at the top level of a component and unconditionally. These [Rules of Hooks](https://reactjs.org/docs/hooks-rules.html) can be enforced with [a linter plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks). There have been heated arguments about this design choice but in practice I haven’t seen it confusing people. I also wrote about why commonly proposed alternative [don’t work](https://overreacted.io/why-do-hooks-rely-on-call-order/).

Internally, Hooks are implemented as [linked lists](https://dev.to/aspittel/thank-u-next-an-introduction-to-linked-lists-4pph). When you call `useState`, we move the pointer to the next item. When we exit the component’s [“call tree” frame](#call-tree), we save the resulting list there until the next render.

[This article](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e) provides a simplified explanation for how Hooks work internally. Arrays might be an easier mental model than linked lists:


```jsx
// Pseudocode
let hooks, i;
function useState() {
  i++;
  if (hooks[i]) {
    // Next renders
    return hooks[i];
  }
  // First render
  hooks.push(...);
}

// Prepare to render
i = -1;
hooks = fiber.hooks || [];
// Call the component
YourComponent();
// Remember the state of Hooks
fiber.hooks = hooks;
```

*(If you’re curious, the real code is [here](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberHooks.js).)*

This is roughly how each `useState()` call gets the right state. As we’ve learned [earlier](#reconciliation), “matching things up” isn’t new to React — reconciliation relies on the elements matching up between renders in a similar way.

## What’s Left Out

We’ve touched on pretty much all important aspects of the React runtime environment. If you finished this page, you probably know React in more detail than 90% of its users. And there’s nothing wrong with that!

There are some parts I left out — mostly because they’re unclear even to us. React doesn’t currently have a good story for multipass rendering, i.e. when the parent render needs information about the children. Also, the [error handling API](https://reactjs.org/docs/error-boundaries.html) doesn’t yet have a Hooks version. It’s possible that these two problems can be solved together. Concurrent Mode is not stable yet, and there are interesting questions about how Suspense fits into this picture. Maybe I’ll do a follow-up when they’re fleshed out and Suspense is ready for more than [lazy loading](https://reactjs.org/blog/2018/10/23/react-v-16-6.html#reactlazy-code-splitting-with-suspense).

**I think it speaks to the success of React’s API that you can get very far without ever thinking about most of these topics.** Good defaults like the reconciliation heuristics do the right thing in most cases. Warnings like the `key` warning nudge you when you risk shooting yourself in the foot.

If you’re a UI library nerd, I hope this post was somewhat entertaining and clarified how React works in more depth. Or maybe you decided React is too complicated and you’ll never look it again. In either case, I’d love to hear from you on Twitter! Thank you for reading.
