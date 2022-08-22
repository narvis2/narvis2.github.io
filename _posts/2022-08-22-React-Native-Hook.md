---
title: React-Native Hook
author: Narvis2
date: 2022-08-22 13:37:00 +0900
categories: [React-Native]
tags: [react-native, hook]
---

안녕하세요. Narvis2입니다.  
이번 시간에는 React-Native Hook 에 관하여 알아볼까 합니다.  
React 에는 use로 시작하는 다양한 함수가 내장되어 있는데, 이 함수들을 Hook이라고 부릅니다.  
**Hook을 사용하여 상태 관리, 최적화, 컴포넌트 작동 흐름 관리 등 다양한 기능을 구현할 수 있습니다.**

## 🚩 Hook 의 규칙
---
1. `Hook`은 Component 최상위 레벨에서만 사용해야 합니다.
> **_주의❗️_** `Hook`은 조건문이나 반복문 또는 중첩 함수에서 호출하면 안됩니다.
>> 만약 함수의 흐름 중간에 `return`을 하는 경우에는 `Hook`은 함수가 `return`되기 전에 사용되어야 합니다.
2. 여러 `Hook`을 사용하여 직접 `Hook`을 만들 수 있습니다.
> 이를 `Custom Hook`이라고 부릅니다. `react`패키지외에서 불러오는 Hook은 모두 `Custom Hook`입니다.
3. `Hook`은 `Custom Hook` 또는 함수 Component 에서만 사용할 수 있습니다.
> **_주의❗️_** Class에서는 사용이 불가능하며, `react`와 관련없는 일반적인 `JavaScript` 함수에서 사용하면 Error가 발생합니다.

## useState - 상태 값을 관리하는 함수
---
- useState 가 호출되면 두 가지 원소가 들어있는 배열을 반환합니다.
> - 첫 번째 원소 : 상태 값
> - 두 번째 원소 : 상태를 업데이트하는 함수
- 해당 코드에서 사용한 문법은 `배열 구조 분해 할당`을 사용중이며, Button Click 시 visible을 바꿔주는 코드입니다.👇


```javascript
import React, {useState} from 'react';
import {SafeAreaView, Button} from 'react-native';

function App() {
    const [visible, setVisible] = useState(true);
    const onPress = () => {
        setVisible(!visible);
    };

    return (
        <SafeAreaView>
            <Button title="토글" onPress={onPress} />
        </SafeAreaView>
    );
};
```
- useState 를 사용해 Boolean 뿐만 아니라 숫자, 객체, 배열 등의 형태를 가진 상태를 관리할 수도 있습니다.

## 🚩 useEffect - 특정 값을 Observing 하는 함수
---
- Component에서 특정 상태가 바뀔 때마다 원하는 코드를 실행할 수 있는 함수
- Component가 마운트(가장 처음 화면에 나타남)되거나 언마운트(화면에서 컴포넌트가 사라짐)될 때 원하는 코드를 실행할 수도 있습니다.
> - 첫 번째 인자 : 주시하고 싶은 값이 바뀌었을 때 호출하고 싶은 함수를 넣습니다.
> - 두 번째 인자 : 주시하고 싶은 값을 배열 안에 넣습니다.
- 예시 todos를 옵저빙하다가 todos가 바뀌면 log를 찍는 코드👇


```javascript
import React, {useState, useEffect} from 'react';

function App() {
    const today = new Date();

    const [todos, setTodos] = useState([
        {id: 1, text: 'Hello', done: true},
        {id: 2, text: 'Hello 2', done: false},
        {id: 3, text: 'hello 3', done: false},
    ]);

    useEffect(() => {
        console.log(todos);
    }, [todos]);
}
```
- useEffect에 등록한 함수는 두 번째 인자의 배열 안에 넣은 값이 바뀔 때도 호출되지만, Component가 가장 처음 렌더링됐을 때도 호출됩니다.

## 🚩 useNavigation - 내비게이션 Hooks
---
- Screen 으로 사용되고 있지 않은 컴포넌트에서도 `navigation`객체를 사용할 수 있습니다.


```javascript
import {useNavigation} from '@react-navigation/native';

function Test() {
    const navigation = useNavigation();

    retrun (
        <Button
            title= "Detail 열기"
            onPress={() => navigation.navigate('Detail', {id: 1})} 
        />
    );
};
```

## 🚩 useRoute - 내비게이션 Hooks
---
- useNavigation과 비슷하게, Screen이 아닌 Component에서 `route`객체를 사용할 수 있게 됩니다.
> Android 에서 Fragment/Activity 간 데이터를 주고 받는다 생각하면 됩니다.


```javascript
import {useRoute} from '@react-navigation/native';

function Test() {
    const route = useRoute();
    return <Text>id: {rounte.params.id}</Text>;
}
```

