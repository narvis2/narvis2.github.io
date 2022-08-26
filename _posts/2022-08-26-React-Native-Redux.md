---
title: React-Native Redux로 전역 상태 관리하기
author: Narvis2
date: 2022-08-25 16:19:00 +0900
categories: [React-Native, Redux]
tags: [typescript, redux, react-native]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 `Redux`를 사용하여 전역 상태를 관리하는 방법에 대하여 알아보겠습니다.  
기존에는 `React`의 `Context API`를 사용하여 전역 상태를 관리했습니다.  
`Context API`를 사용하면 웬만한 기능은 모두 구현할 수 있습니다. 하지만 전역적으로 다뤄야 할 상태가 다양해지고 복잡해질수록 준비해야 할 코드가 많아집니다.  
특히 최적화가 필요한 경우에는 상태를 가져오는 `Context`와 상태를 업데이트하는 `Context`를 따로 관리해야 해서 준비할 코드의 양이 더욱 많아질 수 있습니다.
따라서 `Redux`에 관하여 알아보도록 하겠습니다.  

## 🚩 Redux (리덕스)
---
- `Redux`의 상태 업데이트 방법은 `useReducer`와 비슷합니다.
- 상태를 업데이트하는 `리듀서` 함수를 기반으로 작동합니다.
> **_✅ 참고_**
> - action: 변화를 정의하는 객체이며 `type`필드를 반드시 지니고 있어야 합니다.
> - state: 상태를 나타냅니다.
> **_주의 ❗️_** 리듀서 함수에서는 `state`와 `action`값을 참조하여 업데이트된 상태를 반환해야 합니다.  
> **_예제 👇_**
``` javascript
function reducer(state, action) {
    // action에 따라 업데이트된 상태를 반환
}
```
- `Context API`보다 더 적은 코드로 기능을 구현할 수 있고, 선응을 최적화하기에도 용이합니다.
- `미들웨어`라는 기능도 사용할 수 있습니다.

### 1. Module(모듈) 작성하기
- `Reduc`에서 상태 관리를 하기 위해 `Module`을 작성해야 합니다.
- `action 의 type`: `action type`은 문자열입니다. 여기서의 `type`은 `TypeScript`와는 다릅니다. `action`객체를 만들 때 `type`으로 사용될 값입니다.
> **_✅ 참고_** : `action type`은 주로 대문자로 선언합니다.  
> **_예제 👇_**
``` javascript
// action 의 type
const INCREASE = 'increase';
const DECREASE = 'decrease';
const INCREASE_BY = 'increase_by';
const DECREASE_BY = 'decrease_by';
```
- `action` 생성 함수
> **_✅ 참고_**
> - `action` 생성 함수는 `action` 객체를 만들어주는 함수입니다.
> - `action` 객체를 만들때마다 직접 객체를 작성하는 게 아니라, 함수를 재사용하여 생성할 수 있도록 `action 생성 함수`를 만듭니다.
> - `action 생성 함수`는 추후 `Component`에서 불러와야 하므로 `export`해야 합니다.  
> **_예제 👇_**
``` javascript
// action 생성 함수
export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });
export const increaseBy = (by) => ({ type: INCREASE_BY, by });
export const decreaseBy = (by) => ({ type: DECREASE_BY, by });
```
- 초기 상태
> **_예제 👇_**
``` javascript
const initialState = {
    value: 1
};
```
- `리듀서` 함수
> **_✅ 참고_**
> - `state`와 `action` 파라미터를 받아와서 업데이트된 상태를 반환합니다.
> - `useReducer`의 `리듀서`와 다른점은 초기 상태를 기본 파라미터 문법을 사용해 `state = initialState` 형식으로 지정해줘야 하며, `default :` 케이스에서는 오류를 발생시키지 않고 `state`를 반환하도록 만들어야 한다는 점입니다.
> - `리듀서` 함수에서는 `상태의 불변성`을 유지하면서 업데이트 해줘야 합니다. 따라서 `state`를 직접 수정하면 안 되고, 새로운 값을 만들어서 반환해줘야 합니다.
>> **_주의 ❗️_**  
>> - 객체에 여러 값이 있다면 `{...state, value: state.value + 1}`과 같은 형식으로 `불변성`을 유지해야 합니다.
>> - 만약 배열인 경우에는 `push`, `splice` 처럼 배열에 직접 변화를 일으키는 함수를 사용하지 말고 `concat`, `filter`처럼 새로운 배열을 생성시키는 함수를 사용해야 합니다.
>> - `리듀서` 함수는 추후 여러 `리듀서`를 합쳐야 하므로 이 또한 `export`해줘야 합니다. 주로 `export default`로 내보내줍니다.  
> **_예제 👇_**
``` javascript
// 리듀서 함수
export function counter(state= initialState, action) {
    switch (action.type) {
        case INCREASE:
            return { value: state.value + 1 };
        case DECREASE:
            return { value: state.value - 1};
        case INCREASE_BY:
            return { value: state.value + action.by };
        case DECREASE_BY:
            return { value: state.value - action.by };
        default:
            return state;
    }
}
```
- 전체 코드


