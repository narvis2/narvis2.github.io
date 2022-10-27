---
title: React-Native React-Query
author: Narvis2
date: 2022-08-25 16:19:00 +0900
categories: [React-Native, React-Query]
tags: [react-query, react-native, useQuery, useMutation]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 서버의 값을 client 에 가져오거나 캐싱, 값 업데이트, 에러 핸들링 등 비동기 과정을 더욱 편하게 하는데 사용하는 `react-query`에 대하여 알아보겠습니다.

## 🚩 React-Query

- 🍬 서버의 값을 client 에 가져오거나 Caching, value updating, error handling 등 **_비동기 과정을 더욱 편하게_** 하는데 사용됨
- 서버와 클라이언트를 분리합니다.

### 🍀 1. React-Query 장점

- 캐싱
  - `get`을 한 데이터에 대해 `update`를 하면 자동으로 `get`을 다시 수행한다.
  - Ex 👉 게시판의 글을 가져왔을 때 게시판의 글을 생성하면 게시판 글을 `get`하는 `api`를 자동으로 실행할 수 있음
  - 데이터가 오래되었다고 판단되면 다시 `get` (`invalidateQueries`)
  - 동일 데이터를 여러번 요청하면 한번만 요청한다. (Option 에 따라 중복 호출, 허용 시간 조절 가능)
- 무한 스크롤
  - `Infinite Queries`를 사용하여 pagination 처리 가능
- 비동기 과정을 선언적으로 관리가 가능하다.
- react 의 `Hook`과 사용하는 구조가 비슷하다.

### 🍀 2. useQuery

- 데이터를 `get` 하기 위한 `api` (post, update 는 `useMutation` 을 사용)
- 첫 번째 파라미터
  - `Unique Key`가 들어감, **_요청의 결과물이 특정 변수에 따라 달라진다면_** 꼭 `Unique key`에 포함해야 함
  - `Unique key`는 String 과 []을 받는다. 배열로 넘기면 0번 값은 String 값으로 다른 Component에서 부를 값이 들어가고 두 번째 값을 넣으면 Query 함수 내부에 파라미터로 해당 값이 전달됨
- 두 번째 파라미터
  - 비동기 함수(api 호출 함수)가 들어감 (`promise`가 들어가야 함)
  - 이 Component 가 랜더링될 때 해당 함수를 호출하고, 이에 대한 상태가 관리됨
- 첫 번째 파라미터로 설정한 `Unique Key`는 다른 Component 에서도 해당 `key`를 사용하면 호출 가능
- `Return` 값
  - Api 요청의 성공, 실패 여부 및 Api 결과 값을 포함한 객체
  - > `status` 👉 API의 요청 상태를 문자열로 나타냄
    - > `loading` 👉 아직 데이터를 받아오지 않고, 현재 데이터를 요청 중
    - > `error` 👉 오류 발생
    - > `success` 👉 데이터 요청 성공
    - > `idle` 👉 비활성화된 상태(따로 설정해 비활성화한 경우)
  - > `isLoading` 👉 status === 'loading' 과 같음
  - > `isError` 👉 status === 'error' 와 같음
  - > `isSuccess` 👉 staus === 'success' 와 같음
  - > `isIdle` 👉 staus === 'idle' 과 같음
  - > `error` 👉 오류가 발생했을 때 오류 정보를 가지고 있음
  - > `data` 👉 요청 성공한 데이터를 가리킴
  - > `isFetching` 👉 데이터가 요청 중일 때 true가 됨
    - > 데이터가 이미 존재하는 상태에서 재요청할 때 `isLoading` 은 false 이지만, `isFetching` 은 true 임
  - > `refetch` 👉 다시 요청을 시작하는 함수
- 비동기로 작동
  - 여러개의 비동기 `Query`가 있다면 `useQuery` 보다는 `useQueries` 를 사용하는 것이 좋음
- `enabled` 를 사용하면 `useQuery` 를 동기적으로 사용 가능
  - `useQuery`의 3번째 인자로 `Option` 값이 들어가는데 그 `Option`의 `enabled`에 값을 넣으면 `enabled` 값이 `true` 일때 `useQuery`를 실행함