## 🚩 useFocusEffect - 내비게이션 Hooks
---
- 화면에 포커스가 잡혔을 때 특정 작업을 할 수 있게 하는 `Hook`입니다.
- 다른 화면을 열었다가 돌아왔을 때 특정 작업을 하고 싶다면 `useFocusEffect` `Hook`을 사용해야 합니다.
- 현재 화면에서 다른 화면으로 넘어갈 때 특정 작업을 하고 싶다면 `useFocusEffect`에서 함수를 만들어 반환하면 됩니다.
> **_주의❗️_** `useFocusEffect`는 꼭 `useCallback`과 같이 사용해야 합니다.
>> 만약 `useCallback`을 사용하지 않으면 Component가 리렌더링될 때마다 `useFocusEffect`에 등록한 함수가 호출됩니다.  
>> `useCallback` : Component 내부에서 함수를 만들 때, 새로 만든 함수를 사용하지 않고 이전에 만든 함수를 다시 사용하도록 만들어줍니다.
>> 그리고 그 함수 내부의 로직에서 의존하는 값이 있다면 의존하는 값이 바뀌었을 때 함수를 교체할 수 있도록 해줍니다.


```javascript
import React, {useEffect, useCallback} from 'react';
import {createMaterialBottomTabNavigator} from '@react-navigation/material-bottom-tabs';
import {Text, Button, View} from 'react-native';
import Icon from 'react-native-vector-icons/MaterialIcons';
import {useNavigation, useFocusEffect} from '@react-navigation/native';

const Tab = createMaterialBottomTabNavigator();

function OpenDetailButton() {
    const navigation = useNavigation();

    return (
        <Button
            title="Detail 1 열기"
            onPress={() => navigation.push('Deatil', {id: 1})} 
        />
    );
};

function HomeScreen() {
    useFocusEffect(
        useCallback(() => {
            console.log('이 화면을 보고 있어요.');
            return () => {
                console.log('다른 화면으로 넘어갔어요.');
            }
        }, []),
    );

    retrun (
        <View>
            <Text>Home</Text>
            <OpenDetailButton />
        </View>
    )
};
```

## 🚩 useMemo - 메모리 최적화
---
- 메모이제이션해 최적화할 수 있습니다.
> **메모이제이션** : 동일한 계산을 반복해야 할 때 불필요한 연산을 제거하기 위해 이전에 계산한 값을 재사용해 처리를 최적화하는 것을 의미합니다.
- 예제 👇
  
``` javascript
const value = useMemo(() => compute(a, b), [a, b]);
```
- 설명 👉 `a`, `b`의 값이 변경될 때만 값이 연산됩니다.
- 전체 예제 👇
  
``` javascript
import {fromat} from 'date-fns';
import React, {useContext, useMemo, useState} from 'react';
import CalendarView from '../components/CalendarView';
import FeedList from '../components/FeedList';
import LogContext from '../contexts/LogContext';

function CalendarScreen() {
    const {logs} = useContext(LogContext);
    const [selectedDate, setSelectedDate] = useState(format(new Date(), 'yyyy-MM-dd'),);

    const markedDates = useMemo( () =>
        logs.reduce((acc, current) => {
            const formattedDate = format(new Date(current.date), 'yyyy-MM-dd');
            acc[formattedDate] = {marked: true};

            return acc;
        }, {}),
        [logs],
    );
}
```
- 설명 👉 logs 배열이 바뀔 때만 logs.reduce 함수가 수행됩니다.

## 🚩 useReducer - 여러가지 상태를 관리
---
- 상태를 관리할 때 사용할 수 있는 또 다른 Hook 함수 입니다.
- state: 상태
- action: 변화를 정의하는 객체
- reducer: state와 action을 파라미터로 받아와 그다음 상태를 반환하는 함수입니다.
- dispatch: action을 발생시키는 함수
- `useState`를 여러번 사용하는 상황에서 사용하면 유용할 수 있습니다. (무조건 이 Hook을 사용할 필요는 없음)
- **각기 다른 상태를 동시에 업데이트하는 상황에서 `useReducer`를 고민해보면 좋습니다.** 👇
  
``` javascript
const onPressDate = () => {
    // 버튼 클릭 시 useState 함수를 2개 사용
    setMode('date');
    setVisible(true);
};
```
- 카운터를 useReducer로 구현하는 예제 👇
  
``` javascript
const initialState = {value: 1};

// state: 현재 상태, action: dispatch 함수의 인자로 넣은 액션 객체를 가리킵니다.
function reducer(state, action) {
    swich (action.type) {
        case 'increase':
            return {value: state.value + 1};
        case 'decrease':
            return {value: state.value - 1};
        default:
            throw new Error('Unhandled action type');
    }
}

function Counter() {
    const [state, dispatch] = useReducer(reducer, initialState);

    const onIncrase = () => dispatch({type: 'increase'});
    const onDecrease = () => dispatch({type: 'decrease'});

    return (...)
}
```
- 상태를 업데이트하는 로직을 Component 바깥에 구현할 수 있다는 장점이 있습니다.
- dispatch라는 함수 하나로 다양하게 업데이트할 수 있기 때문에 Context와 함께 사용하면 유용합니다.
> - useReducer의 첫 번째 인자 : reducer 함수를 넣습니다. 
> - useReducer의 두 번째 인자 : 상태의 초기값 입니다.
> - 함수의 결과 물의 첫 번째 원소 : 현재 상태
> - 함수의 결과 물의 두 번째 원소 : dispatch 함수
> - reducer에서 반환하는 값 : 그다음 업데이트할 값으로 사용


## 마치며
---
이번시간에는 React-Native Hook에 관하여 알아보았습니다.  
해당 포스팅은 React-Navive에 대하여 처음 공부하면서 작성한 글입니다.  
추후 계속 업데이트될 예정입니다.