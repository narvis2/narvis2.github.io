---
title: React-Native Redux + TypeScript
author: Narvis2
date: 2022-11-30 11:54:00 +0900
categories: [React-Native, Redux]
tags: [redux, react-native, redux-typescript]
---

안녕하세요. Narvis2 입니다.  
이번시간에는 `Redux`를 `TypeScript`와 함께 사용하는 법에 대하여 간단한 예제를 통하여 알아보도록 하겠습니다.

## 🍀 Redux + TypeScript

- `Redux` + `TypeScript` 적용 👉 `yarn add redux react-redux @types/react-redux`

  > 1️⃣ **_예제_** ) `Redux Module` 작성 👇

  ```typescript
  /**
   * ✅ Action Type
   * as const 👉 나중에 Action 객체를 만들게 action.type 의 값을 추론하는 과정에서
   * action.type 이 string 으로 추론되지 않고 'INCREASE'와 같이 실제 문자열 값으로 추론 되도록 해줌
   */
  const INCREASE = "INCREASE" as const;
  const DECREASE = "DECREASE" as const;

  // ✅ Action 생성 함수
  export const increase = () => ({ type: INCREASE });
  export const decrease = () => ({ type: DECREASE });

  /**
   * ✅ 모든 Action 객체들에 대한 타입을 준비
   * ReturnType<typeof _____> 👉 특정 함수의 반환값을 추론해줌
   * Action Type 을 선언 할 떄 as const 를 하지 않으면 이 부분이 제대로 작동하지 않음
   */
  type CounterAction =
    | ReturnType<typeof increase>
    | ReturnType<typeof decrease>;

  // ✅ 이 리덕스 모듈에서 관리 할 State의 Type을 선언
  type CounterState = {
    count: number;
  };

  // ✅ 초깃값 (상태가 객체가 아니라 그냥 숫자여도 상관 없음)
  const initialState: CounterState = {
    count: 0,
  };

  /**
   * ✅ Reducer를 작성
   * Reducer 에서는 state 와 함수의 반환값이 일치하도록 작성해야함
   * Action 에서는 CounterAction 을 타입으로 설정
   */
  export default function counter(
    state: CounterState = initialState,
    action: CounterAction
  ): CounterState {
    switch (action.type) {
      case "INCREASE":
        return { count: state.count + 1 };
      case "DECREASE":
        return { count: state.count - 1 };
      default:
        return state;
    }
  }
  ```

  > 2️⃣ **_예제_**) `Root Reducer` 작성 👇

  ```typescript
  import { combineReducers } from "redux";
  import counter from "./counter";

  // ✅ Root Reducer 생성
  const rootReducer = combineReducers({ counter });

  export default rootReducer;

  /**
   * Root Reducer 의 반환값을 유추해줌
   * 추후 이 타입을 컨테이너 Component에서 불러와서 사용해야 하므로 내보냄
   */
  export type RootState = ReturnType<typeof rootReducer>;
  ```

  > 3️⃣ **_예제_** ) `useSelector`, `useDispatch` 사용 👇

  ```typescript
  import React from "react";
  import Counter from "./Counter";
  import { RootState } from "../modules";
  import { useSelector, useDispatch } from "react-redux";
  import { increase, decrease } from "../modules/counter";

  const CounterContainer = () => {
    /**
     * ✅ 현재 상태를 조회
     * 상태를 조회할 때는 state의 타입을 RootState로 지정해야함
     */
    const counter = useSelector((state: RootState) => state.counter);
    // Dispatch 함수를 가져옴 (action을 발생시켜 상태를 업데이트)
    const dispatch = useDispatch();

    function onIncrease() {
      dispatch(increase());
    }

    function onDecrease() {
      dispatch(decrease());
    }

    const props = {
      count: counter.count,
      onIncrease,
      onDecrease,
    };

    return <Counter {...props} />;
  };

  export default CounterContainer;
  ```

  > 4️⃣ **_예제_** ) `Counter.tsx` 👇

  ```typescript
  import React from "react";
  import { Pressable, Text, View } from "react-native";

  type CounterProps = {
    count: number;
    onIncrease: () => void;
    onDecrease: () => void;
  };

  const Counter = ({ count, onIncrease, onDecrease }: CounterProps) => {
    return (
      <View
        style={{
          flexDirection: "column",
          justifyContent: "center",
          alignItems: "center",
        }}
      >
        <Text style={{ fontSize: 16 }}>{count}</Text>
        <Pressable onPress={onIncrease} style={{ padding: 20 }}>
          <Text>{+1}</Text>
        </Pressable>
        <Pressable onPress={onDecrease} style={{ padding: 20 }}>
          <Text>{-1}</Text>
        </Pressable>
      </View>
    );
  };

  export default Counter;
  ```

  > 5️⃣ **_예제_** ) `Provier`를 사용하여 `App.tsx`에 적용 👇

  ```typescript
  import { Provider } from "react-redux";
  import { createStore } from "redux";
  import CounterContainer from "./src/components/CounterContainer";
  import rootReducer from "./src/modules";

  const store = createStore(rootReducer);

  const App = () => {
    const isDarkMode = useColorScheme() === "dark";

    const backgroundStyle = {
      backgroundColor: isDarkMode ? Colors.darker : Colors.lighter,
    };

    return (
      <Provider store={store}>
        <SafeAreaView style={backgroundStyle}>
          <CounterContainer />
        </SafeAreaView>
      </Provider>
    );
  };

  export default App;
  ```
