---
title: Typescript 에서 Context API 사용하기
author: Narvis2
date: 2022-08-22 15:19:00 +0900
categories: [React-Native, TypeScript]
tags: [typescript, context-api, react-native]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 지난 포스팅에 이어 `TypeScript`에서 `Context API`를 사용하는 방법에 대하여 알아보겠습니다.  
지난 포스팅 👉 [Typescript 에서 Hook 사용하기](https://narvis2.github.io/posts/React-Native-Using-Hooks-In-Typescript/)

## 🚩 TypeScript 로 Context API 사용하기
- `createContext` 함수를 호출할 때 `Generic`을 설정해줘야 합니다.
- `AuthContext.Provider`를 사용하지 않았을 경우에는 기본 값이 `null` 이므로 `AuthContextValue | null`을 `createContext`의 `Generic`으로 설정해야 합니다.
> **_예제 👇_**
``` typescript
interface User {
    id: number;
    username: string;
}
interface AuthContextValue {
    user: User | null;
    setUser(user: User): void;
}
const AuthContext = createContext<AuthContextValue | null>(null);
```
- `Context` 전용 `Provider` Component를 따로 만들 때 해당 Component 에서 `children Props`를 사용하기 때문에 해당 타입도 파라미터 부분에서 지정해줘야 합니다.
> **_예제 👇_**  
``` typescript
export function AuthContextProvider({children}: {children: React.ReactNode}) {
    // TODO: AuthContext.Provider 렌더링
}
```
- 전용 `Hook`을 만들 때 유효성 검사를 한 후 에러를 `throw`해줘야 함수의 반환값 타입이 `AuthContextValue`가 됩니다.
> 나중에 해당 `Context`를 더 편하게 사용할 수 있습니다. 그렇지 않으면 `Hook`을 사용할 때 따로 유효성 검사를 해줘야 합니다.  
> **_예제 👇_**
``` typescript
export function useAuth() {
    const auth = useContext(AuthContext);
    if (!auth) {
        throw new Error('AuthContextProvider is not used');
    }
    return auth;
}
```
> auth의 유효성을 검사해줘야, useAuch의 반환값 타입이 AuthContextValue로 되고, throw 하여 null 타입을 제외합니다.


- 전체 코드 `AuthContext.tsx` 작성


``` typescript
import React, {createContext, useContext, useState} from 'react';

interface User {
    id: number;
    username: string;
}

interface AuthContextValue {
    user: User | null;
    setUser(user: User): void;
}

const AuthContext = createContext<AuthContextValue | null>(null);

export function AuthContextProvider({children}: {children: React.ReactNode}) {
    const [user, setUser] = userState<User | null>(null);

    return (
        <AuthContext.Provider value={{user, setUser}}>
            {children}
        </AuthContext.Provider>
    );
}

export function useAuth() {
    const auth = useContext(AuthContext);
    // auth의 유효성을 검사해줘야 useAuth의 반환값 타입이 AuthContextValue로 됨
    if (!auth) {
        // throw 하여 null 타입 제외
        throw new Error('AuthContextProvider is not used');
    }

    return auth;
}
```

## 🚩 마치며
--- 
이번 포스팅에서는 Typescript 에서 Context API 사용하는 방법에 대하여 알아보았습니다.  
다음 포스팅에서는 `Typescript` 로 `React-Navigations`를 사용하는 방법에 대하여 알아보도록 하겠습니다.  
[Typescript로 React-Navigations를 사용하는 방법](https://narvis2.github.io/posts/React-Native-Using-React-Navigation-With-Typescript/)