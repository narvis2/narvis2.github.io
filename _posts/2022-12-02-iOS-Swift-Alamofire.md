---
title: iOS SwiftUi Alamofire
author: Narvis2
date: 2022-12-02 10:59:00 +0900
categories: [Swift, Alamofire]
tags: [iOS, Alamofire]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `Swift`에서 `http` 통신을 하기 위해 사용하는 `Alamofire`에 대하여 알아보도록 하겠습니다.

# 🍀 Alamofire

---

- `URLSession` 에서 불편한 점을 한번더 보완한 라이브러리
- `URLSession`을 기반으로 하여 어려운 네트워킹 작업을 감춰주기 때문에 **_주요 로직에 집중할 수 있게 해줌_**
- `GET`, `POST`, `Download`, `DELETE`, `PATCH` 등 다양한 `Http` 통신을 지원
- ✅ 즉, `Alamofire`는 **_`HTTP`네트워킹을 하는데 자주 사용하게 되는 코드나 함수를 더 쉽게 사용할 수 있도록 모아놓은 것_**
- ℹ️ [Github URL](https://github.com/Alamofire/Alamofire)
- 👍 **_대표적인 기능_**
  - `AF.upload` 👉 파일을 업로드함
  - `AF.download` 👉 파일을 다운로드 하거나 이미 진행중인 다운로드를 재개함
  - `AF.request` 👉 파일 전송과 무관한 다른 `HTTP` 요청

## ☘️ AF.request

- 첫 번째 파라미터 👉 `url`
- `method` 👉 통신방식 선택 (`get`, `post`, `delete`, `patch`, `put` 등)
- `parameters` 👉 `paramters` 가 없는 경우 `nil` 값 할당, **_`[String:String]` 형태로 보낼 수 있음_**
  > `paramters: ["foo": "bar"]` 이런식으로 보낼 수 있음
- `encoding` 👉 `URL`이기에 `URLEncoding` 넣기
- `headers` 👉 헤더 값 할당, `json` 형식으로 받게끔 작성, **_`[String:String]`형태로 보낼 수 있음_**
- `validate` 👉 유효성 검사
- `responseJSON` 👉 정보를 받는 부분

### 🌱 GET

> **_예제_** 👇

```swift
import Alamofire

func getTest() {
    let url = "https://jsonplaceholder.typicode.com/todos/1"
    AF.request(
        url,
        method: .get,
        parameters: nil,
        encoding: URLEncoding.default,
        headers: ["Content-Type":"application/json", "Accept":"application/json"]
    )
    .validate(statusCode: 200..<300) // 200 ~ 300 사이의 상태코드만 허용
    .responseJSON { (json) in
        // 가쟈온 데이터를 사용
    }
}
```

### 🌱 POST

> **_예제_** 👇

```swift
import Alamofire

func postTest() {
    let url = "https://ptsv2.com/t/im8p3-1592789118/post"
    var request = URLRequest(url: URL(string: url)!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.timeoutInterval = 10

    // POST로 보낼 정보
    let params = ["id":"아이디", "pw":"패스워드"] as Dictionary

    // httpBody 에 parameters 추가
    do {
        try request.httpBody = JSONSerialization.data(withJSONObject: parmas, options: [])
    } catch {
        print("http Body Error")
    }

    AF.request(request).responseString { (response) in
        switch response.result {
        case .success:
            print("POST 성공")
        case .failure(let error):
            print("🚫 Alamofire Request Error\nCode:\(error._code), Message: \(error.errorDescription!)")
        }
    }
}
```

## ☘️ AF.Download

- 첫 번째 파라미터 👉 `Url`
- `method` 👉 통신방식 선택 (`get`, `post`, `delete`, `patch`, `put` 등)
- `parameters` 👉 `paramters` 가 없는 경우 `nil` 값 할당
- `encoding` 👉 `JSON`이기에 `JSONEncoding` 넣기
- `to` 👉 **_파일 경로 지정 및 다운로드 옵션 설정_**

  > 옵션 👉 이전 파일 삭제, 디렉토리 생성 등등..

  > **_예제_** 👇

  ```swift
  func downTest() {
      let url = "http://212.183.159.230/50MB.zip"
      // 파일 메니저
      let fileManager = FileManager.default
      // 앱 경로
      let appURL = fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
      // 파일 이름 url 의 맨 뒤 컴포넌트로 지정 (50MB.zip)
      let fileName: String = URL(string: url)!.lastPathComponent
      // 파일 경로 생성
      let fileURL = appURL.appendingPathComponent(fileName)
      // 파일 경로 지정 및 다운로드 옵션 설정 (이전 파일 삭제, 디렉토리 생성)
      let destination: DownloadRequest.Destination = { _, _, in
          return (fileUrl, [.removePreviousFile, .createIntermediateDirectories])
      }

      // 다운로드 시작
      AF.download(
          url,
          method: .get,
          parameters: nil,
          encoding: JSONEncoding.default,
          do: destination
      ).downloadProgress { (progress) in
          // 이 부분에서 프로그레스 수정
          self.progressView.progress = Float(progress.fractionCompleted)
          self.progressLabel.text = "\(Int(progress.fractionCompleted * 1000))"
      }.response { response in
          if response.error != nil {
              print("파일다운로드 실패")
          } else {
              print("파일다운로드 완료")
          }
      }
  }
  ```