- `useQuery Option`
  - `enable` 👉 boolean 타입의 값을 설정, 이 값이 false 이면 Component 가 마운트될 때 자동으로 요청하지 않음. **_refetch 함수로만 요청이 시작됨_**
  - `retry` 👉 boolean or number or (failureCount: number, error: TError) => boolean 타입의 값을 설정
    - 요청이 실패했을 때 재요청할지 설정할 수 있음
    - > 이 값을 `true`로 했을때 👉 실패했을 때 성공할 때까지 계속 반복 요청
    - > 이 값을 `false`로 했을때 👉 실패했을 떄 재요청하지 않음
    - > 이 값을 `3` 으로 하면 👉 3번까지만 재요청
    - > 이 값을 `함수 타입`으로 설정하면 👉 실패 횟수와 오류 타입에 따라 재요청할지 함수 내에서 결정할 수 있음
  - `retryDelay` 👉 number or (retryAttempt: number, error: TError) => number 타입 값을 설정
    - 시간 단위는 ms(밀리세컨드 0.001초)임
    - > 기본값 👉 (retryAttempt) => Math.min(1000 \* 2 \*\* failureCount, 30000)
    - > 실패 횟수 n 에 따라 2의 n 제곱 초만큼 기다렸다가 재요청하고 최대 30초까지 기다림
  - `staleTime` 👉 데이터의 유효 시간을 ms 단위로 설정, 기본값 0 (데이터를 조회한 순간 데이터는 바로 유효하지 않게 됨)
  - `cacheTime` 👉 데이터의 캐시 시간을 ms 단위로 설정, 기본값은 5분, 캐시 시간은 Hook을 사용하는 Component 가 언마운트되고 나서 해당 데이터를 얼마나 유지할지 결정함
    > **_🍀 참고 🍀_** `staleTime` 과 `cacheTime` 의 차이 👇
    >
    > - `useQuery` 를 사용할 때 `staleTime` 옵션은 기본값이 0 임. 즉, **_데이터를 조회한 순간 데이터는 바로 유효하지 않게됨_**. 데이터가 유효하지 않다면 기회가 주어졌을 때 다시 요청하여 데이터를 최신화 해야함
    > - 재요청 기회가 주어지는 시점 : 똑같은 `Cache key` 를 사용하는 `useQuery` 를 사용중인 Component 가 마운트될 때
    > - `cacheTime`은 `useQuery Hook`을 사용한 Component가 언마운트되고 나서 해당 데이터를 얼마 동안 유지할지에 대한 설정, 기본값은 5
    > - 만약 `useQuery`를 사용한 Component 가 언마운트되고 나서 5분안에 다시 마운트된다면 isLoading 값이 true로 되지 않고, 처음 렌더링하는 시점부터 data 값이 이전에 불러온 데이터로 채워져 있게 된다. 그리고 `staleTime`에 따라 해당 데이터가 유효하다면 재요청하지 않고, 유효하지 않다면 재요청한다.
  - `refetchInterval` 👉 false or number 타입값을 설정
    - 이 설정으로 n초마다 데이터를 새로고침하도록 설정할 수 있음
    - 시간 단위는 ms 임
  - `refetchOnmount` 👉 boolean or 'always' 타입의 값을 설정
    - 이 설정으로 Component가 마운트될 때 재요청하는 방식을 설정할 수 있음
    - > 기본 값: true
    - > true일 때는 데이터가 유효하지 않을 때 재요청함
    - > false일 때는 Component가 다시 마운트되어도 재요청하지 않음
    - > 'always'일 때는 데이터의 유효 여부와 관계없이 무조건 재요청함
  - `onSuccess` 👉 (data: Data) => void 타입의 함수를 설정, 데이터 요청이 성공하고 나서 특정 함수를 호출하고 싶을 때 사용
  - `onError` 👉 (error: Error) => void 타입의 함수를 설정, 데이터 요청이 실패하고 나서 특정 함수를 호출하고 싶을 때 사용
  - `onSettled` 👉 (data?: Data, error?: Error) => void 타입의 함수를 설정, 데이터 요청의 성공 여부와 관계없이 요청이 끝나면 특정 함수를 호출하도록 설정
  - `initialData` 👉 Data | () => Data 타입의 값을 설정, `Hook` 에서 사용할 데이터의 초깃값을 지정하고 싶을 때 사용
  - `refetchOnWindowFocus` 👉 `React-Query`는 사용자가 사용하는 윈도우가 다른 곳을 갔다가 다시 화면으로 돌아오면 이 함수를 재실행함, 그 재실행 여부 옵션

> **_예제 👇_** 기본 설정 > index.js 에 작성
>
> >

```javascript
import { QueryClient, QueryClientProvider } from "react-query";
import { ReactQueryDevtools } from "react-query/devtools";

const queryClient = new QueryClient();
const root = ReactDOM.createRoot(documnet.getElementById("root"));

root.Render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <ReactQueryDevtools initialIsOpen={true} />
      <App />
    </QueryClientProvider>
  </React.StrictMode>
);
```

> **_예제 👇_** `useQuery` 사용
>
> >

