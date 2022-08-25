---
title: React-Native Context API
author: Narvis2
date: 2022-08-22 16:03:00 +0900
categories: [React-Native, Context]
tags: [react-native, context-api]
---

안녕하세요. Narvis2입니다.  
이번 시간에는 React-Native `Context API` 에 관하여 알아볼까 합니다.  
`Context API`는 `react`에 내장된 기능으로 Props를 사용하지 않아도 특정 값이 필요한 Component끼리 쉽게 값을 공유할 수 있게 해줍니다.  
**주로 프로젝트에서 전역 상태를 관리할 때 많이 사용합니다.**

## 🚩 Context API
---
- Component의 상태를 공유할 때 주로 사용합니다. (서로 다른 화면에서 상태를 공유)
> Android로 치면 activityViewModels()를 생각하시면 됩니다.
- 주로 프로젝트 전역 상태를 관리할 때 많이 사용합니다.
- 새로운 `Context`를 만들 때는 `createContext`함수를 사용합니다. 👇

```javascript
import {createContext} from 'react';

const LogContext = createContext('안녕하세요');

export default LogContext;
```


- Context를 만들면 `LogContext.Provider`라는 Component와 `LogContext.Consumer`라는 Component가 만들어 집니다.
> - `Provider`는 Context안에 있는 값을 사용할 Component들을 감싸주는 용도로 사용합니다.
> - `Provider`의 `value`라는 Props 를 통해 여러 Component에서 공유할 수 있는 값을 지정합니다.
> - `Provider` Component를 사용하면 이 Component 내부에 선언된 모든 Component에서 Context안의 값을 사용할 수 있습니다.


```javascript
import React from 'react';
import {NavigationContainer} from '@react-navigation/native';
import RootStack from './sreens/RootStack';
import LogContext from './contexts/LogContext';

function App() {
    // 이렇게하면 RootStack에 있는 모든 Component에서 해당 Context를 통해 value값을 사용할 수 있습니다.
    return (
        <NavigationContainer>
            <LogContext.Provider value="안녕하세요">
                <RootStack />
            </LogContext.Provider>
        </NavigationContainer>
    );
}

export default App;
```


- 해당 Context 값 사용 👇


``` javascript
import React from 'react';
import {View, Text} from 'react-native';
import LogContext from '../contexts/LogContext';

function FeedsScreen() {
    return (
        <View>
            <LogContext.Consumer>
                {(value) => <Text>{value}</Text>}
            </LogContext.Consumer>
        </View>
    );
}

export default FeedsScreen;
```

## 🚩 useContext - Hook 함수
---
- `Render Props` 와 `LogContext.Consumer` 대체 
- Context의 값을 훨씬 간결하게 사용 가능합니다. 👇


``` javascript
import React, {useContext} from 'react';
import {View, Text} from 'react-native';
import LogContext from '../contexts/LogContext';

function FeedsScreen() {
    const value = useContext(LogContext);

    return (
        <View>
            <Text>{value}</Text>
        </View>
    );
}

export default FeedsScreen;
```


- Component에서 `JSX`를 반환하기 전에 값을 조회할 수 있기 때문에 Component 로직 작성이 더욱 간편합니다.

## 🚩 Context에서 유동적인 값 다루기
---
- `Provider`를 사용하는 Component에서 Context의 상태를 관리하는 것보다는 `Provider` 전용 Component를 따로 만드는 것이 **유지보수성**이 더 높습니다. 특히 Context에서 다루는 로직이 복잡할 때는 전용 Component를 만드는 것이 좋습니다.
- 별도의 Context Provider Component 만들기 예제 코드 👇


``` javascript
import React from 'react';
import {createContext, useState} from 'react';

const LogContext = createContext();

export function LogContextProvider({children}) {
    const [text, setText] = useState('');
    return (
        <LogContext.Provider value={{text, setText}}>
            {children}
        </LogContext.Provider>
    );
}

export default LogContext;
```


- Context Provider Component 사용 👇


``` javascript
import React from 'react';
import {NavigationContainer} from '@react-navigation/native';
import RootStack from './screens/RootStack';
import {LogContextProvider} from './contexts/LogContext';

function App() {
    return (
        <NavigationContainer>
            <LogContextProvider>
                <RootStack />
            </LogContextProvider>
        </NavigationContainer>
    );
}

export default App;
```

- useContext를 사용하여 Context Provider로 제공된 Data 사용 👇


``` javascript
import React, {useContext} from 'react';
import {View, TextInput} from 'react-native';
import LogContext from '../contexts/LogContext';

function FeedsScreen() {
    const {text, setText} = useContext(LogContext);

    return (
        <View>
            <TextInput>
                value={text}
                onChangeText={setText}
                placeHolder="텍스트를 입력하세요."
            </TextInput>
        </View>
    );
}

export default FeedsScreen;
```

## 🚩 마치며
이번 포스팅에서는 Context API에 대하여 알아보았습니다.  
Context API는 서로 다른 화면간에 데이터를 공유하기 위해 사용하며 Android 의 Fragment간 데이터 공유에 사용하는 activityViewModels()를 생각하면 쉽습니다.  
