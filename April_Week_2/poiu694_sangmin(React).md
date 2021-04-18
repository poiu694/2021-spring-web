# [DSC-Web] React Study [2021-04-18]

------------------------------------------------------------------------------------------------------------------------------

## React Hook

## React Hook 특징

- React Hook은 **props, state, context, refs, lifecycle**에 관한 직관적인 API를 제공해준다고 한다.
- **Class Component** -> **Functional Component**로 바꾸어 준다.
- [High Order Component Pattern](https://url.kr/rg4itml)을 사용할 필요가 없이 로직을 공유 가능하게 한다.
- **Custom Hook**을 만들 수 있어 유용한 기능을 담긴 Hook을 사용자간의 공유가 가능하다.

## React Hook  종류

- ###  State Hook ( useState )

  - useState는 State를 Class가 아닌곳에서 다룰 수 있게 해준다.

  - 기본형은 다음과 같다.

    ```react
    import {useState} from "react";
    
    const [value, setValue] = useState(0);
    ```

  - useState는 두 개를 반환한다. 0-index 에서는 state, 1-index에서는 state를 변경해주는 함수를 반환한다.

  - 초기값은 첫 번째 렌더링에만 딱 한 번만 사용한다.

  - useState 예시

    ```react
    import React, { useState } from "react";
    
    const Example = () =>{
      const [count, setCount] = useState(0);
    
      return (
        <div>
          <p> {count} times</p>
          <button onClick={() => setCount(count + 1)}>
            Click
          </button>
        </div>
      );
    }
    ```

- ### Effect Hook ( useEffect )

  - 데이터를 가져오거나, DOM을 직접 조작하는 **“side effects”**를 수행한다.

  - Class형식에서 사용하던
    **componentDidMount, componentDidUpdate, componentWillUnmout**
    를 하나로 통합하여 사용한다.

  - useEffect 예시

  - ```react
    import React, { useState, useEffect } from "react";
    
    const Example = () =>{
      const [count, setCount] = useState(0);
    
      useEffect(() => {
        document.title = `click ${count} times`;
      }, [count]);
    
      return (
        <div>
          <p> {count} times</p>
          <button onClick={() => setCount(count + 1)}>
            Click
          </button>
        </div>
      );
    }
    ```

    위의 코드에서 useEffect가 해주는 역할은 DidMount, UpdateMount이다.

    즉, 중복되는 코드를 줄여서 사용할 수 있다.

  - 하지만, 내가 원치 않은 상황에서 update를 하게 되는 경우도 생길 수 있다.

    - 이를 방지해주려고 useEffect에서는 두 개의 파라미터를 사용한다.

  - 첫 번째로는 rendering이 될 때 어떻게 동작할 것인지,

    - 두 번째로는 dependency을 설정해준다.

  - 위에서 코드는 [count]로 dependency를 설정해주었다.

    이를 통해서 count가 update될 경우에만 첫번째 파라미터가 실행이 될 것이다.

  - 반대로 []로 dependency를 주게 된다면 count가 update가 되더라도

    첫번째 파라미터가 실행이 되지 않는다.

- ### Context Hook ( useContext )

  - createContext를 반환 받아 그 context의 현재값을 사용합니다.

  - 기본형은 다음과 같다.

  - ```react
    const value = useContext(MyContext);
    ```

    context의 현재값은 Hook을 호출하는 컴포넌트
    가장 가까이에 있는 MyContext.Provide에 의 해 결정된다.

  - context값이 변경되면 항상 리렌더링 된다.

    - 해결 : [메모이제이션](https://github.com/facebook/react/issues/15156#issuecomment-474590693)

- **Reducer Hook ( useReducer )**

  - useState의 대체함수.

  - 기본형은 다음과 같다.

  - ```react
    import React, {useReducer} from "react";
    
    const reducer = (state, action) =>{
      switch (action.type) {
        case 'INC':
          return state + 1;
        case 'DEC':
          return state - 1;
        default:
          return state;
      }
    }
    
    const [state, dispatch] = useReducer(reducer, initialState);
    ```

  - reducer는 위와 같이 함수로 정의한다.

  - dispatch는 action을 해주는 함수이며, 사용은 다음과 같이 한다.

  - ```react
    dispatch({type: 'INC'}); // Increment
    dispatch({type: 'DEC'}) // Decrement
    ```

    다수의 하윗값을 포함하는 복잡한 정적 로직을 만드는 경우나
    ​새로운 State가 이전 State에 의존적인 경우 주로 사용한다.

  - 자세한 업데이트를 트리거 하는 컴포넌트의 성능을 최적화 가능.
    콜백 대신 dispatch를 전달 할 수 있기 때문이다.

그 외에도 useRef, useMemo 등 여러개가 더 있다.

------------------------------------------------------------------------------------------------------------------------------

## Custom Hook 제작

- ### UseFetch

  - UseFetch는 외부 API에서 data를 가져올 때 사용한다.

  - fetch를 사용하여 넣었는데 나중에 axios를 공부하고 수정/유지 할 예정이다.

  - fetch를 사용할 때 현재는 then으로 구현했지만, async/await를 공부하고 바꿀 예정이다.

  - 코드블록

  ```react
  import React, {useState, useEffect} from "react";
  
  const UseFetch = (initialURL) =>{
      const [load, setLoad] = useState(null); 
      const [data, setData] = useState(null);
      const [error, setError] = useState(null);
      const [url, setUrl] = useState(initialURL);
  
      useEffect(() => {
          if(!url) return;
          setLoad(true);
          setData(null);
          setError(null);
  
          fetch(url)
              .then((res) => res.json())
              .then((data) => {
                  setLoad(false);
                  if(data.cod >= 400){
                      setError(data.message);
                      return;
                  }
                  setData(data);
              })
              .catch((error) =>{
                  setLoad(false);
                  setError(error);
              });
      }, [url]);
  
  
      return {data, error, load, setUrl};
  }
  
  export default UseFetch;
  ```

- ### UseLocalStorage

  - LocalStorage에 데이터를 저장하거나 가져올 때 사용한다.

  - json파일을 임시로 만들어서 데이터를 저장하고 있었다.

    - json파일에 직접 Update하는 방법을 찾다가
      [StackOverflow](https://url.kr/gyenlo)를 읽어보고 Node JS를 이용하는게 낫다고 생각했다.

  - 대신 대체로 localStorage에 저장하자고 생각했다.

  - [블로그](https://www.daleseo.com/react-hooks-use-web-storage/)에서  참고하려다 가져왔다.

  - 코드블록

    ```react
    const useLocalStorage = (key, initialState) => {
      const [state, setState] = useState(
        () => JSON.parse(window.localStorage.getItem(key)) || initialState
      );
    
      useEffect( () =>{
        window.localStorage.setItem(key, JSON.stringify(state));
      }, [key, state]);
    
      return [state, setState];
    }
    ```

------------------------------------------------------------------------------------------------------------------------------

## Route

- SPA페이지에서도 메인화면, 글쓰기 화면, 글목록 화면 등을 구현하기 위해서 만들어 진 기능이다.

- 연습지를 조금 더 짬뽕스럽게 관리할 수 있겠다.

- 기본형태는 다음과 같다.

- ```react
  import React from "react";
  import { Route } from "react-router-dom";
  import { Home, About } from "pages";
  
  
  const App = () =>{
      return (
          <>
              <Route exact path="/" component={Home}/>
              <Route path="/about" component={About}/>
          </>
      );
  }
  
  export default App;
  ```

  exact를 사용해서 기본 페이지에서 다른 페이지가 나오지 않게 만들 수 있다.

- Route로 설정하면 3개의 props를 전달 받는다.

  - **history**  : [참고자료](https://reactrouter.com/web/api/history)
  - **location** : [참고자료](https://reactrouter.com/web/api/location)
  - **match** : [참고자료](https://reactrouter.com/web/api/match)

- Route 경로에 특정 값을 넣는 방법으로는 params와 query를 사용하는 2가지 방법이 있다.

------------------------------------------------------------------------------------------------------------------------------
< 해보고 싶어요 >

Custom Hook 부분은 정말 매력적이다.

[npmjs](https://www.npmjs.com/package/react-hooks-lib#useFetchInitialUrl-initialOptions-onMount)

위에서 제공하는 기능들을 참고해서 자체 제작해보고 싶다.

연습지에 React-Select를 이용해서 카테고리별 필터링 해주는 기능을 만들어보고 싶다.

Context API에 대해 이해가 덜 되어서 공부를 더 할 예정.

리덕스도 나중에..?
