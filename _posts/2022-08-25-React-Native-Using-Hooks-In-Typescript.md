---
title: Typescript 에서 Hook 사용하기
author: Narvis2
date: 2022-08-22 13:57:00 +0900
categories: [React-Native, TypeScript]
tags: [typescript, hook, react-native]
---

안녕하세요. NarviS2 입니다.  
지난 포스팅에서는 `TypeScript`에 관하여 기초적인 문법을 알아보았습니다.  
지난 포스팅을 안보신 분들은 👉 [Typescript에 관하여](https://narvis2.github.io/posts/TypeScript/)를 통해 참고하시길 추천드립니다.  
또한 이번 포스팅에서는 `TypeScript`로 `Hook`을 사용하는 방법을 예제를 통하여 알아보도록할텐데 `Hook`에 관하여 모르시는 분들은
👉 [React-Native Hook](https://narvis2.github.io/posts/React-Native-Hook/)을 참고하시길 권장합니다.

## 🚩 React Component를 TypeScript로 작상하기
---
- `props`에 `type`이 붙는 것 말고는 달라질 것이 없습니다. (`props` 가 없는 `Component`는 `JavaScript`로 작성할 때와 똑같습니다.)
> **_주의❗️_** `React Component`를 작성할 때는 `tsx`확장자, `JSX`를 사용하지 않는 코드는 `ts`확장자를 사용합니다.
  
### 1. Props 사용하기
- `Props`가 필요해지면 `Props`를 위한 `type`을 선언해줘야 합니다.
- `Props`를 위한 `type`은 `interface`를 사용하여 선언합니다.
- **_예제👇_**


``` typescript
import React from 'react';
import {View, Text} from 'react-native';

interface Props {
    name: string;
}

function Profile({name}: Props) {
    return (
        <View>
            <Text>{name}</Text>
        </View>
    );
}

export default Profile;
```


### 2. 옵셔널 Props 사용하기
- `interface`에서 특정 필드에 `?`를 붙여서 타입을 선언합니다.
> **_참고_** : 옵셔널 Props란 생략가능한 Props라는 뜻입니다.


``` typescript
import React from 'react';
import {View, Text, StyleSheet} from 'react-native';

interface Props {
    name: string;
    isActive?: boolean;
}

function Profile({name, isActive}: Props) {
    return (
        <View style={isActive && styles.activeStyle}>
            <Text>{name}</Text>
        </View>
    );
}

const styles = StyleSheet.create({
    activeStyle: {
        backgroundColor: 'yellow',
    },
});

export default Profile;
```
- `Profile` Component를 사용할 때 `isActive` Props를 생략해도 `TypeScript`오류가 발생하지 않습니다.

### 3. 기본값 Props 사용하기
- `defaultProps` 사용 👇


``` typescript
import React from 'react';
import {View, Text, StyleSheet, Image} from 'react-native';

interface Props {
    name: string;
    isActive?: boolean;
    image: string;
}

function Profile({name, isActive, image}: Props) {
    return (
        <View style={isActive && styles.activeStyle}>
            <Image source={{uri: image}} />
            <Text>{name}</Text>
        </View>
    );
}

Profile.defaultProps = {
    image: ''.
};

const styles = StyleSheet.create({
    activeStyle: {
        backgroundColor: 'yellow',
    },
});

export default Profile;
```

- `JavaScript` 기본값 매개변수 문법 사용
> - Props 파라미터가 구조분해되는 과정에서 특정 값의 기본값을 지정할 수 있습니다.
> - 해당 필드에 옵셔널 `?`를 붙혀주셔야 합니다.
> **_예제 👇_**


``` typescript
import React from 'react';
import {View, Text, StyleSheet, Image} from 'react-native';

interface Props {
    name: string;
    isActive?: boolean;
    image?: string;
}

function Profile({name, isActive, image='이미지 주소 넣기'}: Props) {
    return (
        <View style={isActive && styles.activeStyle}>
            <Image source={{uri: image}} />
            <Text>{name}</Text>
        </View>
    );
}

const styles = StyleSheet.create({
    activeStyle: {
        backgroundColor: 'yellow',
    },
});

export default Profile;
```

### 4. children Props 사용하기 (React.ReactNode)
- `JSX` 타입의 Props를 받아오는 상황에서는 `React.ReactNode` 타입을 사용
> **_✅ 참고_** : 만약 `children` Props가 반드시 필요하지 않다면 `children ?: React.ReactNode` 라고 입력하여 이 Props를 생략해도 된다는 것을 명시해야함
> **_예제 👇_**


``` typescript
import React from 'react';
import {View, Text, StyleSheet, Image} from 'react-native';

interface Props {
    name: string;
    isActive?: boolean;
    image?: string;
    children: React.ReactNode;
}

function Profile({
    name,
    isActive,
    image='이미지 주소 넣기',
    children,
}: Props) {
    return (
        <View style={isActive && styles.activeStyle}>
            <Image source={{uri: image}} />
            <Text>{name}</Text>
            <View>{children}</View>
        </View>
    );
}

const styles = StyleSheet.create({
    activeStyle: {
        backgroundColor: 'yellow',
    },
});

export default Profile;
```

- App.tsx 에서 사용 👇


``` typescript
import React from 'react';
import {Text} from 'react-native';
import Profile from './profile/';

function App() {
    return (
        <Profile name="NarviS2">
            <Text>Hello World</Text>
        </Profile>
    );
}

export default App;
```

## 🚩 TypeScript 에서 useState 사용하기
---
- `Generic`을 사용하여 상태의 타입을 지정해줄 수 있습니다.
> **_✅ 참고_** : `useState`에 넣은 인자의 타입을 유추할 수 없거나 `여러 타입`을 지닌 경우에는 `Generic`을 꼭 설정해줘야 합니다.  
> **_예제 👇_**


``` typescript
import React, {useState} from 'react';
import {Button, Text, TextInput, View} from 'react-native';

function MessageForm() {
    // 해당 제네릭은 생략가능
    const [text, setText] = useState<String>('');
    // 여러 타입을 지닌 경우
    const [lastMessage, setLastMessage] = useState<{
        message: string;
        date: Date;
    } | null>(null);

    const onPress = () => {
        setLastMessage({
            message: text,
            date: new Date(),
        });
        setText('');
    };

    return (
        <View>
            <TextInput value={text} onChangeText={setText} />
            <Button title="PRESS ME" onPress={onPress} />
            {lastMessage && (
                <View>
                    <Text>
                        마지막 메시지: {lastMessage.message} (
                            {lastMessage.date.toLocaleString()}
                        )
                    </Text>
                </View>
            )}
        </View>
    );
}

export default MessageForm;
```

## 🚩 TypeScript 에서 useRef 사용하기
---
- `useRef`에 담을 값의 타입을 `Generic`으로 설정할 수 있습니다.


``` typescript
import React, {useState, useRef, useEffect} from 'react';
import {Button, Text, TextInput, View} from 'react-native';

function MessageForm() {
    // 해당 제네릭은 생략가능
    const [text, setText] = useState<String>('');
    // 여러 타입을 지닌 경우
    const [lastMessage, setLastMessage] = useState<{
        id: number;
        message: string;
        date: Date;
    } | null>(null);

    // useRef 사용
    const nextId = useRef<Number>(1);
    // 처음 렌더링될 때는 null이고 한번 렌더링된 뒤에는 TextInput의 인스턴스가 담깁니다.
    const inputRef = useRef<TextInput | null>(null);

    const onPress = () => {
        setLastMessage({
            id: nextId.current,
            message: text,
            date: new Date(),
        });
        setText('');
        nextId.current += 1;
    };

    // useEffect를 통해 이 컴포넌트가 보여지는 순간 TextInput에 포커스가 잡히도록 설정
    useEffect(() => {
        if (!inputRef.current) {
            retrun;
        }

        inputRef.current.focus();
    }, []);

    return (
        <View>
            <TextInput value={text} onChangeText={setText} ref={inputRef}/>
            <Button title="PRESS ME" onPress={onPress} />
            {lastMessage && (
                <View>
                    <Text>
                        마지막 메시지: {lastMessage.message} (
                            {lastMessage.date.toLocaleString()}
                        )
                    </Text>
                </View>
            )}
        </View>
    );
}

export default MessageForm;
```

## 🚩 TypeScript 에서 useReducer 사용하기
---
- `useReducer`를 사용할 때는 리듀서에서 관리할 `state`와 `action`타입을 선언해야 합니다.
- `state` 선언 (`interface` 사용)
> **_예제 👇_**
``` typescript
interface CounterState {
    value: number;
}
const initialState: CounterState = {
    value: 1,
};
```
- `type` 선언 (`Type Alias` 사용)
> **_예제 👇_**
``` typescript
type CounterAction = {type: 'increment'} | {type: 'decrement'; by: number};
```
- 만약 `action`의 갯수가 많아질 경우 (여러줄로 작성)
> **_예제 👇_**
``` typescript
type CounterAction = 
  | {type: 'increment'}
  | {type: 'decrement'; by: number}
  | {type: 'random'}
  | {type: 'nothing'};
```
> 또는 각 `action` 마다 `interface` 타입을 선언한 뒤 `Type Alias`로 합칠 수도 있음
``` typescript
interface IncrementAction {
    type: 'increment';
}
interface DecrementAction {
    type: 'decrement';
    by: number;
}
type CounterAction = IncrementAction | DecrementAction;
```
- `reducer()` 함수 선언
> `state`와 `action`을 파라미터로 받아와 그다음 상태를 반환하는 함수입니다.  
> **_예제 👇_**
``` typescript
function reducer(state: CounterState, action: CounterAction) {
    switch (action.type) {
        case 'increment':
            return {
                value: state.value + 1
            };
        case 'decrement':
            return {
                value: state.value - action.by
            };
        default:
            throw new Error('Unhandled action type');
    }
}
```
- 사용 및 전체 코드
``` typescript
interface CounterState {
    value: number;
}
const initialState: CounterState = {
    value: 1,
};
type CounterAction = {type: 'increment'} | {type: 'decrement'; by: number};
function reducer(state: CounterState, action: CounterAction) {
    switch (action.type) {
        case 'increment':
            return {
                value: state.value + 1
            };
        case 'decrement':
            return {
                value: state.value - action.by
            };
        default:
            throw new Error('Unhandled action type');
    }
}
function Counter() {
    const [state, dispatch] = useReducer(reducer, initialState);
    return (
        <View>
            <Text>{state.value}</Text>
            <Button
                title:"+1"
                onPress={() => dispatch({type: 'increment'})}
            />
            <Button
                title:"-1"
                onPress={() => dispatch({type: 'decrement'})}
            />
        </View>
    );
}
export default Counter;
```

## 마치며
---
이번 포스팅에서는 `React Component`에서 `TypeScript`를 작성하는법과 `TypeScript`로 `Hook`을 작성하는 법에 대하여 알아보았습니다.  
다음 시간에는 `TypeScript`로 `Context API`를 사용하는 법에 대하여 알아보도록 하겠습니다.