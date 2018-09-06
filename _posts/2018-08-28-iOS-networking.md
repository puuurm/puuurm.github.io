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

HTTP 메시지들의 전송은 전송계층인 TCP/IP 를 통해 이루어진다. TCP/IP 커넥션이 맺어지면 클라이언트와 서버 간에 주고받는 메시지들은 손실/손상되거나 순서가 바뀌지 않고 안전하게 전달된다.

#### 1. Mac OS/ iOS에서 HTTP 통신하기

애플이 제공하는 URL loading 시스템은 생성한 https 또는 사용자 정의 프로토콜과 같은 표준 프로토콜을 사용하여 URL로 식별되는 자원에 대한 액세스를 제공한다. 로딩은 비동기 적으로 수행된다.

URLSession 인스턴스는 네트워크 데이터 전송 

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

- URLSessionConfiguration : URL 세션에 관한 다양한 설정이 가능하다.

  - 타임 아웃 설정 : 데이터를 전부 받아오지 못할 때, 얼마 동안 요청을 시도할 것인지 설정한다.

    ```swift
    // 추가 데이터를 기다릴 때 얼마까지 기다릴지 설정. default: 60 sec
    var timeoutIntervalForRequest: TimeInterval { get set }

    // 백그라운드 세션이 실패한 작업을 얼마동안 자동으로 실행 할건지 설정. default: 7 days
    var timeoutIntervalForResource: TimeInterval { get set }
    ```


  - 캐시 정책 : 세션 설정의 종류를 지정함으로써 캐시 정책을 설정할 수 있다.

    ```swift
    // 영속적인 디스크 기반 캐시를 사용하며, 사용자 키체인에 자격 증명을 저장한다. 쿠키를 저장함.
    class var `default`: URLSessionConfiguration { get }

    // 캐시, 자격 증명, 세션 관련된 데이터를 디스크에 저장하지 않는다. 대신 세션 관련 데이터를 RAM에 저장한다.
    class var ephemeral: URLSessionConfiguration { get }

    // 백그라운드 상태에서 HTTP 업로드/다운로드가 수행된다.
    class func background(withIdentifier identifier: String) -> URLSessionConfiguration
    ```

    ```swift
    // 특정 세션에서 요청했던 캐시된 응답. default session의 경우 캐시된다고 했는데, 이곳에서 설정한다.
    var urlCache: URLCache? { get set }
    ```

  - 클라이언트를 식별하기 위한 장치인 쿠키에 관한 정책을 설정할 수 있다.

- Delegate : 세션/ task 레벨의 이벤트 처리를 URLSessionDelegate 프로토콜을 채택하여 구현할 수 있다. delegate를 사용하지 않으면 completionHandler 에서 응답을 처리한다.

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