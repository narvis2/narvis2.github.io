---
title: Typescript 로 React-Navigations 사용하기
author: Narvis2
date: 2022-08-22 16:19:00 +0900
categories: [React-Native, TypeScript]
tags: [typescript, react-navigation, react-native]
---

안녕하세요. NarviS2 입니다.  
이번시간에는 `TypeScript`로 `React-Navigations`을 사용하는 방법에 대하여 알아보도록 하겠습니다.  
타입 설정을 잘 하면 새로운 화면을 띄우는 과정에서 화면의 이름을 **자동 완성** 하고, `route 파라미터`의 타입을 검증할 수 있습니다.  
또한 화면 Component에서는 해당 화면에서 받아올 `파라미터`들의 타입을 확인할 수 있습니다.

## 🚩 라이브러리 설치
---
- `TypeScript`가 공식적으로 지원되기 때문에 별도로 해야할 작업은 없습니다.
>**_명령어 👇_**
``` terminal
$ yarn add @react-navigation/native react-native-screens react-native-safe-area-context @react-navigation/native-stack @react-navigation/bottom-tabs
```

## 🚩 NavigationContainer 적용
---
- `App.tsx` 에서 `NavigationContainer`를 설정합니다. 👇


``` typescript
import React from 'react';
import {NavigationContainer} from '@react-navigation/native';

function App() {
    return <NavigationContainer>{/* TODO: 화면 추가 */}</NavigationContainer>;
}

export default App;
```

## 🚩 Native Stack Navigation 사용하기 
---
- `RootStack.tsx` 에서 `native-stack`을 사용합니다. 👇


``` typescript
import React from 'react';
import {Button, Text, View} from 'react-native'
import {createNativeStackNavigator} from '@react-navigation/native-stack';
import {useNavigation} from '@react-navigation/native';

const Stack = createNativeStackNavigator();

function HomeScreen() {
    const navigation = useNavigation();

    const onPress = () => {
        navigation.navigate('Detail');
    };

    return (
        <View>
            <Text>Home</Text>
            <Button title="Open Detail" onPress={onPress} />
        </View>
    );
}

function DetailScreen() {
    return (
        <View>
            <Text>Detail</Text>
        </View>
    );
}

function RootStack() {
    return (
        <Stack.Navigator>
            <Stack.Screen component={HomeScreen} name="Home" />
            <Stack.Screen component={DetailScreen} name="Detail" />
        </Stack.Navigator>
    );
}

export default RootStack;
```
> `TypeScript`를 사용할 경우 이 코드에서 `navigation.native('Detail')`을 할 때 `Detail` 부분에 빨간 줄이 그어지게 됩니다.  
> **_주의 ❗️_** `TypeScript`에서 `native-stack navigation`을 사용할 때는 어떤 화면에 어떤 파라미터가 필요한지 `StackParamList`타입을 정의해줘야 합니다.
> 이 타입을 사용하면 다른 화면을 띄울 때 화면의 이름과 `route 파라미터`를 검증할 수 있습니다.

- `Native Stack Navigation`에서 사용할 화면 타입 선언하기
> `createNativeStackNavigator` 의 `Generic`으로 사용합니다.  
> **_주의 ❗️_** RootStackParamList 타입을 선언할 때는 `interface`가 아닌 `type`을 사용하여 선언해야 합니다.
``` typescript
type RootStackParamList = {
    Home: undefined;
    Detail: {
        id: number;
    };
};
const Stack = createStackNavigator<RootStackParamList>();
```

## 🚩 useNavigation 사용하기 
---
- `useNavigation`을 사용할 때는 `NavigationPop`을 선언해야 합니다.
- 선언한 타입을 `useNavigation` 의 `Generic`으로 지정해줘야 합니다.
- `native-stack`의 경우에는 `NativeStackNavigationProp`을 불러와서 선언할 수 있습니다.
> **_주의 ❗️_** `TypeScript`를 사용하는 환경에서는 `NativeStackNavigationProp`을 사용하여 선언된 `NavigationProp`이 `useNavigation`의 `Generic`으로 설정되어 있지 않으면 `navigation.push`,`navigation.pop` 등의 함수를 사용할 수 없습니다.


