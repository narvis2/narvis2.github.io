---
title: TypeScript
author: Narvis2
date: 2022-08-22 13:37:00 +0900
categories: [React-Native, TypeScript]
tags: [typescript]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 `TypeScript`에 대하여 알아보도록 하겠습니다.  
`JavaScript`는 동적 타입 언어 입니다. 즉, 특정 변수를 선언했을 때 다른 타입으로 대입할 수 있습니다.  
`JavaScript`의 변수의 타입은 `Runtime`에서 결정됩니다. 따라서 프로젝트의 규모가 커질수록 실수할 가능성도 높아지고 타입 추론이 일부만 되기 때문에 개발이 불편할 수 있습니다. 또한 오류 발생이 `Runtime`에서 발생하므로 오류를 핸들링하기가 불편합니다.  
이런 문제를 해결하기 위해 `TypeScript`를 사용하여 `JavaScript`에 정적 타입 시스템을 적용합니다.

## 🚩 TypeScript
---
- `MicroSoft`에서 만든 오픈 소스 프로그래밍 언어입니다.
- `JavaScript`에 `정적 타입`을 사용할 수 있습니다.
- `TypeScript`로 작성된 코드는 `tsc(TypeScript Compiler)`를 통해 `JavaScript`코드로 변환됩니다.
- `TypeScript`의 확장자는 `ts` 또는 `tsx` 입니다.
> **_주의❗️_** __React Component를 작성한다면 확장자를 tsx로 해야 JSX가 지원됩니다__. JSX를 사용하지 않는다면 확장자를 ts로 저장하면 됩니다.
- `TypeScript`가 적용된 프로젝트에서는 선언된 변수를 사용하는 곳이 없으면 Error를 발생시키는 규칙이 있습니다.
> **_주의❗️_** `.eslintrc.js` 파일을 열어서 밑의 코드를 입력합니다.
``` javascript
module.exports = {
    root: true,
    extends: '@react-native-community',
    rules: {
        '@typescript-eslint/no-unused-vars': 0,
    }
}
```


## 🚩 TypeScript React Native 에서 사용
---
``` terminal
$ npx react-native init 프로젝트이름 --template react-native-template-typescript
```

## 🚩 TypeScript 기초 문법
---
### 1. 기본 타입
- 기본 타입들은 타입을 명시해주지 않아도 자동으로 `타입추론`됩니다.
> 변수가 하나의 타입만 지니고 있다면 타입 선언을 생략해도 됩니다.
- string : 문자열
- number : 숫자
- boolean : true / false
- any : 모든 타입
- undefined : 변수는 존재하나, **어떠한 값으로도 할당되지 않아 자료형이 정해지지(undefined) 않은 상태**
- | : or
> **예제 코드 👇**  


``` typescript
let message: string = 'Hello World';
let value: number = 1;

// 문자열 or Null 타입
let nullableString: string | null = null;
nullableString = 'Hi';

// undefined or 숫자 타입 
let undefinedDrNumber: undefined | number;
undefinedDrNumber = 'Hi';

// 숫자 or 문자열 or Null 타입
let numberOrStringOrNull: number | string | null = null;
numberOrStringOrNull = 1;
numberOrStringOrNull = 'Bye';
numberOrStringOrNull = null;

let isCompleted: boolean = true;
isCompleted = false;

// 모든 타입의 값을 대입할 수 있음
let anyValue: any = null;
anyValue = undefined;
anyValue = 1;
anyValue = 'hello world';
```

### 2. 함수 타입
- 여러 반환값 타입을 지닌 함수 👇
> **_✅참고_** : 함수에서 무엇을 반환하느냐에 따라 반환값의 타입이 결정되도록 함수 반환값을 따로 설정안함 
``` typescript
function sum(a: number, b: number) {
    return a + b;
}
```
- 명시적으로 함수 반환값 선언 👇
> **_✅참고_** : 함수의 반환값이 지정한 타입과 다른 값을 반환한다면 오류가 발생합니다.
``` typescript
function sum(a: number, b: number): number {
    retrun a + b;
}
```

