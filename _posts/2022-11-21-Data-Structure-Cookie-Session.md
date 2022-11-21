---
title: 자료구조 Cookie, Session
author: Narvis2
date: 2022-11-21 09:02:00 +0900
categories: [DataStructure]
tags: [cookie, session, data structure]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 `쿠키(Cookie)`와 `세션(Session)`에 대하여 알아보고자 합니다.

## 🍀 사용 이유

---

- `Cookie`와 `Session`은 주로 자주 사용되는 데이터를 꺼내사용해야 하는 경우 사용됨
- `HTTP`의 특징으로 인해 사용 👇
  > - `Connectionless(비 연결지향)` 👉 `HTTP`는 기본적으로 `요청`&`응답`만 하고 끊어버림
  > - `Stateless(상태정보유지안함)` 👉 `HTTP`는 상태정보를 저장하지 않음, 따라서 `Session` 혹은 `Cookie`로 데이터를 저장함

## 🍀 Cookie (쿠키)

---

- 브라우저에 **_휘발성 데이터_** 를 저장하는 방식
- 유저들의 효율적이고 안전한 웹 사용을 보장하기 위하여 사용
- 웹 사이트 접속시 접속자의 개인 장치에 다운로드되고, 브라우저에 저장되는 작은 `Text`파일
- 웹 사이트는 쿠키를 통해 접속자의 장치를 인식하고, 접속자의 설정과 과거 이용내역에 대한 일부 데이터를 저장
  > **_참고_** 👉 특정 사이트에 가면, **_자동적으로 아이디와 비밀번호가 입력_** 되는 경우
- 이름, 값, `만료날짜`, 경로 정보가 들어있음
  > **_참고_** 👉 `일정 시간동안` 데이터를 저장할 수 있어서 **_로그인 상태를 유지_** 하거나, **_사용자 정보를 일정 시간 동안 유지해야 하는 경우_** 에 주로 사용
- `Cookie`가 생성되면 브라우저는 `Request Header`에 자동적으로 `Cookie` 정보를 넣어서 보내게 됨
  > **_참고_** 👉 매 요청마다 `Cookie`를 담아서 보냄
- ✅ 사용 경우
  > - 1️⃣ `Session`관리 👉 서버에 저장해야 할 로그인, 장바구니, 게임 스코어 등의 정보 관리
  > - 2️⃣ `개인화` 👉 사용자 선호 **_테마_** 등의 세팅
  > - 3️⃣ `트래킹` 👉 사용자의 행동을 기록하고 분석하는 용도
- ❗️ 단점 ❗️
  > - 사용자 정보가 누출될 수 있음. (보안성 낮음)

### ☘️ Cookie 의 Flow (흐름)

![Desktop View](/assets/img/datastructure/cookie-flow.png){: width="100%" }

- 1️⃣ 웹 브라우저(`Client`)에서 `서버`로 요청(`request`)
- 2️⃣ `서버`에서 상태를 유지하고 싶은 값을 `Cookie`로 생성
- 3️⃣ `서버`가 응답(`response`)할 때 `HTTP` `Header`에 `Cookie`를 포함해서 `Client`에게 전송
- 4️⃣ `Cookie`는 `Client`에 파일 단위로 저장
- 5️⃣ 이후 `Client`에서 이 `Cookie`를 포함하여 요청(`request`)

## 🍀 Session (세션)

---

- 서버 메모리에 저장되는 정보
  > **_참고_** 👉 서버에 저장되기 때문에 `Cookie`와는 달리 **_사용자 정보가 노출되지 않음_**
- 수명 👉 `Client`가 웹 서버에 연결된 순간부터 웹 브라우저를 닫아 서버와의 `HTTP` 끝날 때 까지의 기간
- 서버에 `Session`에 대한 정보(Session 상태, Client 상태, Session 데이터 등)를 저장해 놓고 `Session` `Cookie`(고유한 Session ID 값)를 `Client`에게 주어 서버가 `Client`를 식별할 수 있도록 하는 방식 자체를 의미
- ✅ 특징
  > - 1️⃣ 따로 용량의 제한이 없음(서버의 능력에 따라 다를 수 있음)
  > - 2️⃣ 서버에 `Session` 객체를 생성하며, 각 `Client` 마다 고유한 `Session` `ID` 값을 부여함.
  > - 3️⃣ `Cookie`를 사용하여 `Session ID` 값을 `Client`에 보냄
  > - 4️⃣ 웹 브라우저가 종료되면 `Session`, `Cookie`는 삭제됨

### ☘️ Session 의 Flow (흐름)

![Desktop View](/assets/img/datastructure/session-flow.png){: width="100%" }

- 1️⃣ 웹 브라우저(`Client`)에서 `서버`로 요청(`request`)
- 2️⃣ `서버`내부에서 해당 `Client`에게 `Session ID`를 부여
- 3️⃣ `서버`가 응답(`response`)할 때 `HTTP` `Header`에 `Session ID`를 포함해서 `Client`에게 전송
- 4️⃣ 웹 브라우저(`Client`)는 이후 웹 브라우저(`Client`)를 닫기까지 부여된 `Session ID`를 기억함
- 5️⃣ 이후 `Client`에서 이 `Session ID`를 포함하여 요청(`request`)
- 6️⃣ 브라우저를 종료하면 `Session ID`를 삭제

## 🍀 Cookie 와 Session 의 차이

---

|           | Cookie          | Session              |
| --------- | --------------- | -------------------- |
| 저장 위치 | 브라우저(Local) | 서버                 |
| 보안성    | 낮음            | 높음                 |
| 수명      | 반영구          | 브라우저 종료시 삭제 |
| 속도      | 빠름            | 느림                 |
