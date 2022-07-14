---
title: 자료구조 스택 Stack, 큐 Queue 에 대하여
author: Narvis2
date: 2022-07-14 10:02:00 +0900
categories: [DataStructure]
tags: [stack, queue, data structure]
---

저는 비전공 개발자입니다.  
다른 직종의 회사에서 근무하면서 퇴근 후 독학으로 Android 개발을 시작하였습니다.  
독학 9개월차에 한 기업에 입사지원을 하게 되었고 그 기업의 면접에서 나왔던 질문이 Stack과 Queue였습니다.  
기초가 부족했던 저는 해당 질문에 답을 하지 못하였습니다. 그럼에도 결국 취업은 하였지만..  
저와 같은 길을 걷는 분들께 작은 도움이 되고 싶어 해당 포스팅을 작성합니다.  
본 포스팅은 Java 의 Stack과 Queue를 중심으로 설명합니다.  
매우 기초적인 부분을 설명드릴 것 이므로 더욱 자세한 정보를 원하시면 구글링을 추천드립니다.  
그럼 자료구조 **_Stack_**과 **_Queue_**에 대하여 알아보겠습니다.

## Stack (Last In First Out)
---
![Desktop View](/assets/img/datastructure/stack.png){: width="50%" }
- Stack의 입출력 순서는 **_LIFO(Last In First Out)_** 즉, **_"후입선출"_** 방식입니다. 위의 그림을 보시면 이해가 편하실 겁니다. Push는 "삽입" Pop은 "삭제"라고 보시면 되겠습니다. 간단하게 "처음 들어온 것이 마지막으로 나간다." 라고 생각하시면 됩니다.
- 데이터를 임시 저장할 때 사용합니다.

### Stack 의 Method 
- push(E item) [삽입] : 파라미터로 들어오는 요소를 해당 Stack의 제일 상단에 **_"삽입"_**합니다.  
  
- Object pop() [추출, 제거] : 해당 Stack의 제일 상단에 있는(제일 마지막으로 저장된) 요소를 반환하고, 해당 요소를 Stack에서 **_"제거"_**합니다.  
  
- boolean empty() : 해당 Stack이 비어있으면 true, 비어있지 않으면 false를 반환합니다. 즉, Stack이 비어있는지 유무를 확인합니다.
  
- Object peek() [추출] : 해당 Stack의 제일 상단에 있는(제일 마지막으로 저장된) 요소를 반환합니다.(Pop() 처럼 반환 후 제거하지 않습니다.)
  
- int search(Object o) [찾기] : 해당 Stack에서 파라미터로 들어오는 객체가 존재하는 위치의 Index를 반환합니다. 이때, Index는 제일 상단에 있는(제일 마지막으로 저장된) 요소의 위치부터 0이 아닌 1부터 시작합니다.
> **사용 사례** -> 웹 브라우저 방문 기록(뒤로가기), 실행 취소, 후위 표기법 계산  
특히 **_뒤로가기_**를 생각하시면 이해가 쉽습니다. 뒤로가기 시 제일 마지막에 띄웠던 브라우저부터 순서대로 나오게 됩니다.

## Queue (First In First Out)
---
![Desktop View](/assets/img/datastructure/queue.png){: width="60%" }
- Queue의 입출력 순서는 **_FIFO(First In First Out)_** 즉, **_"선입선출"_** 방식입니다. 위의 그림을 보시면 이해가 편합니다. 간단하게 "처음 들어온 것이 처음 나간다." 라고 생각하시면 됩니다.
- Stack과 마찬가지로 데이터를 임시 저장할 때 사용합니다.
- 가장 첫 번째 요소와 제일 끝 원소로만 접근이 가능합니다.  

### Queue 의 Method
- boolean add(E e) : 해당 Queue의 맨 뒤에 요소를 삽입합니다. 삽입이 성공하면 **_"ture"_**, Queue에 여유공간이 없어 삽입에 실패하면 **_"IllegalStateException"_** 을 발생 시킵니다.  
  
- E element() : 해당 Queue의 맨 앞에 있는(제일 먼저 저장된) 요소를 반환하고, 해당 요소를 **_"제거"_**합니다.
  
- boolean offer(E e) : 그림의 enqueue의 역할과 같으며, 해당 Queue의 맨 뒤에 요소를 삽입합니다. 삽입이 성공하면 **_"true"_**, Queue에 여유공간이 없어 삽입에 실패하면 IllegalStateException이 아닌 **_"false"_**를 반환합니다.
  
- E peek() : 해당 Queue의 맨 앞에 있는(제일 먼저 저장된) 요소를 반환하고, Queue가 비어있는 경우 Null을 반환합니다. (해당 요소를 Queue에서 제거하지 않습니다.)
  
- E poll() : 해당 Queue의 맨 앞에 있는(제일 먼저 저장된) 요소를 반환하고, Queue가 비어있는 경우 Null을 반환하며 **_"해당 요소를 Queue에서 제거"_**합니다.
  
- E remove() : 해당 Queue의 맨 앞에 있는(제일 먼저 저장된) 요소를 제거합니다.
> **사용 사례** -> 최근 사용 문서, 은행 업무, 콜센터 대기 시간, Buffer, Cache  
즉, 시간 순서대로 처리할 필요가 있을 때 주로 사용한다.  

## 마치며..
지금까지 자료구조 Stack과 Queue에 대하여 알아보았습니다.  
도움이 되셨길 바라며 더 유용한 정보로 다시 찾아뵙겠습니다.