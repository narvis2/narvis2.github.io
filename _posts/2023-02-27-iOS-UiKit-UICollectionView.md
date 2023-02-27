---
title: iOS Swfit UIKit UICollectionView
author: Narvis2
date: 2023-02-27 21:30:00 +0900
categories: [Swift, UIKit]
tags: [iOS, UIKit, UITabBarController, UITabBar]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `iOS` `UIKit` `UICollectionView`에 대하여 알아보도록 하겠습니다.

## 🦋 UICollectionView

- 데이터 항목의 정렬된 `Collection`을 관리하고, `Cutom`한 `Layout`을 사용해 표시하는 객체
- `List`, `Slide` 등 다양한 형태로 표현 가능

### 🌸 UICollectionView 구성 요소

- `Supplementary View`, `Cell`, `Decoration View`로 구성되어 있음
- `Supplementary View` 👉 `Section`에 대한 정보를 표시 (`Header`, `Footer`라고 생각하면 됨, 필수 구현 x)
- `Cell` 👉 `CollectionView`의 `Content` 표시
- `Decoration View` 👉 `CollectionView`에 대한 **_배경_**을 꾸밀 때 사용

### 🌸 UICollectionViewFlowLayout

- 1️⃣ `Flow Layout` 객체를 작성하고 `Collection View`에 이를 할당한다.
- 2️⃣ `Cell`의 `width`, `height`를 정한다.
- 3️⃣ 필요한 경우 `Cell`들 간의 좌우 최소 간격, 위아래 최소 간격을 설정한다.
- 4️⃣ `Section`에 `Header`와 `Footer`가 있다면 이것들의 크기를 지정한다.
- 5️⃣ `Layout`의 `Scroll` 방향을 설정한다.

### 🌸 UICollectionViewDataSource

- `Collection View`로 보여지는 `Content`들을 관리하는 객체
- `Collection View`의 `Content(데이터)`를 관리하고 해당 `Content`를 표현하는 데 필요한 `View`

  > **_참고_** 👇
  >
  > - `func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int` 👉 지정된 섹션에 표시할 항목의 개수를 묻는 메서드
  >
  > - `func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell` 👉 컬렉션뷰의 지정된 위치에 표시할 셀을 요청하는 메서드.
  >
  > - `optional func numberOfSections(in collectionView: UICollectionView) -> Int` 👉 컬렉션뷰의 섹션의 개수를 묻는 메서드. 이 메서드를 구현하지 않으면 섹션 개수 기본 값은 1.
  >
  > - `optional func collectionView(_ collectionView: UICollectionView, canMoveItemAt indexPath: IndexPath) -> Bool` 👉 지정된 위치의 항목을 컬렉션뷰의 다른 위치로 이동할 수 있는지를 묻는 메서드.
  >
  > - `optional func collectionView(_ collectionView: UICollectionView, moveItemAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath)` 👉 지정된 위치의 항목을 다른 위치로 이동하도록 지시하는 메서드

### 🌸 UICollectionViewDelegate

- `Content`의 표현, 사용자와의 상호작용과 관련된 것들을 관리하는 객체
- 셀의 선택 및 강조표시를 관리하고 해당 셀에 대한 작업을 수행할 수 있는 메서드를 정의
- 필수로 구현하지 않아도 상관없음

  > **_참고_** 👇
  >
  > - `optional func collectionView(_ collectionView: UICollectionView, shouldSelectItemAt indexPath: IndexPath) -> Bool` 👉 지정된 셀이 사용자에 의해 선택될 수 있는지 묻는 메서드.
  >
  > - `optional func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath)` 👉 지정된 셀이 선택되었음을 알리는 메서드.
  >
  > - `optional func collectionView(_ collectionView: UICollectionView, shouldDeselectItemAt indexPath: IndexPath) -> Bool` 👉 지정된 셀의 선택이 해제될 수 있는지 묻는 메서드. 선택 해제가 가능한 경우 true로 응답하며, 그렇지 않다면 false로 응답.
  >
  > - `optional func collectionView(_ collectionView: UICollectionView, didDeselectItemAt indexPath: IndexPath)` 👉 지정된 셀의 선택이 해제되었음을 알리는 메서드.
  >
  > - `optional func collectionView(_ collectionView: UICollectionView, shouldHighlightItemAt indexPath: IndexPath) -> Bool` 👉 지정된 셀이 강조될 수 있는지 묻는 메서드. 강조해야 하는 경우 true로 응답하며, 그렇지 않다면 false로 응답.
  >
  > - `optional func collectionView(_ collectionView: UICollectionView, didHighlightItemAt indexPath: IndexPath)` 👉 지정된 셀이 강조되었을 때 알려주는 메서드.
  >
  > - `optional func collectionView(_ collectionView: UICollectionView, didUnhighlightItemAt indexPath: IndexPath)` 👉 지정된 셀이 강조가 해제될 때 알려주는 메서드.
