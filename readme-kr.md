<h1 align=center style="max-width: 100%;">
  <img width="300" alt="sagen Logo" src="https://user-images.githubusercontent.com/26024412/101279836-780ddb80-3808-11eb-9ff5-69693c56373e.png" style="max-width: 100%;"><br/>
</h1>

[![Build Status](https://travis-ci.com/jungpaeng/sagen.svg?branch=main)](https://travis-ci.com/jungpaeng/sagen)
[![Maintainability](https://api.codeclimate.com/v1/badges/0c2a4ad6c9ad60f3b2cf/maintainability)](https://codeclimate.com/github/jungpaeng/sagen/maintainability)
[![Test Coverage](https://api.codeclimate.com/v1/badges/0c2a4ad6c9ad60f3b2cf/test_coverage)](https://codeclimate.com/github/jungpaeng/sagen/test_coverage)

![min](https://badgen.net/bundlephobia/min/sagen@latest)
![minzip](https://badgen.net/bundlephobia/minzip/sagen@latest)
![dependency-count](https://badgen.net/bundlephobia/dependency-count/sagen@latest)
![tree-shaking](https://badgen.net/bundlephobia/tree-shaking/sagen@latest)

[Korean](./readme-kr.md) | [English](./readme.md)

## ⚙ 설치 방법
#### npm
```bash
$ npm install --save sagen
```
#### yarn
```bash
$ yarn add sagen
```

## 🏃 시작하기

sagen은 Provider 없이 multiple store를 제공하는 상태 관리 라이브러리입니다.

#### store 만들기

store를 생성해 state를 관리할 수 있습니다.

```typescript
import { createStore } from 'sagen';

const globalStore = createStore({ num: 0, str: '' });
```

#### state 값 관리

`useGlobalStore` hook을 사용하면 `state` 값과 `setState` 함수를 반환받을 수 있습니다.

이는 `setState`와 사용 방법이 같고 동기로 동작합니다.

```jsx
import React from 'react';
import { useGlobalStore } from 'sagen';

const App = () => {
  const [state, setState] = useGlobalStore(globalStore);

  return (
    <div>
      <p>number: {state.num}</p>
      <p>string: {state.str}</p>
      <button
        onClick={() => setState(curr => ({ ...curr, num: curr.num + 1 }))}
      >
        click me
      </button>
    </div>
  );
};
```

#### middleware for sagen

**sagen은 Redux의 미들웨어를 호환합니다.**

다음은 redux의 간단한 logger middleware 입니다.

```ts
import { createStore, composeMiddleware } from 'sagen-core';

const loggerMiddleware = store => next => action => {
  console.log('현재 상태', store.getState());
  console.log('액션', action);
  next(action);
  console.log('다음 상태', store.getState());
}

const store = createStore(0, composeMiddleware(loggerMiddleware));
const [state, setState] = useGlobalStore(store);

setState(1);
```

**console log**

```console
현재 상태,  0
액션, 1
다음 상태,  1
```

## Recipes

#### state selector

state 값을 가져올 때 `selector` 함수를 넘겨줘서 state 값을 가공할 수 있습니다.

기본적으로, `===` 연산자로 기존 값과 새로운 값을 비교하기 때문에 아래와 같이 `state`에서 필요한 값만을 사용하는 것이 좋습니다.

```jsx
import React from 'react';
import { createStore, useGlobalStore } from 'sagen';

const globalStore = createStore({ num: 0, str: '' });
const numberSelector = state => state.num;
const stringSelector = state => state.str;

const NumberChild = () => {
  const [num, setValue] = useGlobalStore(globalStore, numberSelector);
  const handleClickBtn = React.useCallback(() => {
    setValue((curr) => ({
      ...curr,
      num: curr.num + 1,
    }));
  }, []);

  return (
    <div className="App">
      <p>number: {num}</p>
      <button onClick={handleClickBtn}>Click</button>
    </div>
  );
};

const StringChild = () => {
  const [str] = useGlobalStore(globalStore, stringSelector);

  return (
    <div className="App">
      <p>string: {str}</p>
    </div>
  );
};

const App = () => {
  const [number, setState] = useGlobalStore(globalStore, numberSelector);

  return (
    <div>
      <NumberChild />
      <StringChild />
    </div>
  );
};
```

#### action, dispatch

`createStore` 함수로 생성한 `store`에 `action`을 추가할 수 있습니다.

```typescript jsx
const store = createStore(0);
const storeDispatch = createDispatch(store);
const storeAction = store.setAction((getter) => ({
  INCREMENT: () => getter() + 1,
  ADD: (num) => getter() + num,
}));

const App = () => {
  const [state, setState] = useGlobalStore(store);

  return (
    <div className="App">
      <p>number state: {state}</p>
      <button onClick={() => storeDispatch(storeAction.INCREMENT)}>
        ClickMe
      </button>
      <button onClick={() => storeDispatch(storeAction.ADD, 100)}>
        ClickMe
      </button>
    </div>
  );
};
```

위와 같이 `store`에 `action`을 추가한 뒤, `dispatch`를 통해 사용할 수 있습니다.

이것은 `customSetState`를 더 쉽게 작성할 수 있게 해줍니다.

#### shallowEqual

객체 또는 배열 등 `===`로 비교힐 수 없는 값의 경우, `shallowEqual` 함수를 넘겨서 값을 비교할 수 있습니다.

```jsx
import React from 'react';
import { createStore, useGlobalStore, shallowEqual } from 'sagen';

const globalStore = createStore({ num: 0, str: '' });
const storeSelector = state => state;

const App = () => {
  const [state, setState] = useGlobalStore(globalStore, storeSelector, shallowEqual);

  return (
    <div>
      ...
    </div>
  );
};
```

#### React 없이 사용하기

`sagen`은 React 없이 사용할 수 있습니다.

[sagen-core](https://www.npmjs.com/package/sagen-core) 라이브러리를 사용해보세요.

## 📜 License
sagen is released under the [MIT license](https://github.com/jungpaeng/react-manage-global-state/blob/main/LICENSE).

```
Copyright (c) 2020 jungpaeng

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