```javascript
const Todo = () => {
  const { isLoading, isError, data, error } = useQuery("todos", fetchTodoList, {
    refetchOnWindowFocus: false,
    retry: 0,
    onSuccess: (data) => {
      console.log(data);
    },
    onError: (e) => {
      console.log(e.message);
    },
  });

  if (isLoading) {
    return (
      <View>
        <Text>{"Loading"}</Text>
      </View>
    );
  }

  if (isError) {
    return (
      <View>
        <Text>{"에러"}</Text>
      </View>
    );
  }

  // Something...
};
```

### 🍀 3. useQueries

- `useQuery`를 비동기로 여러개 실행할 경우 사용
- `promise.all`과 마찬가지로 하나의 배열에 각 `Query`에 대한 상태 값이 객체로 들어옴

> **_예제 👇_** `useQueries` 사용 (lol 룬과 스펠을 받아오는 예시)
>
> >

```javascript
const result = useQueries([
  /**
   * getRune -> Unique Key
   * riot.version -> 파라미터
   */
  {
    queryKey: ["getRune", riot.version],
    queryFn: () => api.getRunInfo(riot.version),
  },
  {
    queryKey: ["getSpell", riot.version],
    queryFn: () => api.getSpellInfo(riot.version),
  },
]);

useEffect(() => {
  console.log(result);
  const loadingFinishAll = result.some((result) => result.isLoading);
  console.log(loadingFinishAll);
}, [result]);
```

> **_예제 👇_** `useQueries` 사용 > `Unique Key` 활용 > `Unique Key`를 배열로 넣으면 `queryFn(쿼리 함수)` 내부에서 변수로 사용 가능
>
> >

```javascript
const result = useQueries([
  {
    queryKey: ["getRune", riot.version],
    queryFn: (params) => {
      // 결과 -> {queryKey: ['getRune', '12.1.1'], pageParam: undifined, meta: undifined}
      console.log(params);
      return api.getRunInfo(riot.version);
    },
  },
  {
    queryKey: ["getSpell", riot.version],
    queryFn: () => api.getSpellInfo(riot.version),
  },
]);
```

### 🍀 4. useMutation

- 값을 바꿀 때 사용하는 api
- 데이터를 생성(`POST`), 수정(`UPDATE`), 삭제(`DELETE`)할 떄 사용하는 `Hook`
- **_특정 함수에서 우리가 원하는 때에 직접 요청을 시작하는 형태_**
- 요청 관련 상태의 관리와 요청 처리 전/후로 실행할 작업을 쉽게 설정할 수 있음
- 첫 번쨰 인자 👉 `Promise`를 반환하는 함수
- 두 번째 인자 👉 **_이 작업이 처리되기 전후로 실행할 함수를 넣음_**(생략가능)
  > - onMutate 👉 요청 직전 처리, 여기서 반환하는 값은 하단 함수들의 context로 사용
  > - onError 👉 데이터 요청이 실패하고 나서 특정 함수를 호출하고 싶을 때 사용
  > - onSuccess 👉 데이터 요청이 성공하고 나서 특정 함수를 호출하고 싶을 때 사용
  > - onSettled 👉 데이터 요청의 성공 여부와 관계없이 요청이 끝나면 특정 함수를 호출하도록 설정
- `return` 값 👉 `useQuery` 와 동일
  - `mutate` 👇
    > - 요청을 시작하는 함수
    > - 첫 번째 인자 👉 API 함수에서 사용할 인자
    > - 두 번쨰 인자
    > - {onSuccess, onSettled, onError} 객체, (생략가능)
    > - option 에 설정된 함수가 먼저 호출되고, mutate 두 번째 파라미터에 넣은 함수가 호출됨
  - `mutateAsync` 👉 `mutate`와 인자는 동일, 함수를 호출했을 때 반환 값이 `Primse`
  - `staus` 👉 요청 상태를 문자열로 변환(idle, loading, error, success)
  - `error` 👉 오류 정보
  - `data` 👉 요청 성공 시 데이터가 담겨있음
  - `reset` 👉 **_상태를 모두 초기화하는 함수_**

> **_예제 👇_** `useMutation` 사용
>
> >

```javascript
const Index = () => {
  const [id, setId] = useState('');
  const [password, setPassword] = useState('');

  const loginMutation = useMutation(loginApi, {
    // variable -> {loginId: xxx, password: xxx}
    onMutate: variable => {
      console.log("onMutate", variable);
    },
    onError: (error, variable, context) => {
      // TODO: Error 핸들링
    },
    onSuccess: (data, variables, context) => {
      console.log("success", data, varibles, context);
    },
    onSettled: () => {
      console.log('end);
    },
  });

    // API 호출에 사용될 인자
  const onSubmit = () => {
    loginMutation.mutate({loginId: id, password: password});
  }
}
```

