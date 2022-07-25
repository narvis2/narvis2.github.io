---
title: 자료구조 Hash, HashMap, LinkedHashMap
author: Narvis2
date: 2022-07-25 13:37:00 +0900
categories: [DataStructure]
tags: [hash, hashCode, hashTable, hashMap, linkedHashMap]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 Hash에 대하여 알아보고 더불어 HashMap, LinkedHashMap에 대하여 간략히 알아보고자 합니다.

## Hash
- 단방향 암호화 기법으로 산술 연산을 통하여 **"입력된 값을 출력 데이터가 있는 곳을 알 수 있는 값으로 바꾸는 것"**을 의미합니다.
- 데이터가 저장된 객체들과 일일이 동등성을 비교하지 않고, 산술 연산으로 데이터에 접근할 수 있기 때문에 속도가 빠릅니다.
- Hash Fuction을 통해 입력 값이 데이터가 있는 곳을 알 수 있는 출력 값으로 연결될 수 있으므로 **Hash Fuction은 입력값을 출력값으로 Mapping 해주는 함수입니다.**

## HashCode
- 객체를 식별하는 하나의 정수 값입니다.
- Object의 hashCode() 메서드는 **객체의 메모리 번지를 이용해서 HashCode를 만들어 Return**하기 때문에 객체마다 다른 값을 가지고 있습니다.
- 객체의 값을 동승성 비교 시 hashCode()를 Overriding 할 필요성이 있는데 hashCode()를 Overrding 할 때는 equals()도 반드시 Overring 해야합니다.
- 실행 중에 객체의 유일한 integer 값을 반환합니다.
> **_참고_**❗️ equals 와 hashCode의 관계
> - Java 프로그램을 실행하는 동안 equals에 사용된 정보가 수정되지 않았다면 hashCode는 항상 동일한 정수값을 반환해야 합니다.(Java의 프로그램을 실행할때 마다 달라지는 것은 상관이 없습니다.)
> - 두 객체가 equals 에 의해 동일하다면, 두 객체의 hashCode() 값 또한 일치해야 합니다.
> - 두 객체가 equals 에 의해 동일하지 않다면, 두 객체의 hashCode() 값은 일치하지 않아도 됩니다.
- **HashTable, HashSet, HashMap 과 같은 자료 구조는 "자료를 저장하기 위한 위치를 선택하기 위해 HashCode를 이용"합니다.**
> **_참고_**❗️ HashTable, HashSet, HashMap 의 객체 동등성 비교 방식
> 1. hashCode() 메서드를 실행하여 Return 된 HashCode 값이 같은 지를 비교합니다. 
> 2. HashCode 값이 다르면 다른 객체로 판단하고, HashCode 값이 같으면 equals() 메소드로 다시 비교합니다.
> 3. 위의 2개가 모두 맞아야 동등 객체로 판단합니다. 즉, HashCode 값이 다른 Entries 끼리는 동치성 비교를 시도 조차 하지않습니다.

## HashTable 작동 원리
- HashMap, HashSet 과 동작원리는 같습니다.  
  
1. HashTable 은 <Key, Value> 형태로 데이터를 저장합니다. 이 때 Hash Fuction 을 이용하여 Key 값을 기준으로 고유한 식별값인 Hash 값을 만들고(HashCode가 해시 값을 만드는 역할을 합니다.) 이 해시 값을 버킷(Bucket)에 저장합니다.
2. 하지만 HashTable 크기는 한정적이기 때문에 같은 서로 다른 객체라 하여도 같은 Hash값을 갖게될 수 있습니다.(이것을 Hash Collisions 즉, 해쉬 충돌 이라고 합니다.)
3. 이런 경우 해당 버킷(Bucket)에 LinkedList 형태로 객체를 추가합니다.
> **_참고_**❗️ LinkedList 는 아이템의 갯수가 8개 이상으로 넘어가면 TreeMap 자료 구조로 저장됩니다.
4. 이 처럼 같은 Hash 값을 버킷(Bucket)안에 다른 객체가 있는 경우 equals가 사용됩니다.  
  
> 1️⃣ HashTable 에 put 메서드로 객체를 추가하는 경우
> - 값이 같은 객체가 이미 존재한다면(equals()가 true) 기존 객체를 덮어씁니다.
> - 값이 같은 객체가 없다면(equals()가 false) 해당 entry를 LinkedList에 추가합니다.  
  
> 2️⃣ HashTable 에 get 메서드로 객체를 조회하는 경우
> - 값이 같은 객체가 있다면(equals()가 true) 그 객체를 return 합니다.
> - 값이 같은 객체가 없다면(equals()가 false) null을 return 합니다.

## HashMap
- Hash 기법을 사용하여 구현된 Map 입니다. 즉, Hash 기법을 사용하여 데이터를 보관하는 자료 구조입니다.
- **자료를 저장하기 위한 위치를 선택하기 위해 HashCode를 이용합니다.**
- **데이터가 추가되는 순서를 유지하지 않으나** Hashing 기법의 특징인 **빠른 접근 속도**가 장점입니다.
- 데이터에 접근하거나 감색할 때 데이터를 순회(반복)하면서 일일이 빌교하는 일반적인 자료 구조와 다르게 **key 값을 통해 한 번의 산술연산으로 데이터에 접근할 수 있기 때문에 아주 빠른 접근 속도와 검색 속도가 특징**입니다.
- 데이터를 입력 순서대로 저장하거나 정렬하여 저장하지 않습니다.
> **_참고_**❗️ 데이터를 순서대로 저장하고 싶으면 LinkedHashMap을 사용합니다.
- 배열을 기반으로 데이터를 관리하기 때문에 데이터의 총 개수를 미리 얼추 알고있다면 초기 용량을 정해주는 것이 좋습니다.

## LinkedHashMap
- Hash 기법을 사용하여 데이터를 보관하는 자료 구조입니다.
- **데이터를 추가한 순서대로 보관합니다.(데이터를 순서대로 저장)** 따라서 LinkedHashMap 을 반복문 돌려서 순회하면 순서대로 접근이 가능합니다.
- 데이터의 총 개수를 미리 알고 있다면 초기 용량을 정해주는 것이 좋습니다.