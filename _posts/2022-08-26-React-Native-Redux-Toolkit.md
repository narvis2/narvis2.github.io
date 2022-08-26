---
title: React-Native Redux Toolkit 사용
author: Narvis2
date: 2022-08-25 16:19:00 +0900
categories: [React-Native, Redux]
tags: [redux-toolkit, redux, react-native]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 `Redux Toolkit`에 대하여 알아보도록 하겠습니다.  
`Redux`를 사용하면 `action type`을 선언하고, `action 생성 함수`를 선언하고, `Reducer`에서 해당 `action`을 위한 케이스를 작성해야 합니다. 그리고 `Module`의 상태가 복잡해지면 불변성을 유지하면서 상태를 업데이트하기 위해 적상해야 할 코드의 양도 많아질 수 있습니다.  
이런 상황에 `Redux Toolkit`을 사용하면 `Module`을 작성할 때 `action type`, `action 생성 함수`, `reducer`를 한번에 작성할 수 있어 편리합니다.  
자세한 설명은 밑에서 하도록 하겠습니다.  
`Redux`에 대하여 잘 모르시면 해당 포스팅을 참고하세요. 👉 [React-Native Redux에 대하여](https://narvis2.github.io/posts/React-Native-Redux/)

## 🚩 Redux Toolkit
---
- `Module`을 작성할 때 `action type`, `action 생성 함수`, `reducer`를 한번에 작성할 수 있어 편리합니다.
- 상태를 업데이트할 때 불변성에 대해 신경 쓰지 않아도 됩니다.
- 상태를 직접 업데이트했을 때 내장된 라이브러리 `immer`를 사용하여 불변성을 유지하면서 업데이트해줍니다.
- `createSlice` : `action 생성 함수`와 `reducer`를 합친 개념입니다.
> **_✅ 참고_**
> - `createSlice`를 사용하면 `action 생성 함수`와 `reducer`를 동시에 만들 수 있기 때문에 `action type`을 따로 선언하지 않아도 됩니다. 
> - 만약 `action type`을 조회해야 할 일이 생기면 `action 생성 함수`의 `type` 필드를 확인하면 됩니다. 
>> - `counterSlice.actions`: `action 생성 함수`들이 들어있는 객체
>> - `counterSlice.reducer`: `reducer` 함수
>> - `action`의 `payload` : `Redux Toolkit`에서는 `action`에 추가로 넣어줄 값의 이름이 `playload`로 통일되며, 액션 생성 함수의 파라미터를 통해 정해집니다.
>>> - 예) increaseBy(5) 라고 `action 생성 함수`를 호출하면 `action.payload`의 값은 `5`가 됩니다.
>> - 자동으로 이루어지는 불변성 관리
>>> - `createSlice`에서 사용하는 `Reducer` 메서드에서는 불변성을 신경쓰지 말고 `state`값을 직접 수정해도 됩니다.
>>>> - 라이브리 자체에서 `immer`라는 라이브러리가 내장되어 있어 불변성을 자동으로 관리해줍니다.
>> - `slice`를 만들 때는 `name`을 지정해야 합니다.
>>> - `name` : `action type`이 만들어질 때 `name/액션 이름` 형태로 만들어 집니다.
>>>> - 예) `slice` 의 `name`이 `counter`일 경우, `increase`의 `action type`은 `counter/increase`가 됩니다.


``` javascript
import {createSlice} from '@reduxjs/toolkit';

const initialState = {value: 1};

const counterSlice = createSlice({
    name: 'counter',
    initialState,
    reducers: {
        increase(state, action) {
            state.value += 1;
        },
        decrease(state, action) {
            state.value -= 1;
        },
        increaseBy(state, action) {
            state.value += action.playload;
        },
        decreaseBy(state, action) {
            state.value -= action.playload;
        }
    }
});

export const {increase, decrease, increaseBy, decreaseBy} = counterSlice.actions;
export default counterSlice.reducer;
```
> 위의 코드에서 increase(state, action)은 `Shorthand method names`라는 문법으로 **객체의 메서드를 선언하는 문법**입니다.  
> 해당 코드는 아래의 코드와 같습니다. 👇
``` javascript
increase: function(state, action) {
    state.value += 1;
}
```

## 🚩 Typescript 와 함꼐 Redux Toolkit 사용하기
---
- `initialState`에 타입을 달아줘야 합니다.
- `action type`에 `PlayloadAction`을 사용해야 합니다.
- `playload` 값의 `type`은 `PlayloadAction`의 `Generic`으로 지정해줄 수 있습니다.


``` typescript
import {createSlice, PlayloadAction} from '@reduxjs/toolkit';

interface CounterState {
    value: number
};

const initialState: CounterState = {value: 1};

const counterSlice = createSlice({
    name: 'counter',
    initialState,
    reducers: {
        increase(state, action) {
            state.value += 1;
        },
        decrease(state, action) {
            state.value -= 1;
        },
        increaseBy(state, action: PlayloadAction<number>) {
            state.value += action.payload;
        },
        decreaseBy(state, action: PlayloadAction<number>) {
            state.value -= action.payload;
        }
    }
});

export const {increase, decrease, increaseBy, decreaseBy} = counterSlice.actions;

export default counterSlice.reducer;
```

## 🚩 마치며
--- 
이번 시간에는 `Redux Toolkit`에 대하여 알아보고 또한 `TypeScript`을 통해 `Redux Toolkit`을 사용하는 방법에 대하여 알아보았습니다.  
다음 시간에는 더욱 유용한 글로 찾아뵙겠습니다.