``` typescript
import React from 'react';
import {Button, Text, View} from 'react-native';
import {
    createNativeStackNavigator,
    NativeStackNavigationProp,
} from '@react-navigation/native-stack';
import {useNavigation} from '@react-navigation/native';

type RootStackParamList = {
    Home: undefined;
    Detail: {
        id: number;
    };
};

// export를 붙여 다른 파일에서 이를 불러와 사용할 수 있게 설정(추후 내비게이션을 감싸는 방법을 다룰 때 불러와서 사용)
export type RootStackNavigationProp = NativeStackNavigationProp<RootStackParamList>;

const Stack = createStackNavigator<RootStackParamList>();

function HomeScreen() {
    // Generic 을 사용하면 특정 화면으로 navigate할 때 route 파라미터의 타입을 검증할 수 있습니다.
    const navigation = useNavigation<RootStackNavigationProp>();

    const onPress = () => {
        navigation.navigate('Detail', {id: 1});
    };

    return (
        <View>
            <Text>Home</Text>
            <Button title="Open Detail" onPress={onPress} />
        </View>
    );
}
```

## 🚩 useRoute 사용하기
---
- 이 `Hook`을 사용할 때는 `RouteProp`을 사용하여 선언한 타입을 `Generic`으로 설정해야 합니다.
> **_✅ 설명_** `RouteProp` 👇
>> - 첫 번째 `Generic`: 현재 사용 중인 내비게이션의 `route 파라미터` 타입을 위해 선언한 `RooteStackParamList`를 넣습니다.
>> - 두 번째 `Generic`: 화면 이름을 넣습니다.


``` typescript
import React from 'react';
import {Button, Text, View} from 'react-native';
import {
    createNativeStackNavigator,
    NativeStackNavigationProp,
} from '@react-navigation/native-stack';
import {useNavigation, RouteProp, useRoute} from '@react-navigation/native';

type RootStackParamList = {
    Home: undefined;
    Detail: {
        id: number;
    };
};

// export를 붙여 다른 파일에서 이를 불러와 사용할 수 있게 설정(추후 내비게이션을 감싸는 방법을 다룰 때 불러와서 사용)
export type RootStackNavigationProp = NativeStackNavigationProp<RootStackParamList>;

const Stack = createStackNavigator<RootStackParamList>();

type DetailScreenRouteProp = RouteProp<RootStackParamList, 'Detail'>();

function HomeScreen() {
    // Generic 을 사용하면 특정 화면으로 navigate할 때 route 파라미터의 타입을 검증할 수 있습니다.
    const navigation = useNavigation<RootStackNavigationProp>();

    const onPress = () => {
        navigation.navigate('Detail', {id: 1});
    };

    return (
        <View>
            <Text>Home</Text>
            <Button title="Open Detail" onPress={onPress} />
        </View>
    );
}

function DetailScreen() {
    const {params} = useRoute<DetailScreenRouteProp>();

    return (
        <View>
            <Text>Detail {params.id}</Text>
        </View>
    );
}

function RootStack() {
    return (
        <Stack.Navigator>
            <Stack.Screen component={HomeScreen} name="Home" />
            <Stack.Screen component={DetailScreen} name="Detail" />
        </Stack.Navigator>
    );
}

export default RootStack;
```

- `App.tsx`에서 사용


``` typescript
import React from 'react';
import {NavigationContainer} from '@react-navigation/native';
import Rootstack from './screens/RootStack';

function App() {
    return (
        <NavigationContainer>
            <RootStack />
        </NavigationContainer>
    );
}

export default App;
```