> **_예제 👇_** `invalidateQueries`를 사용하여 UPDATE 후 GET 함수를 간단하게 실행  
> `mutation` 함수가 성공할 때, `unique key`로 맵핑된 `GET`함수를 `invalidateQueries`에 넣어주면 됨  
> **_만약 `mutation`에서 `return`된 값을 이용해서 `GET`함수의 파라미터를 변경해야 할 경우 `setQueryData`를 사용_**
>
> >

```javascript
const mutation = useMutation(postTodo, {
  onSuccess: () => {
    // postTodo 가 성공하면 todos 로 맵핑된 useQuery API 함수를 실행
    queryClient.invalidateQueries("todos");
  },
});

// 만약 mutation에서 return된 값을 이용해서 GET 함수의 파라미터를 변경해야할 경우 setQueryData를 사용
const queryClient = useQueryClient();

// data 가 fetchTodoById 로 들어감
const mutation = useMutation(editTodo, {
  onSuccess: (data) => {
    queryClient.setQueryData(["todo", { id: 5 }], data);
  },
});

const { status, data, error } = useQuery(["todo", { id: 5 }], fetchTodoById);

mutation.mutate({
  id: 5,
  name: "nkh",
});
```

### 🍀 5. react-suspense 와 react-query 사용

- 비동기를 좀 더 선언적으로 사용할 수 있음.
- `suspense` 를 사용하여 `loading` 을, `Error bundary`를 사용하여 `error handling`을 더욱 직관적으로 할 수 있음.
- `suspense` 를 사용하기 위해 `QueryClient` 에 `option` 을 하나 추가해야 함

> **_예제 👇_** Global 하게 `suspense` 를 사용한다고 정의 > src/index.js 에 선언
>
> >

```javascript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 0
      suspense: true
    }
  }
})

ReactDom.render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  ...
)
```

> **_예제 👇_** 함수마다 `suspense`를 사용하는 예시
>
> >

```javascript
const { data } = useQuery("test", testApi, { suspense: ture });

return (
  <Suspense fallback={<div> loading... </div>}>
    <ErrorBoundary fallback={<div>에러발생</div>}>
      <div>{data}</div>
    </ErrorBoundary>
  </Suspense>
);
```

### 🍀 6. React-Query Cache 데이터 새로고침

- `React-Query`에는 `useQuery`를 사용할 때 입력한 `Unique Key`를 통해 데이터를 만료시키고, 새로 불러올 수 있도록 처리할 수 있다.
- `invalidateQueries` 👇
  - 캐시를 만료시켜 데이터를 새로 고침
  - `Unique Key`를 사용하여 데이터를 만료시키고, API를 재요청하는 방식
- `useQueryClient` 👉 이 `Hook`은 이전에 `App Component`에서 `QueryClientProvider`에 넣었던 `queryClient`를 사용할 수 있게 해줌
  - `getQueryData` 👇
    > - `Unique Key`를 사용하여 Cache Data 를 조회할 수 있음
    > - 데이터가 `undifined`일 수 있으니, 만약 데이터가 존재하지 않는다면 `빈 배열`을 사용하도록 준비
    > - `TypeScript` 환경에서는 `Generic`을 지정하면 반환값의 데이터 type 을 설정할 수 있음
  - `setQueryData` 👇
    > - Cache Data 를 업데이트하는 메서드
    > - 데이터를 두 번째 인자에 넣어도되고, `업데이트 함수`형태의 값을 인자로 넣을 수 있음
    > - 만약 `업데이트 함수` 형태로 넣는다면 `getQueryData`는 생략 가능

> **_예제 👇_** `QueryClient`를 사용하여 데이터 새로고침
>
> >

```javascript
const queryClient = useQueryClient();

const { mutate: write } = useMutation(writerArticle, {
  onSuccess: () => {
    queryClient.invalidateQueries("articles");
    navigation.goBack();
  },
});
```

> **_예제 👇_** `QueryClient`로 `Cache Data`를 직접 `UPDATE`하기  
> **_API를 재요청하지 않고 `Cache Data`를 `UPDATE`_**
>
> >

```typescript
const queryClient = useQueryClient();

const { mutate: write } = useMutation(writeArticle, {
  onSuccess: (article) => {
    const articles = queryClient.getQueryData<Article[]>("articles") ?? [];
    queryClient.setQueryData("articles", articles.concat(article));
  },
});
```

> **_예제 👇_** `getQueryData` 생략, `UPDATE` 함수 형태의 값을 인자로 넣음  
> `Unique Key`로 데이터 조회 후 그 데이터를 `UPDATE` 함수를 사용하여 `UPDATE`
>
> >

```typescript
const queryClient = useQueryClient();

const { mutate: write } = useMutation(writeArticle, {
  onSuccess: (article) => {
    queryClient.setQueryData<Article[]>("articles", (articles) =>
      (articles ?? []).concat(article)
    );
  },
});
```
