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

#### 1. TCP/IP

서버/클라이언트 사이의 HTTP 메시지 전송은 전송계층인 TCP와 네트워크 계층인 IP를 통해 이루어진다. TCP는 프로세스 간의 통신이라면 IP는 호스트 대 호스트 통신을 가능하게 한다. IP는 목적지 호스트에 메시지를 전달할 뿐 적절한 프로세스에게 메세지를 전송하려면 TCP가 필요하다.

TCP는 프로그램으로 부터 데이터 블록을 받으면 이를 여러 개의 세그먼트로 분리한다. 세그먼트에는 sequence number가 존재하는데 이 시퀀스 넘버 덕분에 순서가 바뀌지 않고 안전하게 전달할 수 있다. 또한 TCP는 신뢰성을 실현하기 위해 재 전송 체계를 사용한다. 종료 시간 안에 ACK 세그먼트가 수신되지 않으면 데이터가 재전송 된다.

#### 2. Mac OS/ iOS에서 HTTP 통신하기

애플이 제공하는 URL loading 시스템은 생성한 https 또는 사용자 정의 프로토콜과 같은 표준 프로토콜을 사용하여 URL로 식별되는 자원에 대한 액세스를 제공한다. 로딩은 비동기 적으로 수행된다.

URLSession 인스턴스를 사용해 통신하는 방법은 다음과 같다.

1. Session을 생성한다.
2. URL, Request 객체를 만든다.
3. Task를 생성한다.
4. Task의 resume() 메소드를 호출하여 요청을 실행한다.
5. 응답은 completionHandler 혹은 delegate를 통해 받는다.

#### 2-1. Session 생성하기

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
    /* 
    추가 데이터를 기다릴 때 얼마까지 기다릴지 설정. 
    default: 60 sec
    */
    var timeoutIntervalForRequest: TimeInterval { get set }

    /* 
    백그라운드 세션이 실패한 작업을 얼마동안 자동으로 실행 할건지 설정. 
    default: 7 days
    */
    var timeoutIntervalForResource: TimeInterval { get set }
    ```


  - 캐시 정책 : 세션 설정의 종류를 지정함으로써 캐시 정책을 설정할 수 있다.

    ```swift
    /*
    영속적인 디스크 기반 캐시를 사용하며, 사용자 키체인에 자격 증명을 저장한다. 
    쿠키를 저장함.
    */
    class var `default`: URLSessionConfiguration { get }

    /* 
    캐시, 자격 증명, 세션 관련된 데이터를 디스크에 저장하지 않는다. 
    대신 세션 관련 데이터를 RAM에 저장한다.
    */
    class var ephemeral: URLSessionConfiguration { get }

    /* 
    백그라운드 상태에서 HTTP 업로드/다운로드가 수행된다.
    */
    class func background(withIdentifier identifier: String) -> URLSessionConfiguration
    ```

    ```swift
    /*
    특정 세션에서 요청했던 캐시된 응답. 
    default session의 경우 캐시된다고 했는데, 이곳에서 설정한다.
    default session일 때, urlCache의 기본 값은 URLCache.shared 이다.
    */
    var urlCache: URLCache? { get set }
    ```

  - 클라이언트를 식별하기 위한 장치인 쿠키에 관한 정책을 설정할 수 있다.

- Delegate : 세션/ task 레벨의 이벤트 처리를 URLSessionDelegate 프로토콜을 채택하여 구현할 수 있다. delegate를 사용하지 않으면 completionHandler 에서 응답을 처리한다.

#### 2-2. URL, Request 객체 만들기 

모든 요청은 URL을 필요로 한다. Request 객체는 요청 메시지에 해당하는 객체로 앞서 얘기한 HTTP 메소드, url, 바디를 포함하여 캐시 정책 정보를 가지고 있다. HTTP 메소드는 디폴트로 "GET" 값을 가진다.

#### 2-3. Task 생성 

Task는 url 세션에서 수행된다. 

- dataTask

  서버의 응답 데이터를 메모리에 저장한다. Background 세션을 제외한 모든 종류의 세션에서 지원된다.

- downloadTask

  서버의 응답 데이터를 temporary file에 기록한다. Suspended/not running 상태에서도 다운로드가 지속된다. (temporary file은 앱이 not running 상태일 때 해당 디렉토리의 데이터를 전부 삭제한다. iTunes나 iCloud에 백업되지 않는다.)

- uploadTask

  업로드 태스크는 데이터 태스크를 상속 받았다. 앱이 실행되지 않는 동안에도 백그라운드 업로드를 지원한다.

#### 3. 캐시 데이터 접근하기

세션을 생성하여 클라이언트와 서버간에 통신을 하면 응답 데이터를 캐시한다고 했다. 그렇다면 이 캐시 데이터의 접근/삭제와 같은 관리는 어떻게 해야할까?

- 캐시 정책을 설정하여 캐시 데이터를 관리할 수 있다.

  URLRequest.CachePolicy으로 캐시 정책을 설정한다. 캐시 정책은 요청 마다 설정할 수도 있고, 세션 마다 설정할 수도 있다. 만약 세션 마다 캐시 정책을 설정하려면, URLSessionConfiguration의 requestCachePolicy 프로퍼티를 사용한다. URLRequest.CachePolicy 는 4개지 타입의 캐시 정책이 있다.

  1. reloadIgnoringLocalCacheData: 캐시를 사용하지 않고 원 서버에서만 로드한다.

  2. returnCacheDataDontLoad: 캐시 데이터의 나이(age)와 유효기간(expiration

     date)에 상관없이 존재하는 캐시 데이터를 사용한다. 없다면 fail을 리턴.

  3. returnCacheDataElseLoad: 캐시 데이터의 나이(age)와 유효기간(expiration

     date)에 상관없이 존재하는 캐시 데이터를 사용한다. 없다면 원본 소스에서 로딩.

  4. useProtocolCachePolicy: returnCacheDataElseLoad와 같지만, 캐시 데이터의 만료일과 age 초과시 원본 소스에서 변화된 내용을 체크한다는 점에서 다르다. 원본에 변화가 있다면 원본 데이터를, 아니면 캐시 데이터를 리턴한다.

  HTTP는 Cache-Control과 Expires라는 특별한 헤더들을 이용해서 원 서버가 각 데이터에 유효 기간을 붙일 수 있게 해준다. Cache-Control의 max-age 값은 초 단위 이며, 데이터가 생성된 이후 max-age (초) 까지 캐시 데이터가 유효하다는 의미이다. 즉, 데이터가 생성된 현재 시간에서 유효 기간까지의 사이 간격을 초 단위로 나타낸 것이다. Expires는 절대 시간이다. 컴퓨터의 시간이 올바르게 맞추어져 있을 때 제대로 작동한다.

  ```
  Expires: Mon, 01 Oct 2018, 05:00:00 GMT
  Cache-Control: max-age=484200
  ```

- 캐시에 직접 접근하여 관리

  - 앞에서 URLSessionConfiguration의 urlCache 인스턴스를 통해 캐시 설정을 한다고 했다.
  - 특정 request에 대한 캐시된 응답을 확인하기 위해서는 urlCache의 cachedResponse(for:)을 호출한다. 이 메소드를 호출하면 CachedURLResponse가 리턴된다.
  - urlCache의 removeCachedResponse 관련 메소드를 사용하여 캐시된 데이터를 직접 지울 수 있다.





---

참고

[애플 공식문서: URL loading system](https://developer.apple.com/documentation/foundation/url_loading_system)

[https://www.raywenderlich.com/567-urlsession-tutorial-getting-started](https://www.raywenderlich.com/567-urlsession-tutorial-getting-started)