---
title: React-Native Redux Middleware
author: Narvis2
date: 2022-11-30 16:42:00 +0900
categories: [React-Native, Redux]
tags: [redux, react-native, redux-middleware]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 `Redux Middleware`에 대하여 알아보도록 하겠습니다.  
`Redux Middleware`는 `REST API` 요청 상태를 관리하기 위해 주로 사용합니다.  
`Redux Middleware`를 사용하면 `Action`이 `Dispatch`된 다음, `Reducer`에서 해당 `Action`을 받아와서 `Update`하기 전에 추가적인 작업을 할 수 있습니다.  
자세한 건 밑에서 알아보도록 하겠습니다.

## 🍀 Redux Middleware

- `Action`을 `Dispatch`했을 때 `Reducer`에서 이를 처리하기에 앞서 **_사전에 지정된 작업을 실행할 수 있게 해줌_** 즉, **_`Action`과 `Reducer`사이의 중간자_**
- 특정 조건에 따라 `Action`이 무시되게 만들 수 있음
- `Action`을 `console`에 출력하거나, 서버쪽에 `logging`을 할 수 있음
- `Action`이 `Dispatch` 되었을 때 이를 수정해서 `Reducer`에게 전달되도록 할 수 있음
- 특정 `Action`이 발생했을 때 이에 기반하여 다른 `Action`이 발생되도록 할 수 있음
- 특정 `Action`이 발생했을 때 특정 `Javascript` 함수를 실행시킬 수 있음
- ✅ **_`Redux Middleware`는 비동기 작업을 처리할 때 주로 사용_**
  > - 비동기 작업에 관련된 `Middleware` 라이브러리
  >   > - `redux-thunk`
  >   >   > - 함수를 기반으로 작동
  >   >   > - `redux-toolkit`에 내장되어 있어 적용하기가 간단
  >   >   > - `redux`에서 `type`을 지닌 객체가 아닌, 함수 타입을 `Dispatch` 할 수 있게 해줌
  >   > - `redux-saga`
  >   >   > - `Generator`를 기반으로 사용
  >   >   > - 특정 경우에 특정 `Action`을 모니터링 할 수도 있음
  >   >   > - 특정 `Action`이 `Dispatch` 되었을때 원하는 함수를 호출하거나, 또는 `라우터`를 통해 다른 주소로 이동하는 것이 가능
  >   > - `redux-observable`
  >   >   > - `RxJs`를 기반으로 작동
  >   >   > - 특정 경우에 특정 `Action`을 모니터링 할 수도 있음
  >   >   > - 특정 `Action`이 `Dispatch` 되었을때 원하는 함수를 호출하거나, 또는 `라우터`를 통해 다른 주소로 이동하는 것이 가능
  >   > - `redux-promise-middleware`

### ☘️ Redux-Thunk

- `redux`에서 비동기 작업을 처리 할 때 가장 많이 사용하는 `middleware`
- 함수를 기반으로 작동
- `action`객체가 아닌 함수를 `Dispatch`할 수 있게 해줌.
- `Redux DevTools`와 함께 사용
- 적용 👉 `yarn add redux-thunk`

  > 1️⃣ **_예제_** ) `redux-thunk` `middleware` 적용 > `App.tsx` 👇

  ```typescript
  import rootReducer from "./src/modules";
  import { applyMiddleware, createStore } from "redux";
  import thunk from "redux-thunk";

  const store = createStore(rootReducer, applyMiddleware(thunk));
  ```

  > 2️⃣ **_예제_** ) `thunk` 함수 생성 > `Redux Module` 에 작성 👇

  ```typescript
  // ✅ thunk 함수 생성
  export const increaseAsync = () => (dispatch: Dispatch<CounterAction>) => {
    setTimeout(() => dispatch(increase()), 1000);
  };
  export const decreaseAsync = () => (dispatch: Dispatch<CounterAction>) => {
    setTimeout(() => dispatch(decrease()), 1000);
  };
  ```

  > **_설명_** 👉 `CounterAction` 을 `AnyAction`으로 교체해도 상관 없음

  > 3️⃣ **_사용_** ) `thunk` 함수 사용 👇

  ```typescript
  /**
   * ✅ 현재 상태를 조회
   * 상태를 조회할 때는 state의 타입을 RootState로 지정해야함
   */
  const counter = useSelector((state: RootState) => state.counter);
  // Dispatch 함수를 가져옴 (action을 발생시켜 상태를 업데이트)
  const dispatch = useDispatch();

  function onIncrease() {
    dispatch(increaseAsync());
  }

  function onDecrease() {
    dispatch(decreaseAsync());
  }
  ```