### 3. 옵서녈 파라미터
- 생락해도 되는 파라미터를 의미합니다. 👇
> **_✅참고_** : 밑의 코드에서 isDouble 은 boolean | undefined 로 설정됩니다. 
``` typescript
function process(a: number, b: number, isDouble?: boolean) {
    const sum = a + b;
    return isDouble ? sum * 2 : sum;
}
```

### 4. interface
- `TypeScript`에서 객체나 클래스를 위한 Type을 정할 때 사용합니다.
- 선언된 객체 타입은 변수의 타입이나 파라미터의 타입으로 사용할 수 있습니다.
> **_예제👇_**
``` typescript
interface Profile {
    id: number;
    username: string;
    displayName: string;
}
function printUsername(profile: Profile) {
    console.log(profile.username);
}
const profile: Profile = {
    id: 1,
    username: 'narvis2',
    displayName: 'Youngjun Choi',
};
printUsername(profile)
```
- 다른 `interface`에서 참조
> **_예제👇_**
``` typescript
interface Profile {
    id: number;
    username: string;
    displayName: string;
}
interface RelationShip {
    from: Profile;
    to: Profile;
}
const relationship: RelationShip = {
    from: {
        id: 1,
        username: 'narvis2',
        displayName: 'Youngjun Choi',
    },
    to: {
        id: 2,
        username: 'astatrte',
        displayname: 'Jun'
    },
};
```

### 5. interface 상속하기
- 기존 interface 타입에 **필드를 더 추가할 때** 는 상속을 통해 처리할 수 있습니다. 👇
> **_예제👇_**
``` typescript
interface Profile {
    id: number;
    username: string;
    displayName: string;
}
interface Account extends Profile {
    email: string;
    password: string;
}
const account: Account = {
    id: 1,
    username: 'narvis2',
    displayName: 'Youngjun Choi',
    email: 'narvis2@naver.com',
    password: '123123',
};
```

### 6. 옵셔널 속성
- `interface`에서 옵셔널 속성을 다룰 때는 함수의 옵셔널 파라미터와 비슷하게 `?`를 사용합니다.
> **_예제👇_**
``` typescript
interface Profile {
    id: number;
    username: string;
    displayname: string;
    photoURL?: string;
}
const profile: Profile = {
    id: 1,
    usename: 'narvis2',
    displayName: 'Youngjun Choi',
}
const profileWithPhoto: Profile = {
    id: 1,
    usename: 'narvis2',
    displayName: 'Youngjun Choi',
    photoURL: 'photo.png',
}
```

### 7. 클래스에서 interface를 implement 하기
- `interface`에서 구현해야 하는 메서드들의 타입을 미리 선언하고, 이에 맞춰 클래스에서 구현하는 예제 👇
``` typescript
interface Shape {
    getArea(): number;
    getPerimeter(): number;
}
class Circle implements Shape {
    // Class의 기본 생성자 
    constructor(private radius: number) {}

    getArea() {
        return Math.PI * Match.pow(this.radius, 2);
    }
    getPerimeter() {
        return 2 * Math.PI * this.radius;
    }
}
class Rectangle implements Shape {
    constructor(private width: number, private height: number) {}

    getArea() {
        return this.width + this.height;
    }
    getPerimeter() {
        return 2 * (this.width + this.height);
    }
}
```
> **_✅참고_** : constructor 부분에 private radius: number 는 다음 형식과 같음 👇
``` typescript
class Circle implements Shape {
    radius: number;
    constructor(radius: number) {
        this.radius = radius;
    }
}
```

