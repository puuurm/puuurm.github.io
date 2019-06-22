---
layout: post
title: "HTTP 메소드: PUT과 POST"
date: 2019-06-22 20:28:00 +0900
categories: PUT And POST
---

HTTP 메소드는 대표적으로 GET, POST, PUT, HEAD 등이 있지만, 그 중에서도 PUT과 POST의 정확한 차이를 몰랐다.

- 공통점: CRUD 중에 Create에 속하여 서버 사이드 데이터의 조작을 발생함

- 차이점

  1. URI 생성 주체

  두 메소드 다 새로운 리소스 생성이 가능하지만 ```POST``` 는 ''존재하는 URI" 를 요청하고 응답으로 새로운 URI를 생성한다. ``` PUT``` 은 "새로운 URI" 를 요청한다. 

  ```POST``` 는 새로운 URI를 생성하는 주체가 ```클라이언트라면 ``` , ```PUT``` 은 ```서버```가 새로운 URI를 생성하다.

  ​

  POST로 새로운 포스트를 작성하는 요청 메시지 예시이다.

  ```
  POST /post HTTP1.1
  Host: blog.com
  Content-Type: text/plain: charset=utf-8

  Hello!
  ```

  이런식으로 클라이언트에서 서버에 요청을 했다고 하면, 이미 존재하고 있는 /post 디렉토리 아래에 예를들어 /post/1 라는 새로운 URI를 생성한다. 이 새로운 URI는 응답의 Location 헤더를 통해서 확인할 수 있다.

  ```
  HTTP/1.1 201 Created
  Content-Type: text/plain: charset=utf-8
  Location: http://blog.com/post/1

  Hello!
  ```

  ​

  PUT 역시 새로운 리소스를 작성할 수 있다.

  ```
  PUT /newPost HTTP/1.1
  Host: blog.com

  Hello!
  ```

  여기서 POST와 다른점은 요청 URI가 곧 입력/수정할 URI라는 것이다.

  - /newPost가 존재하지 않는 URI일 경우 이 URI가 새로 작성한 리소스 주소가 된다
  - 이미 존재하는 URI인 경우 이 URI가 가리키는 리소스 내용을 수정한다.



 	2. 멱등성

멱등성이란 몇 번을 조작해도 결과는 같다는 의미이다. PUT은 멱등이지만 POST는 멱등이지 않다. 물론 제대로 개발하지 않다면 둘 다 멱등하지 않을수도..



---

출처

웹을 지탱하는 기술