``` javascript
// action 의 type
const INCREASE = 'increase';
const DECREASE = 'decrease';
const INCREASE_BY = 'increase_by';
const DECREASE_BY = 'decrease_by';

// action 생성 함수
export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });
export const increaseBy = (by) => ({ type: INCREASE_BY, by });
export const decreaseBy = (by) => ({ type: DECREASE_BY, by });

// 초기 상태
const initialState = {
    value: 1
};

// 리듀서 함수
export function counter(state= initialState, action) {
    switch (action.type) {
        case INCREASE:
            return { value: state.value + 1 };
        case DECREASE:
            return { value: state.value - 1};
        case INCREASE_BY:
            return { value: state.value + action.by };
        case DECREASE_BY:
            return { value: state.value - action.by };
        default:
            return state;
    }
}
```

### 2. Root 리듀서 만들기
- `Redux`를 사용할 때는 기능별로 `Module`을 작성해야 합니다.
- 각 `Module`에 있는 여러 `리듀서`를 하나의 `리듀서`로 합쳐야 합니다.
- `리듀서`를 하나로 합칠 때는 `combineReducers`라는 함수를 사용합니다.
- 여러 `리듀서`를 합친 `리듀서`를 `루트 리듀서`라고 부릅니다.


``` javascript
// 3개의 리듀서가 있다고 가정
counter : { value: 1  }
todos: [{
    id: 1, text: '리덕스', done: false
}]
setting: { filter: 'all' }

// 리듀서를 하나로 합침 combineReducers 사용
import {combineReducers} from 'redux'

// combineReducers에 합치고 싶은 reducer 함수 넣기
const rootReducer = combineReducers({
    counter,
    todos,
    setting
});

export default rootReducer;
```

### 3. Store(스토어) 만들기
- `Root Reducer`를 만든 다음에는 `store`를 만들어야 합니다.
- `React Project`에서는 단 하나의 `store`를 생성합니다.
- `Redux store`에서는 `Redux`에서 관리하는 모든 상태가 들어있으며, 현재 상태를 조회할 수 있는 `getState`함수와 `action`을 일으킬 수 있는 `dispatch`함수가 들어있습니다.
> **_✅ 참고_** : `dispatch(increase())`와 같이 `action`객체를 인자에 넣어서 함수를 호출하면 준비한 `reducer`가 호출되면서 상태가 업데이트 됩니다.  
> **_예제 👇_**  
``` javascript
import {createStore} from 'redux'
import rootReducer from './modules/'
const store = createStore(rootReducer);
export default store;
```

### 4. Provider로 React Project에 Redux 적용
- `React Project`에서 작성한 `store`를 사용하려면 `Provider Component`를 사용하여 전체 앱을 감싸줘야 합니다.


``` javascript
import {createStore} from 'redux';
import {Provider} from 'react-native';
import rootReducer from './modules';

const store = createStore(rootReducer);

function App() {
    return (
        <Provider store={store}>
            {/* .. */}
        </Provider>
    );
}

export default App;
```

### 5. useSelector과 useDispatch로 Component에서 Redux 연동하기
- `Provider`를 사용하여 `Redux`를 적용
> **_✅ 참고_**
> - `useSelector` : `Hook`을 사용해 `리덕스`의 **상태를 조회**할 수 있습니다.
>> - `useSelector Hook` 에서는 셀렉터 함수를 인자로 넣어줍니다.
>> - 이 함수에서 `state` 파라미터는 `store`가 지니고 있는 현재 상태를 가리킵니다.
>> - 셀렉터 함수에서는 이 `Component`에서 사용하고 싶은 값을 반환하면 됩니다.
> - `useDispatch` : `action`을 발생시켜 **상태를 업데이트**할 수 있습니다. 
>> - 셀렉터를 사용해 조회한 상태가 바뀔 때마다 `Component`가 렌더링됩니다.
>> - `useSelector`는 최적화되어 있기 때문에 우리가 원하는 상태가 바뀔 때만 리렌더링합니다.
>>> **_주의 ❗️_** 이 `component`에서 의존하지 않는 `Redux`에서 관리하는 다른 상태가 변경됐을 때는 렌더링되지 않습니다. 이 점이 `Context`를 사용했을 때와 가장 다른 차이점 입니다.


``` javascript
import {useSelector, useDispatch} from 'react-redux';
import {increase, decrease} from './modules/counter/';

function Counter() {
    const value = useSelector(state => state.counter.value);

    const dispatch = useDispatch();

    const onPressIncrease = () => {
        dispatch(increase());
    };

    const onPressDecrease = () => {
        dispatch(decrease()));
    };
}

export default Counter;
```

## 🚩 Redux (리덕스) 개념 정리
---
- `Redux`를 사용할 때는 상태의 종류별로 `Module`을 작성해야 합니다.
- `Module`에는 `action type`, `action 생성 함수`, `Reducer`를 선언합니다.
- 여러 `Reducer`를 `combineReducers`로 합쳐 `Root Reducers`를 만듭니다.
- `createStore`를 사용하여 `store`를 만듭니다.
- `Provider`를 사용하여 `React Project`에 `Redux`를 적용합니다.
- `useSelector`와 `useDispatch`를 사용하여 `Redux`의 상태를 조회하거나 업데이트합니다.


## 🚩 마치며
이번 포스팅에서는 `Redux`에 관하여 알아보았습니다.  
`Redux`는 `useReducer`와 비슷하며 `Redux`를 통해 전역 상태를 관리하면 더 적은양의 코드로 최적화된 상태를 관리할 수 있습니다.  
다음 포스팅에서는 `Redux Toolkit`에 대하여 알아보도록 하겠습니다.