### 8. 배열 타입
- `TypeScript`에서 **배열 타입** 을 다룰 때는 타입 뒤에 `[]`를 붙여줍니다.
> **_예제👇_**
``` typescript
const numbers: number[] = [1, 2, 3, 4, 5];
const texts: string[] = ['hello', 'world'];
// 객체를 배열로
interface Person {
    name: string;
}
const people: Person[] = [{name: 'narvis2'}, {name: 'Choi Young Jun'}];
```

### 9. Type Alias
- 타입에 별칭을 붙여주는 기능
> **_예제👇_**
``` typescript
type Persion = {
    name: String;
};
const person: Person = {
    name: 'Choi Young Jun',
};
// 객체 배열 타입
type People = Person[];
const people: People = [{name: 'Choi Young Jun'}];
// Android 로 치면 Enum Class
type Color = 'red' | 'orange' | 'yellow';
const color: Color = 'red';
```
- `type`을 사용하여 만든 객체 타입에도 필드를 추가할 수 있습니다. `&` 문자를 사용
> **_예제👇_**
``` typescript
type Persion = {
    name: string;
};
// Persion 객체에 job 이라는 필드 추가
type Employee = Persion & {
    job: string;
};
// Employee 객체를 인스턴스화 하여 필드에 값 추가
const empolyee: Emplyee = {
    name: 'narvis2',
    job: 'Programmer',
};
```

### 10. Generic(제네릭)
- 함수, 객체, 클래스 타입에서 사전에 정하지 않은 다양한 타입을 다룰 때 사용
  
1. `함수(function)`에서 `Generic` 사용
- `Generic`을 사용하지 않는 경우 `any` 타입 사용
> **_주의❗️_** 해당 형식을 사용할 경우 result 에서 즉 함수 호출하는 쪽에서 type 추론이 되지 않습니다.  
> **_예제👇_**
``` typescript
function wrap(value: any) {
    return {value};
}
const result = wrap('Hello World');
```
- `Generic`을 사용하여 type 추론이 가능하게 설정
> **_예제👇_**
``` typescript
function wrap<T>(value: T) {
    return {value};
}
const result = wrap('Hello World');
```

2. `객체(interface)`에서 `Generic` 사용
- 객체 타입의 특정한 필드에 다양한 값이 올 수 있다고 가정
> **_예제👇_**
``` typescript
interface Item<T> {
    id: number;
    data: T;
}
interface Persion {
    name: string;
}
interface Place {
    location: string;
}
const personItem: Item<Person> = {
    id: 1,
    data: {
        name: "narvis2",
    },
};
const placeItem: Item<Place> = {
    id: 2,
    data: {
        location: "Korea",
    },
};
```

3. `Type Alias`에서 `Generic`사용
``` typescript
type Item<T> = {id: number; data: T}
```

4. `클래스(Class)`에서 `Generic`사용
- `Queue` 클래스를 예제로 사용해 보겠습니다. (선입 선출)
> **_✅참고_** : `Queue`에 대하여 잘 모르신다면 [자료 구조 Queue, Stack 정리](https://narvis2.github.io/posts/Data-Structure-Stack-Queue/) 포스팅을 참고해 주세요.  
> **_예제👇_**
``` typescript
class Queue<T> {
    list: T[] = [];
    get length() {
        return this.list.length;
    }
    enqueue(item: T) {
        this.list.push(item);
    }
    dequeue() {
        return this.list.shift();
    }
}
const queue = new Queue<number>();
queue.enqueue(0);
queue.enqueue(1);
const first = queue.dequeue(); // 0
const second = queue.dequeue(); // 1
```

## 마치며
---
이번 포스팅에서는 `TypeScript`에 대하여 알아보았습니다.  
`JavaScript`는 `동적 타입`이기 때문에 오류 발생이 `Runtime`에 발생하여 `TypeScript`를 사용하여 `정적 타입`으로 바꿔 오류 발생을 `Compile` 시점으로 바꿔 오류 핸들링에 더욱 이점을 보고자 `TypeScript`를 사용합니다.  