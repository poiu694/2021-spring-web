# [DSC-Web] React Study [2021-04-25]

------------------------------------------------------------------------------------------------------------------------------

## Use KAKAO MAP API

- ### 카카오맵 API 받아오기

  - [DevKakao](https://developers.kakao.com/)
  - 여기에서 JS KEY로 MAP API를 가져올 수 있다.

- ### 카카오맵 띄우기

  - [카카오](https://apis.map.kakao.com/web/guide/)에 자세하게 설명이 나와있다.

  - ```react
    var container = document.getElementById('map'); //지도를 담을 영역의 DOM 레퍼런스
    var options = { //지도를 생성할 때 필요한 기본 옵션
        center: new kakao.maps.LatLng(33.450701, 126.570667), //지도의 중심좌표.
        level: 3 //지도의 레벨(확대, 축소 정도)
    };
    
    var map = new kakao.maps.Map(container, options); //지도 생성 및 객체 리턴
    ```

    위에서는 이렇게 하고 있지만, 나는 `React`스럽게 만들어보려고 `Hook`을 가져왔다.

    ```html
    <script type="text/javascript" src="//dapi.kakao.com/v2/maps/sdk.js?appkey=API_KEY&libraries=services"></script>
    ```

  - ```react
    const {kakao} = window;
    const [kakaoMap, setKakaoMap] = useState(null);
    
    // New kakao map
    useEffect(() =>{
        const options = {
            center: new kakao.maps.LatLng(37.517235, 127.047325),
            level: 3
        };
    
        const map = new kakao.maps.Map(container.current, options);
        setKakaoMap(map);
    
    }, [container]);
    
    return(<div id="kakao-map" ref={container}/> );
    ```

    `index.html` Header에 script를 위와 같이 추가한다.

  - `kakao`라는 props를 window에서 가져 온다.

  - 생성할 공간에 ref속성을 줘서 DOM을 관리한다.

  - `useEffect`로 Component가 Mount되고 나서 kakaoMap을 생성한다.

  - `useState`로 카카오 맵을 1개로 계속 유지한다.

  - **성공적으로 띄운 맵을 보면서 기뻐한다.**

- ### 카카오맵 검색 기능(Map.js)

  - 검색 기능을 추가하려면, services 라이브러리도 가져와야 한다.

  - 사실 **index.html**에 맨 뒤에 추가해준 부분으로 이미 추가했기에 더 해줄 필요가 없다.

  - 나는 검색을 해주는 `Head Container`를 하나 만드려고 한다.

  - 그리고 잡다한 기능을 넣어주는 `Map Container`를 만들 것이다.

  - 검색을 하려면 `HeadContainer`와 `MapContainer`에서 입력값을 공유해야 하기에 `Map.js`에서 만들어준다.

  - ```react
    const [searchSpot, setSearchSpot] = useState("");
    ```

    `Head`에는 `setSearchSpot`을 전달해주고, `Map`에는 `searchSpot`을 전달한다.

- ### 카카오맵 검색 기능(MapContainer.js)

  - 검색을 하면 검색한 곳에 Marker를 띄워서 어디가 검색되었는지 가시적으로 보이게 하려고 한다.

  - ```javascript
    // 장소 검색 객체를 생성합니다
    var ps = new kakao.maps.services.Places(); 
    
    // 키워드로 장소를 검색합니다
    ps.keywordSearch('이태원 맛집', placesSearchCB); 
    
    // 키워드 검색 완료 시 호출되는 콜백함수 입니다
    function placesSearchCB (data, status, pagination) {
        if (status === kakao.maps.services.Status.OK) {
    
            // 검색된 장소 위치를 기준으로 지도 범위를 재설정하기위해
            // LatLngBounds 객체에 좌표를 추가합니다
            var bounds = new kakao.maps.LatLngBounds();
    
            for (var i=0; i<data.length; i++) {
                displayMarker(data[i]);    
                bounds.extend(new kakao.maps.LatLng(data[i].y, data[i].x));
            }       
    
            // 검색된 장소 위치를 기준으로 지도 범위를 재설정합니다
            map.setBounds(bounds);
        } 
    }
    
    // 지도에 마커를 표시하는 함수입니다
    function displayMarker(place) {
        
        // 마커를 생성하고 지도에 표시합니다
        var marker = new kakao.maps.Marker({
            map: map,
            position: new kakao.maps.LatLng(place.y, place.x) 
        });
    }
    ```

    카카오가 제공해주는 검색 기능 JS 이다.

  - `ps`객체는 `keywordSearch`함수에서 `placesSearchCB`가 콜백이 된다.

  - `placesSearchCB`검색해서 나온 `data`를 저장하고 있다. 얘네들을 이용하자.

  - 이걸 그대로 작성했었는데, `kakaoMap`이 계속해서 새로 생겨서 문제가 났었다.

  - 그래서 `kakaoMap`을 `Map.js`에서 `useState`로 사용해서 1개로 유지한 이유이다.

  - React코드로 만든건 다음과 같다.

  - ```react
    const MapContainer = ({kakaoMap, searchSpot}) => {
        // 검색 기능
        useEffect(() =>{
            // 오류 났을 시
            if(kakaoMap === null){
                return;
            
    
            // 검색
            const placesSearchCB = (data, status, pagination) =>{
                if(status === kakao.maps.services.Status.OK){
                    // 검색된 장소 위치를 기준으로 지도 범위 재설정
                    // LatLngBounds 객체에 좌표 추가
                    let bounds = new kakao.maps.LatLngBounds();
                    for(let i=0; i<data.length; i++){
                        const pos = new kakao.maps.LatLng(data[i].y, data[i].x);
    
                        const marker = new kakao.maps.Marker({
                            map: kakaoMap,
                            position: pos
                        });
                        bounds.extend(new kakao.maps.LatLng(data[i].y, data[i].x));
                    }
        
                    kakaoMap.setBounds(bounds);
                }
            }
    
            const ps = new kakao.maps.services.Places(); 
            ps.keywordSearch(searchSpot, placesSearchCB);
    
        }, [searchSpot]);
    
        return(
            <>
            </>
        );
    }
    ```

    dep에 `searchSpot`을 넣어주어서 Update될 때마다 객체를 호출하게 해주었다.

JS -> React로 바꾸기만 하면 되는데 생각보다 어려웠다. 아직 React 구조를 만드는데 익숙하지 않아서 해매는 것 같다.

------------------------------------------------------------------------------------------------------------------------------

## React Reconciliation

`Reconciliation`은 React의 Heuristics Diffing Algorithm이다. 작동과정은 아래와 같다.

1. `JSX`가 `Babel`에 의해 트랜스파일링
2. `React.createElement`가 함수 호출에 의해 `React Element Tree` 반환
3. `Tree`를 재귀적으로 순회하면서 이전 트리와 다른 점만 반영(Virtual-DOM)

### Diffing Algorithm

#### 엘리먼트 타입이 다를 경우

- 아예 새로운 트리를 구축한다.

  - `div` -> `button` 등의 타입이 달라질 때

- 컴포넌트는 `ComponenetWillUnmount()`실행되고 새로운 트리가 만들어질 때, 새로운 DOM 노드들이 DOM에 삽입됩니다.

- 그에 따라 컴포넌트 인스턴스는 `UNSAFE_componentWillMount()`가 실행된다.

- `componentDidMount()`가 이어서 실행됩니다. 이전 트리와 연관된 모든 state는 사라집니다.

- ```html
  <div>
    <Counter />
  </div>
  
  <span>
    <Counter />
  </span>
  ```

   예를 들어 위와 같은코드이면 Counter는 아예 사라졌다가 다시 Mount된다.

#### DOM 엘리먼트 타입이 같을 경우

- 변경된 속성만 바뀐다.

- ```html
  <div style={{color: 'red', fontWeight: 'bold'}} />
  
  <div style={{color: 'green', fontWeight: 'bold'}} />
  ```

  fontWeight는 수정되지 않고, color만 수정된다.

#### 자식에 대한 재귀적 처리

- DOM 노드의 자식들을 재귀적으로 처리할 때, React는 기본적으로 동시에 두 리스트를 순회하고 차이점이 있으면 변경을 생성한다.

- `key`를 이용해서 처리한다.

  - React는 `key`를 통해 기존 트리와 이후 트리의 자식들이 일치하는지 확인한다.
    - 잘은 모르겠지만 React에서 `key`를 넣으라 넣으라 하는 이유 중 하나이지 않을까?

  ```html
  <ul>
    <li key="2015">Duke</li>
    <li key="2016">Villanova</li>
  </ul>
  
  <ul>
    <li key="2014">Connecticut</li>
    <li key="2015">Duke</li>
    <li key="2016">Villanova</li>
  </ul>
  ```

  - `key`를 비교하고 추가된 `Connecticut`을 넣어준다.
  - 해당 key는 오로지 형제 사이에서만 유일하면 되고, 전역에서 유일할 필요는 없다고 한다.

#### React에서 말하는 성능이 나빠지는 경우

- 알고리즘은 다른 컴포넌트 타입을 갖는 종속 트리들의 일치 여부를 확인하지 않는다.
  - 만약 매우 비슷한 결과물을 출력하는 두 컴포넌트를 교체하고 있다면, 그 둘을 같은 타입으로 만드는 것이 더 나을 수도 있다.
- 변하는 `key`(`Math.random()`으로 생성된 값 등)를 사용하면 많은 컴포넌트 인스턴스와 DOM 노드를 불필요하게 재생성한다.
  - 이러면 성능이 나빠지거나 자식 컴포넌트의 state가 유실될 수 있다.

------------------------------------------------------------------------------------------------------------------------------

## How does React Hooks re-renders?

[medium](https://url.kr/rij87b)을 요약하려고 한다.

- 결론적으로 React Hook은 Class Component가 Re-rendering 해주는 과정과 같다.
- 아래 글은 단순 Functional 형태에서의 Re-rendering 과정이다. Reconciliation의 내용은 들어가있지 않다.

### 먼저 Class 형태의 Re-rendering 과정을 살펴보자

- ```react
  // extreamly simplified implementation
  OverReact = (function () {
    function render(Component) {
      const instance = new Component()
      instance.render()
      return instance
    }
    class Component {
      constructor(props) {
        this.props = props
      }
      setState(state) {
        this.state = state
        this.render()
      }
    }
    return {
      render,
      Component,
    }
  })()
  ```

  OverReact에서 render는 Component 객체를 만들어서 render를 시키고 return 해준다.

  - 이때, instance.render가 실행해주는 것은 instance의 render 메소드이다.

- this.setState에서 rendering을 트리거 해준다.

- 다음은 예시 코드이다.

- ```react
  OverReact = (function () {
    function render(Component) {
      const instance = new Component()
      instance.render()
      return instance
    }
    class Component {
      constructor(props) {
        this.props = props
      }
      setState(state) {
        this.state = state
        this.render()
        // Component를 extends하기 때문에 여기는 ExtendedComponent에서 직접 구현한 render()메소드가 됩니다
      }
    }
    return {
      render,
      Component,
    }
  })()
  const { render, Component } = OverReact
  class ExtendedComponent extends Component {
    constructor(props) {
      super(props)
      this.state = {
        counter: 0,
        name: "foo",
      }
    }
  
    plusOne() {
      const { state: previousState } = this
      let { counter } = previousState
      counter = counter + 1
      this.setState(Object.assign(previousState, { counter }))
    }
    updateName(name) {
      const { state: previousState } = this
      this.setState(Object.assign(previousState, { name }))
    }
  
    render() {
      const { counter, name } = this.state
      console.log(`rendered, counter: ${counter}, name: ${name}`)
    }
  }
  // 최초의 render
  // 인스턴스는 OverReact.render method에 의해 반환 됩니다
  const instance = render(ExtendedComponent)
  // rendered, counter: 0, name: foo
  instance.plusOne()
  // rendered, counter: 1, name: foo
  instance.updateName("bar")
  // rendered, counter: 1, name: bar
  instance.plusOne()
  // rendered, counter: 2, name: bar
  instance.updateName("baz")
  // rendered, counter: 2, name: baz
  instance.plusOne()
  // rendered, counter: 3, name: baz
  ```

  setState가 instance에 액세스가 되므로, Component를 rendering을 다시 할 수 있음을 알 수 있다.

### Hook에서는 어떻게 될까?

- ```react
  // extreamly simplified implementation
  function render(Component) {
    // ToDo
    // render the Component
  }
  function useState(initialState) {
    // ToDo
    // derive currentState
    let currentState
  
    function setState(newState) {
      // ToDo
      // update currentState with newState
      // rerender the component
    }
    return [currentState, setState]
  }
  OverReact = {
    render,
    useState,
  }
  ```

  크게 달라질 부분이 없다.

- setState 부분이 Component 함수에 접근할 수 있어야 한다.

- OverReact의 render 함수를 trigger해주는 형태로 만들면 된다.

- useState가 여러개 호출되는 경우가 많기에 callId를 사용해 줄 것이다.

  - 자세한 내용은 [medium](https://url.kr/wcd3jk)에서 확인이 가능하나 나는 제대로 이해하지 못했다.

- 다음은 예시 코드이다.

```react
OverReact = (function () {
  let context = {}
  let callId = -1
  function render(Component) {
    context.Component = Component
    const instance = Component()
    instance.render()
    // render가 끝날때마다 callId를 리셋합니다
    callId = -1
    // OverReact.render밖에서 instance.render가 실행되지 않도록 다음을 실행합니다
    delete instance.render
    context.instance = instance
    return context
  }
  function useState(initialState) {
    if (!context) {
      throw new Error("hooks can not be called with out a rendering context")
    }
    if (!context.hooks) {
      context.hooks = []
    }

    callId = callId + 1

    const hooks = context.hooks
    const currentState = hooks[callId] ? hooks[callId] : initialState
    hooks[callId] = currentState
    const setState = (function () {
      const currentCallId = callId
      return function (newState) {
        hooks[currentCallId] = newState
        render(context.Component)
      }
    })()

    return [currentState, setState]
  }
  return {
    render,
    useState,
  }
})()
const { render, useState } = OverReact
function Component() {
  const [counter, setCounter] = useState(0)
  const [name, setName] = useState("foo")

  function plusOne() {
    setCounter(counter + 1)
  }

  function updateName(name) {
    setName(name)
  }
  function render() {
    console.log(`rendered, counter: ${counter}, name: ${name}`)
  }

  return {
    render,
    plusOne,
    updateName,
  }
}
// 최초의 render
// context는 OverReact.render method에 의해 반환 됩니다
const context = render(Component)
// rendered, counter: 0, name: foo
context.instance.plusOne()
// rendered, counter: 1, name: foo
context.instance.updateName("bar")
// rendered, counter: 1, name: bar
context.instance.plusOne()
// rendered, counter: 2, name: bar
context.instance.updateName("baz")
// rendered, counter: 2, name: baz
context.instance.plusOne()
// rendered, counter: 3, name: baz
```

- react에 있는 useState는 setState에서 OverReact의 render를 호출해준다.
- 그리고 OverReact는 Componenet의 Render를 호출해주면서 Re-Rendering이 된다.

------------------------------------------------------------------------------------------------------------------------------

## TO-DO

1. 연습지에 MAP에서 버튼 누르면 LIST에 추가하는 기능 만들 예정입니다!
2. 새로 검색되면 기존 마커가 안없어지는 오류 수정할 예정입니다!
3. Redux를 공부해서 적용을 해볼까 합니다!