## 🚩 Navigation 감싸기
---
- 여러개의 `navigation`을 감쌀 때는 `CompositeNavigationProp`을 사용하여 `NavigationPop`을 합칩니다.
- 하단 탭 네이게이션의 타입을 지정할 때 예제 👇
> **_✅ 설명_**
>> - `Native Stack Navigation` 없이 하단 탭 내비게이션만 단독으로 사용한다면 `type MainTabNavigationProp = BottomTabNavigationProp<MainTabParamList>;`으로 충분합니다.
>> - 하단 탭 내비게이션에서 그 상위에 있는 `RootStack`의 `DetailScreen`에 접근하려면 `NavigationProp`들을 합쳐줘야 합니다.


``` typescript
import React from 'react';
import {
    BottomTabNavigationProp,
    createBottomTabNavigator,
} from '@react-navigation/bottom-tabs';
import {Button, Text, View} from 'react-native';
import {
    CompositeNavigationProp,
    NavigatorScreenParams,
    useNavigation
} from '@react-navigation/native';
import {RootStackNavigationProp} from './RootStack/';

type MainTabParamList = {
    Home: undefined;
    Account: undefined;
};

export type MainTabNavigationProp = CompositeNavigationProp<
    RootStackNavigationProp, 
    BottomTabNavigationProp<MainTabParamList>
>;

// 추후 RootStack 내부 화면에서 navigation.navigate('MainTap', {screen: 'Account'})가 가능하게 해줍니다.
export type MainTapNavigationScreenParams = NavigatorScreenParams<MainTabParamList>;

const Tab = createBottomTabNavigator<MainTapParamList>();

function HomeScreen() {
    const navigation = useNavigation<MainTabNavigationProp>();

    const onPress = () => {
        navigation.navigate('Detail', {id: 1})
    };

    return (
        <View>
            <Text>Home</Text>
            <Button title="Open Detail" onPress={onPress} />
        </View>
    );
}

function AccountScreen() {
    return (
        <View>
            <Text>Account</Text>
        </View>
    );
}

function MainTab() {
    return (
        <Tab.Navigator>
            <Tab.Screen name="Home" component={HomeScreen} />
            <Tab.Screen name="Account" component={AccountScreen} />
        </Tab.Navigator>
    );
}

export default MainTab;
```

## 🚩 RootStack 에서 HomeSceen 지우고 MainTab으로 대체
---
``` typescript
import React from 'react';
import {Text, View} from 'react-native';
import {
    createNativeStackNavigator,
    NativeStackNavigationProp,
} from '@react-navigation/native-stack';
import {RouteProp, useRoute} from '@react-navigation/native';
import MainTab, {MainTabNavigationScreenParams} from './MainTab';

type RootStackParamList = {
    MainTab: MainTabNavigationScreenParams;
    Detail: {
        id: number;
    };
};

export type RootStackNavigationProp = 
    NativeStackNavigationProp<RootStackParamList>;

const Stack = createNativeStackNavigator<RootStackParamList>();

type DetailScreenRouteProp = RouteProp<RootStackParamList, 'Detail'>;

function DetailScreen() {
    const {params} = useRoute<DetailScreenRouteProp>();

    return (
        <View>
            <Text>Detail {params.id}</Text>
        </View>
    );
}

function RootStack() {
    return (
        <Stack.Navigator>
            <Stack.Screen
                component={MainTab}
                name="MainTab"
                options={{
                    headerShown: false,
                }}
            />
            <Stack.Screen component={DetailScreen} name="Detail" />
        </Stack.Navigator>
    );
}

export default RootStack;
```

## 🚩 마치며
--- 
이번 포스팅에서는 `Typescript` 로 `React-Navigations`를 사용하는 방법에 대하여 알아보았습니다.  
해당 내용은 충분히 헷갈릴 수 있으므로 여러번 반복해서 보시는걸 추천합니다.  