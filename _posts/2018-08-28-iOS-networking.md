---
layout: post
title:  "iOS에서 네트워크 통신하기"
date:   2018-08-28 20:21:00 +0900
categories: iOS networking
---

### iOS에서 네트워크 통신을 어떻게 할까?

> 서버에 데이터를 업로드/다운로드 하기 위해 애플은 URLSession을 제공한다. URLSession은 여러 프로토콜을 지원하지만 그 중에서 많이 쓰이는 HTTP 통신을 어떻게 구현하는지 알아보자.

#### 0. HTTP 통신

HTTP는 Hyper Text Transfer Protocol의 약자로 철자 그대로 해석하면 하이퍼텍스트 전송 프로토콜이다. 하이퍼링크를 통해 텍스트가 비 선형적으로 연결된 환경이 하이퍼텍스트이며, 인터넷의 모든 페이지는 하이퍼텍스트의 형태를 가지고 있다. 즉 HTTP는 웹 서버에 존재하는 리소스를 클라이언트가 요청 메시지를 통해 요구하고 서버가 응답 메시지를 통해 전달하기 위한 통신 규약이다.

요청 메시지와 응답 메시지에는 시작줄, 헤더, 바디가 있다. 요청 메시지의 시작줄에는 ```메서드```, ```URL```, ```HTTP 버전```이 있고,  응답 메시지의 시작줄에는 ```HTTP 버전```, ```상태 코드```, ```사유 구절``` 이 있다. 헤더에는 content-length, content-type과 같은 부가 정보가 들어가 있다. 바디는 선택 사항인데, 보통 응답 메시지의 바디에 요청 받은 데이터가 들어있다.

HTTP 메시지들의 전송은 전송계층인 TCP/IP 를 통해 이루어진다. TCP/IP 커넥션이 맺어지면 클라이언트와 서버 간에 주고받는 메시지들은 손실 혹은 손상되거나 순서가 바뀌지 않고 안전하게 전달된다.

#### 1. Mac OS/ iOS에서 HTTP 통신하기

애플이 제공하는 URL loading 시스템은 생성한 https 또는 사용자 정의 프로토콜과 같은 표준 프로토콜을 사용하여 URL로 식별되는 자원에 대한 액세스를 제공한다. 로딩은 비동기 적으로 수행된다.

URLSession 인스턴스를 사용해 통신하는 방법은 다음과 같다.

1. Session을 생성한다.
2. URL, Request 객체를 만든다.
3. Task를 생성한다.
4. Task의 resume() 메소드를 호출하여 요청을 실행한다.
5. 응답은 completionHandler 혹은 delegate를 통해 받는다.

#### 1-1. Session 생성하기

URLSession을 생성하는 방법은 세 가지가 있다.

```swift
init(configuration: URLSessionConfiguration)
init(configuration:delegate:delegateQueue:)
class var shared: URLSession
```

- Shared

  가장 단순해 보이는 것이 shared 싱글턴 인스턴스이다. 책이나 구글링을 통해 네트워크 통신 예제를 보면 URLSession의 shared 인스턴스를 사용하는 것을 자주 볼 수 있다.

  shared 인스턴스는 세션을 새로 만들 필요 없이 싱글톤 인스턴스에 접근만 하면 된다. session configuration, delegate 객체를 설정하지 않기 때문에 제한 사항이 존재한다. 캐시, 쿠키, 인증 또는 커스텀 네트워킹 프로토콜을 사용하여 작업한다면 shared 인스턴스를 사용하지 않고 직접 URLSession 인스턴스를 생성해야 한다.

- URLSessionConfiguration 

  캐시 및 쿠키를 사용하는 방법, 이동 통신망에서 연결을 허용할지 여부와 같은 동작을 제어할 수 있다. 세션 설정은 default, ephemeral, background 세 가지가 있다. 

  ```# default```

  요청에 대한 결과가 파일로 다운로드 될 때를 제외하고는 영구 디스크 기반 캐시를 사용하고 사용자의 키 체인에 자격 증명을 저장한다. 또한 NSURLConnection 및 NSURLDownload 클래스와 동일한 공유 쿠키 저장소에 쿠키 를 저장한다.

  ```# ephemeral```

  Default 세션과 유사하지만 캐시, 자격 증명 저장소 또는 세션 관련 데이터를 디스크에 저장하지 않는다는 점에서 다르다. 대신 세션 관련 데이터가 RAM에 저장된다. 임시 세션이 디스크에 데이터를 쓰는 유일한 경우는 URL의 내용을 파일에 쓸 때이다.

  임시 세션을 사용하는 주된 목적은 개인 정보 보호이다. 잠재적으로 민감한 데이터를 디스크에 쓰지 않으면 나중에 데이터를 변경할 가능성이 줄어 든다. 이러한 이유 때문에 임시 세션은 웹 브라우저 및 기타 유사한 상황에서 개인 브라우징 모드에 이상적이다.

  ```# background(withIdentifier:)```

  앱이 백그라운드에서 실행되는 동안 데이터 파일을 전송하는데 적합한 설정 객체를 초기화한다.

- Delegate

  세션/ task 레벨의 이벤트 처리를 URLSessionDelegate 프로토콜을 채택하여 구현할 수 있다. delegate를 사용하지 않으면 completionHandler 에서 응답을 처리한다.

#### 1-2. URL, Request 객체 만들기 

모든 요청은 URL을 필요로 한다. Request 객체는 요청 메시지에 해당하는 객체로 앞서 얘기한 HTTP 메소드, url, 바디를 포함하여 캐시 정책 정보를 가지고 있다. HTTP 메소드는 디폴트로 "GET" 값을 가진다.

#### 1-3. Task 생성 

Task는 url 세션에서 수행된다. 

- dataTask

  리소스를 요청하고 서버의 응답으로 NSData 객체를 메모리에 리턴한다.

- downloadTask

  백그라운드에서 디스크에 파일 형태로 리소스를 다운로드 한다.

- uploadTask

  업로드 작업은 서버의 응답을 검색하기 전에 데이터를 업로드 할 수 있도록 요청 본문을 쉽게 제공 할 수 있다는 점을 제외하면 데이터 작업과 같다. 백그라운드 세션에서 지원된다.

---

참고

[애플 공식문서: URL loading system](https://developer.apple.com/documentation/foundation/url_loading_system)

[https://www.raywenderlich.com/567-urlsession-tutorial-getting-started](https://www.raywenderlich.com/567-urlsession-tutorial-getting-started)