---
layout: post
title:  "옵저버 패턴"
date:   2018-09-25 18:48:00 +0900
categories: observer pattern
---

### 옵저버 패턴

> 코코아 터치 프레임워크에서 자주 사용하는 옵저버 패턴에 대해 알아본다.

#### 옵저버 패턴이란 ?

옵저버 패턴을 이해하기 위해 ```observer``` 와 ```subject``` 의 정의와 역할에 대해서 알아야 한다.  

옵저버는 리스너(listener)라고도 부른다. subject는 관찰 대상 즉 ''이벤트를 발생시키는 주체''다. observer와 subject가 동작하는 방식은 다음과 같다.

1. subject에 observer 목록을 등록한다. 
2. subject의 상태 변화가 발생한다. 
3. subject는 이벤트를 발생시켜 observer에게 통지한다.
4. obserser는 이벤트를 받아서 처리한다.

#### 옵저버 패턴이 필요한 이유

#### NotificationCenter

코코아 터치 프레임워크에서 옵저버 패턴의 대표적인 예가 NotificationCenter이다. 

```swift
/*
옵저버 등록/삭제하기
*/
func addObserver(forName name: NSNotification.Name?, 
                 object obj: Any?, 
                 queue: OperationQueue?, 
                 using block: @escaping (Notification) -> Void) -> NSObjectProtocol

func addObserver(_ observer: Any, 
                 selector aSelector: Selector, 
                 name aName: NSNotification.Name?,
                 object anObject: Any?)

func removeObserver(_ observer: Any, 
               name aName: NSNotification.Name?, 
             object anObject: Any?)

func removeObserver(_ observer: Any)

/*
옵저버에게 알리기
*/
func post(name aName: NSNotification.Name, 
   object anObject: Any?, 
 userInfo aUserInfo: [AnyHashable : Any]? = nil)

```

---

참조

[https://learnappmaking.com/notification-center-how-to-swift/](https://learnappmaking.com/notification-center-how-to-swift/)

