# React 리렌더링

## 문제점

1초가 지날때 마다 경과시간을 표시해주는 컴포넌트가 있음
1초마다 redux store의 값을 변경해주는데, 전체 페이지 리렌더링이 계속 발생!!!

## 해결순서

1. 리액트 렌더링의 이해

   - 컴포넌트가 최초에 Mount되었을 때
   - `this.setState()`를 통해 컴포넌트의 state를 변경할 때
   - 상위 컴포넌트로부터 새로운 `props`을 전달 받을 때
   - `this.forceUpdate()`를 통해 강제로 렌더링을 할 때

2. 현재 어플리케이션이 리렌더링이 발생하는 원인 파악

   - `this.setState()`를 통해 컴포넌트의 state를 변경할 때
   - 상위 컴포넌트로부터 새로운 `props`을 전달 받을 때

   위 두개 항목이 가능성이 있으나, 현재 이슈되는 상황에서 두가지 모두 아니었음

3. 이슈 분석
   - 이슈되는 상황은 1초마다 redux store의 특정 상태값을 변경
   - `useSelector`를 사용한 Component에서는 모두 리렌더링이 발생
   - Page에서 `useSelector`를 사용하는 경우 **하위 컴포넌트는 `useSelector`를 사용하지 않아도 리렌더링 발생**
4. 해결과정

   1. Page의 `useSelector`를 하위 컴포넌트로 분리하여 `useSelector`를 사용하지 않는 하위컴포넌트가 리렌더링이 발생하지 않도록 수정
   2. 왜 useSelector를 사용한 곳에서는 항상 리렌더링이 발생하는지 원인 파악

      ### One dispatch, one virtual DOM render

      - (dispatch가 store의 상태 변경을 유발하였을 경우) **리덕스는 모든 dispatch에 대해 모든 subscriber를 업데이트함**
      - store의 reducer를 여러개로 분리하는 것과 상관없음
      - 모든 컴포넌트에게 noti를 하는 것은 맞으나, 이전 값과 비교하여 그 값이 바뀌었을 때만 리렌더링 유발함
      - `useSelector`를 사용하였을 경우에는 이전 값과 비교하지만 `===` operator로 reference비교를 하기 때문에 주의해야할 사항이 있음

      **use Selector로 object가 아닌 값을 return 하는 경우**

      ```typescript
      import React from "react";
      import { useSelector } from "react-redux";

      export const CounterComponent = () => {
        const counter = useSelector((state) => state.counter);
        return <div>{counter}</div>;
      };
      ```

      - 이 경우에 counter는 number 값이며, state.counter의 값이 바뀌지 않았다고 하면 `===` operator로 비교했을 때 `true`일 것이므로, 리렌더링 되지 않음.

      **useSelector로 object를 return하는 경우**

      ```typescript
      import React from "react";
      import { useSelector } from "react-redux";

      export const CounterComponent = () => {
        const { counter } = useSelector((state) => ({
          counter: state.counter,
        }));
        return <div>{counter}</div>;
      };
      ```

      - 이 경우에 useSelector의 인자로 들어간 함수가 반환하는 값은 항상 새로운 object이며, `===` operator로 비교했을 때 `false`일 것이므로 state.coutner가 변경되지 않았다고 할지라도 리렌더링 유발함

      **useSelect로 여러개의 값을 return하는 경우**

      ```typescript
      import React from "react";
      import { useSelector } from "react-redux";
      import { shallowEqual } from "react-redux";

      export const CounterComponent = () => {
        const { counter, max } = useSelector(
          (state) => ({
            counter: state.counter,
            max: state.max,
          }),
          shallowEqual
        );
        return <div>{counter}</div>;
      };
      ```

      - 이 경우에 react-redux의 `shallowEqual`함수를 `useSelector`의 두번째 인자로 전달
        `useSelector`의 두번째 파라미터는 `equalityFn`이며, 이전 값과 다음 값을 비교하여 `true`가 나오면 리렌더링을 하지 않고 `false`가 나오면 리렌더링을 함.
      - `shallowEqual`은 객체 안의 가장 겉에 있는 값들을 모두 비교함.