### ☘️ Redux-Thunk + Promise

- ⚠️ `Promise`를 다루는 `Redux Module`을 다룰 때 주의 사항

  - 1️⃣ `Promise`가 시작(`start`), 성공(`success`), 실패(`failure`)했을 때 다른 `Action`을 `Dispatch` 해야함
  - 2️⃣ 각 `Promise`마다 `thunk` 함수를 만들어주어야 함
  - 3️⃣ `Reducer`에서 `Action`에 따라 로딩중(`loading`), 결과(`result`), 에러(`error`) 상태를 변경해주어야 함

  > 1️⃣ **_예제_** ) `modules/posts.tsx` > `Redux Module` 작성 👇

  ```typescript
  import { AnyAction, Dispatch } from "redux";
  import * as postsAPI from "../api/posts"; // api/posts 안의 함수 모두 불러오기

  // ✅ 밑의 대문자 상수는 모두 Action Type임

  // 포스트 여러개 조회하기
  const GET_POSTS = "GET_POSTS"; // 요청이 시작됨
  const GET_POSTS_SUCCESS = "GET_POSTS_SUCCESS"; // 성공
  const GET_POSTS_ERROR = "GET_POSTS_ERROR"; // 실패

  // 포스트 하나 조회하기
  const GET_POST = "GET_POST"; // 요청이 시작됨
  const GET_POST_SUCCESS = "GET_POST_SUCCESS"; // 성공
  const GET_POST_ERROR = "GET_POST_ERROR"; // 실패

  /**
   * thunk 를 사용할 때, 꼭 모든 Action들에 대하여 Action 생성함수를 만들 필요는 없음
   * 그냥, thunk 함수에서 바로 action 객체를 만들어 주어도 괜찮음
   */
  export const getPosts = () => async (dispatch: Dispatch<AnyAction>) => {
    dispatch({ type: GET_POSTS });

    try {
      const posts = await postsAPI.getPosts();
      dispatch({ type: GET_POSTS_SUCCESS, posts });
    } catch (error) {
      dispatch({ type: GET_POSTS_ERROR, error: error });
    }
  };

  // ✅ thunk 함수에서도 파라미터를 받아와서 사용 가능
  export const getPost =
    (id: number) => async (dispatch: Dispatch<AnyAction>) => {
      dispatch({ type: GET_POST });

      try {
        const post = await postsAPI.getPostById(id);
        dispatch({ type: GET_POST_SUCCESS, post });
      } catch (error) {
        dispatch({ type: GET_POST_ERROR, error: error });
      }
    };

  // ✅ 초깃값
  const initialState = {
    posts: {
      loading: false,
      data: null,
      error: null,
    },
    post: {
      loading: false,
      data: null,
      error: null,
    },
  };

  // ✅ Reducer 함수
  export default function posts(state = initialState, action) {
    switch (action.type) {
      case GET_POSTS:
        return {
          ...state,
          posts: {
            loading: true,
            data: null,
            error: null,
          },
        };

      case GET_POSTS_SUCCESS:
        return {
          ...state,
          posts: {
            loading: false,
            data: action.posts,
            error: null,
          },
        };

      case GET_POSTS_ERROR:
        return {
          ...state,
          posts: {
            loading: false,
            data: null,
            error: action.error,
          },
        };

      case GET_POST:
        return {
          ...state,
          post: {
            loading: true,
            data: null,
            error: null,
          },
        };

      case GET_POST_SUCCESS:
        return {
          ...state,
          post: {
            loading: true,
            data: action.post,
            error: null,
          },
        };

      case GET_POST_ERROR:
        return {
          ...state,
          post: {
            loading: true,
            data: null,
            error: action.error,
          },
        };

      default:
        return state;
    }
  }
  ```

  > 2️⃣ **_예제_** ) `thunk` 사용 👇

  ```typescript
  const { data, loading, error } = useSelector((state) => state.posts.posts);
  const dispatch = useDispatch();

  // 컴포넌트 마운트 후 포스트 목록 요청
  useEffect(() => {
    dispatch(getPosts());
  }, [dispatch]);
  ```
