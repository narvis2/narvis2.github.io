---
title: iOS Swfit UIKit UITableView
author: Narvis2
date: 2023-02-27 18:20:00 +0900
categories: [Swift, UIKit]
tags: [iOS, UIKit, UITableView]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `iOS` `UIKit` `UITableView`에 대하여 알아보도록 하겠습니다.

## 🦋 UITableView

- 여러 개의 Cell을 가지고 있고, 하나의 열과 여러 줄의 행을 지니고 있으며, **_수직_** 으로만 `Scroll`이 가능
- `Section`을 이용해 행을 그룹화하여 `Content`를 좀 더 쉽게 탐색할 수 있음
- `Section`의 `Header`와 `Footer`에 `View`를 구성하여 추가적인 정보를 표시할 수 있음

### 🌸 UITableViewDelegate

- `TableView`의 시각적인 부분을 설정하고, 행의 `Action` 관리, 엑세서리 `View` 지원 그리고 `TableView`의 개별 행 편집을 도와 줌
- `TableView`의 동작과 외관을 담당
- `View` 가 변경되는 상황을 `Delegate`가 담당
- 행의 높이, 행을 선택하면 어떤 `Action`을 할건지 정의

  > **_참고_** 👇
  >
  > - `optional func tableView(UITableView, heightForRowAt: IndexPath)` 👉 특정 위치 행의 높이를 묻는 메서드
  >
  > - `optional func tableView(UITableView, indentationLevelForRowAt: IndexPath)` 👉 특정 위치 행의 들여쓰기 수준을 묻는 메서드
  >
  > - `optional func tableView(UITableView, didSelectRowAt: IndexPath)` 👉 지정된 행이 선택되었음을 알리는 메서드
  >
  > - `optional func tableView(UITableView, didDeselectRowAt: IndexPath)` 👉 지정된 행의 선택이 해제되었음을 알리는 메서드
  >
  > - `optional func tableView(UITableView, viewForHeaderInSection: Int)` 👉 특정 `Section`의 `HeaderView` 를 요청하는 메서드
  > - `optional func tableView(UITableView, viewForFooterInSection: Int)` 👉 특정 `Section`의 `FooterView` 를 요청하는 메서드
  >
  > - `optional func tableView(UITableView, heightForHeaderInSection: Int)` 👉 특정 `Section`의 `HeaderView`의 높이를 물어보는 메서드
  > - `optional func tableView(UITableView, heightForFooterInSection: Int)` 👉 특정 `Section`의 `FooterView`의 높이를 물어보는 메서드
  >
  > - `optional func tableView(UITableView, willBeginEditingRowAt: IndexPath)` 👉 `TableView`가 편집모드에 들어갔음을 알리는 메서드
  >
  > - `optional func tableView(UITableView, didEndEditingRowAt: IndexPath?)` 👉 `TableView`가 편집모드에서 빠져나왔음을 알리는 메서드

### 🌸 UITableViewDataSource

- `TableView`를 생성하고 수정하는데 필요한 정보를 `TableView` 객체에 제공
- `data`를 받아 `View`를 그려주는 역할을 담당
- 총 `Section`의 개수, `Section`의 행의 개수, `Section`에 어떤 정보를 표시할 것 인지 정의

  > **_참고_** 👇
  >
  > - `func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int` 👉 각 `Settion`애 표시할 행의 개수를 묻는 메서드
  >
  > - `func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell` 👉 특정 `Index` `Row`의 `Cell`에 대한 정보를 넣어 `Cell`을 반환하는 메서드
  >
  > - `optional func numberOfSections(in tableView: UITableView) -> Int` 👉 총 `Section` 개수를 묻는 메서드
  >
  > - `optional func tableView(UITableView, titleForHeaderInSection: Int) -> String?` 👉 특정 `Section`의 `Header Title`을 묻는 메서드
  >
  > - `optional func tableView(UITableView, titleForFooterInSection: Int) -> String?` 👉 특정 `Section`의 `Footer Title`을 묻는 메서드
  >
  > - `optional func tableView(UITableView, canEditRowAt: IndexPath) -> Bool` 👉 특정 위치의 행이 편집 가능한지 묻는 메서드
  >
  > - `optional func tableView(UITableView, canMoveRowAt: IndexPath) -> Bool` 👉 특정 위치의 행을 재정렬 할 수 있는지 묻는 메서드
  >
  > - `optional func sectionIndexTitles(for tableView: UITableView) -> [String]?` 👉 `TableView` `Section` `Index Title`을 묻는 메서드
  >
  > - `optional func tableView(UITableView, sectionForSectionIndexTitle: String, at: Int) -> Int` 👉 `Index`에 해당하는 `Section`을 알려주는 메서드
  >
  > - `optional func tableView(UITableView, commit: UITableViewCell.EditingStyle, forRowAt: IndexPath)` 👉 스와이프 모드, 편집 모드에서 버튼을 선택하면 호출되는 메서드, 해당 메서드에서는 행에 변경사항을 `Commit` 해야함
  >
  > - `optional func tableView(UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath)` 👉 행이 다른 위치로 이동되면 어디에서 어디로 이동했는지 알려주는 메서드
